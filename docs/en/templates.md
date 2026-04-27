````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Templates

The `client.Templates` sub-client provides access to template configurations and template content. Templates define the message structure for each channel (SMS, Email, WhatsApp), including static text, dynamic variable placeholders, and provider-specific settings.

## Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List template configurations with filters and pagination |
| `GetAsync(id)` | Get a template configuration by ID |
| `GetWithDetailsAsync(id)` | Get a template configuration with full navigation properties |
| `ValidateAsync(request)` | Validate whether a template configuration can be used for sending |
| `GetPreviewAsync(request)` | Preview a rendered template with sample variable values |
| `GetTemplateAsync(id)` | Get the raw template content (body, header, footer) |
| `GetTemplateButtonsAsync(id)` | Get the buttons defined in a WhatsApp template |
| `GetTemplateVariableDefinitionsAsync(id)` | Get the variable definitions for a template |
| `GetTemplateVariablesWithSectionsAsync(...)` | Get variable definitions grouped by section |

## Template naming convention

Template configurations follow this naming pattern:

```
{InstanceCode}_{ApplicationCode}_{NotificationTypeCode}_{NotificationSubtypeCode}_{RegionCode}_{LanguageCode}_{ProviderCode}_{ChannelTypeCode}_v{Version}
```

Example: `prd_billing_trans_invoice_us_en_twilio_sms_v1`

Each segment maps to a dimension used when the platform selects the correct template for a `SendNotificationRequest`.

## Listing templates

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Templates.Models;

var list = await client.Templates.GetListAsync(new GetTemplateConfigurationsRequest
{
    SkipCount      = 0,
    MaxResultCount = 20
});

Console.WriteLine($"Total templates: {list.TotalCount}");
foreach (var item in list.Items)
{
    // item is TemplateConfigurationWithDetailsResponse: TemplateConfiguration + LookupItems
    Console.WriteLine($"{item.TemplateConfiguration?.Id} — {item.TemplateConfiguration?.Name}");
    Console.WriteLine($"  Provider: {item.Provider?.Name}, Country: {item.Country?.Code}, App: {item.Application?.Code}");
}
```
{{end}}

## Getting a template configuration by ID

`GetAsync(id)` returns the bare `TemplateConfigurationResponse` (no navigation properties).

{{if SDK == "csharp"}}
```csharp
var config = await client.Templates.GetAsync(templateConfigurationId);
Console.WriteLine($"Name: {config.Name}");
Console.WriteLine($"Approval state: {config.ApprovalState}");   // NotRequired, Pending, Approved, ...
Console.WriteLine($"Active: {config.IsActive}");
Console.WriteLine($"Linked template ID: {config.TemplateId}");  // null if no template linked yet
```
{{end}}

## Getting a template configuration with details

Returns the configuration along with its lookup data (provider, country, application, channel, etc.) and the linked `Template` (single).

{{if SDK == "csharp"}}
```csharp
var details = await client.Templates.GetWithDetailsAsync(templateConfigurationId);

Console.WriteLine($"Provider: {details.Provider?.Name}");
Console.WriteLine($"Country: {details.Country?.Code}");
Console.WriteLine($"Channel: {details.ChannelType?.Code}");
Console.WriteLine($"Linked template body: {details.Template?.Body}");
```
{{end}}

## Validating a template

Validation checks whether a template configuration is complete for the given combination of application, region, language, and channels. Use this before initiating a send campaign to detect missing channel configurations early.

`TemplateConfigurationValidationRequest` fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `ApplicationCode` | `string` | Yes | Application code (e.g., `"billing"`) |
| `RegionCode` | `string` | Yes | Region code (alphanumeric, max 10 characters, e.g., `"us"`) |
| `Language` | `string` | Yes | Language code (e.g., `"en"`) |
| `NotificationTypeCode` | `string` | Yes | Notification type code (e.g., `"trans"`) |
| `NotificationSubtypeCode` | `string` | Yes | Notification subtype code (e.g., `"invoice"`) |
| `DeploymentEnvironmentId` | `Guid` | Yes | Deployment environment ID |
| `TargetChannels` | `List<TargetChannel>` | No | Channels to validate. If empty, all configured channels are checked. |

`ValidateAsync` returns `TemplateConfigurationValidationResponse`:

| Field | Type | Description |
|---|---|---|
| `Status` | `TemplateConfigurationValidationStatus` | `None` = no channels configured, `Partial` = some channels ready, `All` = all channels ready |
| `ConfiguredChannels` | `List<string>` | Channels that are fully configured |
| `UnconfiguredChannels` | `List<string>` | Channels that are missing or incomplete |

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Templates.Models;

var result = await client.Templates.ValidateAsync(new TemplateConfigurationValidationRequest
{
    ApplicationCode        = "billing",
    RegionCode             = "us",
    Language               = "en",
    NotificationTypeCode   = "trans",
    NotificationSubtypeCode = "invoice",
    DeploymentEnvironmentId = deploymentEnvId,
    // Optionally limit to specific channels:
    TargetChannels = [new TargetChannel { Channel = "sms" }, new TargetChannel { Channel = "email" }]
});

// Status tells you at a glance whether you can send
Console.WriteLine($"Validation status: {result.Status}");

// See exactly which channels are ready
foreach (var channel in result.ConfiguredChannels ?? [])
    Console.WriteLine($"Ready: {channel}");

// See which channels still need configuration
foreach (var channel in result.UnconfiguredChannels ?? [])
    Console.WriteLine($"Not configured: {channel}");
```
{{end}}

## Previewing a template configuration

Returns sample data you can use to send a test notification from a template configuration: a generated transaction ID, codes, sample recipients, and pre-filled variable values. The method takes the **template configuration ID** (not a full request object).

`GetPreviewAsync(Guid templateConfigurationId)` returns `TemplateConfigurationPreviewResponse`:

| Field | Type | Description |
|---|---|---|
| `TransactionId` | `string` | Generated transaction ID for this preview |
| `ApplicationCode` | `string` | Application code |
| `Language` | `string` | Language code |
| `RegionCode` | `string` | Region code |
| `NotificationTypeCode` | `string` | Notification type code |
| `NotificationSubtypeCode` | `string` | Notification subtype code |
| `TargetChannels` | `List<PreviewChannel>` | Each item exposes only `Channel` (the channel code) |
| `Recipients` | `List<PreviewRecipient>` | Each item exposes `Channel` and `Recipient` (sample address) |
| `TemplateVariables` | `List<PreviewTemplateVariable>` | Each item exposes `Key` and `Value` (sample variable) |

{{if SDK == "csharp"}}
```csharp
var preview = await client.Templates.GetPreviewAsync(templateConfigurationId);

Console.WriteLine($"Transaction ID: {preview.TransactionId}");
Console.WriteLine($"App: {preview.ApplicationCode} — Region: {preview.RegionCode}");

foreach (var channel in preview.TargetChannels)
    Console.WriteLine($"Channel: {channel.Channel}");

foreach (var recipient in preview.Recipients)
    Console.WriteLine($"  {recipient.Channel}: {recipient.Recipient}");

foreach (var variable in preview.TemplateVariables)
    Console.WriteLine($"  {variable.Key} = {variable.Value}");
```
{{end}}

## Getting raw template content

{{if SDK == "csharp"}}
```csharp
var template = await client.Templates.GetTemplateAsync(templateId);
Console.WriteLine($"Body: {template.Body}");
Console.WriteLine($"Header type: {template.HeaderType}");
```
{{end}}

## Getting template buttons (WhatsApp)

Returns `List<TemplateButtonResponse>`. Each item has `Id`, `Label`, `Value`, `ButtonType`, `Order`, `TemplateId`, `ConcurrencyStamp`, plus a list of variables used in the button.

{{if SDK == "csharp"}}
```csharp
var buttons = await client.Templates.GetTemplateButtonsAsync(templateId);
foreach (var button in buttons)
    Console.WriteLine($"Button: {button.Label} ({button.ButtonType}) → {button.Value}");
```
{{end}}

## Getting template variable definitions

Variable definitions describe the named placeholders a template accepts. Use the returned `Name` values as keys in `SendNotificationRequest.TemplateVariables`.

`GetTemplateVariableDefinitionsAsync(Guid templateId)` returns `List<TemplateVariableDefinitionResponse>` with: `Id`, `Name`, `SampleValue`, `TemplateId`.

{{if SDK == "csharp"}}
```csharp
var definitions = await client.Templates.GetTemplateVariableDefinitionsAsync(templateId);
foreach (var def in definitions)
    Console.WriteLine($"{def.Name}  (sample: {def.SampleValue})");
```
{{end}}

## Getting template variables with sections

Returns variable definitions enriched with the template sections in which each variable is used. Useful when you need to know whether a variable appears in `Header`, `Body`, `Footer`, etc.

`GetTemplateVariablesWithSectionsAsync(Guid templateConfigurationId, Guid providerId)` returns a flat `List<TemplateVariableDefinitionWithSectionsResponse>`. Each item is a `TemplateVariableDefinitionResponse` extended with:

| Field | Type | Description |
|---|---|---|
| `Sections` | `List<TemplateSection>?` | Sections where this variable is used (`Subject`, `Header`, `Body`, `Footer`, `Buttons`) |
| `NotificationVariablesType` | `NotificationVariablesType` | `Template`, `Auto`, `Property`, `TemplateName`, `TemplateLanguage`, `Optional` |

{{if SDK == "csharp"}}
```csharp
var variables = await client.Templates.GetTemplateVariablesWithSectionsAsync(
    templateConfigurationId,
    providerId);

foreach (var v in variables)
{
    var sections = string.Join(", ", v.Sections ?? []);
    Console.WriteLine($"{v.Name}  type={v.NotificationVariablesType}  sections=[{sections}]");
}
```
{{end}}
