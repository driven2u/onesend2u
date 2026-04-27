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
    SkipCount  = 0,
    MaxResults = 20
});

Console.WriteLine($"Total templates: {list.TotalCount}");
foreach (var template in list.Items)
    Console.WriteLine($"{template.Id} — {template.Name}");
```
{{end}}

## Getting a template by ID

{{if SDK == "csharp"}}
```csharp
var template = await client.Templates.GetAsync(templateId);
Console.WriteLine($"Name: {template.Name}");
Console.WriteLine($"Validation status: {template.ValidationStatus}");
```
{{end}}

## Getting a template with details

Returns the full template configuration including its child template records, buttons, and variable definitions.

{{if SDK == "csharp"}}
```csharp
var details = await client.Templates.GetWithDetailsAsync(templateId);
Console.WriteLine($"Template count: {details.Templates?.Count}");
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

## Previewing a template

Renders the template with sample variable values so you can verify the output before sending. Takes the same lookup fields as `ValidateAsync`.

`GetPreviewAsync` returns `TemplateConfigurationPreviewResponse`:

| Field | Type | Description |
|---|---|---|
| `TransactionId` | `string` | Generated transaction ID for this preview |
| `ApplicationCode` | `string` | Application code |
| `Language` | `string` | Language code |
| `RegionCode` | `string` | Region code |
| `NotificationTypeCode` | `string` | Notification type code |
| `NotificationSubtypeCode` | `string` | Notification subtype code |
| `TargetChannels` | `List<PreviewChannel>` | Rendered output per channel |
| `Recipients` | `List<PreviewRecipient>` | Sample recipients |
| `TemplateVariables` | `List<PreviewTemplateVariable>` | Key/value pairs used in rendering |

{{if SDK == "csharp"}}
```csharp
var preview = await client.Templates.GetPreviewAsync(new TemplateConfigurationValidationRequest
{
    ApplicationCode        = "billing",
    RegionCode             = "us",
    Language               = "en",
    NotificationTypeCode   = "trans",
    NotificationSubtypeCode = "invoice",
    DeploymentEnvironmentId = deploymentEnvId
});

// The rendered output is split by channel — not a single Body field
foreach (var channel in preview.TargetChannels ?? [])
    Console.WriteLine($"Channel: {channel.Channel}");

// See which variables were used in the rendering
foreach (var variable in preview.TemplateVariables ?? [])
    Console.WriteLine($"{variable.Key} = {variable.Value}");
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

{{if SDK == "csharp"}}
```csharp
var buttons = await client.Templates.GetTemplateButtonsAsync(templateId);
foreach (var button in buttons)
    Console.WriteLine($"Button: {button.Text} ({button.Type})");
```
{{end}}

## Getting template variable definitions

Variable definitions describe the placeholders used in a template, including their name, type, and whether they are required.

{{if SDK == "csharp"}}
```csharp
var definitions = await client.Templates.GetTemplateVariableDefinitionsAsync(templateId);
foreach (var def in definitions)
    Console.WriteLine($"{def.Name} ({def.Type}) — Required: {def.IsRequired}");
```
{{end}}

## Getting template variables with sections

Returns variable definitions grouped by template section (header, body, footer, buttons).

{{if SDK == "csharp"}}
```csharp
var withSections = await client.Templates.GetTemplateVariablesWithSectionsAsync(templateId);
foreach (var section in withSections)
{
    Console.WriteLine($"Section: {section.Section}");
    foreach (var variable in section.Variables ?? [])
        Console.WriteLine($"  {variable.Name}");
}
```
{{end}}
