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
{InstanceCode}_{ApplicationCode}_{NotificationTypeCode}_{NotificationSubtypeCode}_{CountryCode}_{LanguageCode}_{ProviderCode}_{ChannelTypeCode}_v{Version}
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

Validation checks whether a template configuration is complete and ready for use. Useful before initiating a send campaign.

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Templates.Models;

var result = await client.Templates.ValidateAsync(new TemplateConfigurationValidationRequest
{
    TemplateConfigurationId = templateId
});

if (result.IsValid)
{
    Console.WriteLine("Template is ready to use.");
}
else
{
    foreach (var error in result.Errors ?? [])
        Console.WriteLine($"Error: {error}");
}
```
{{end}}

## Previewing a template

Renders the template body with sample variable values so you can verify the output before sending.

{{if SDK == "csharp"}}
```csharp
var preview = await client.Templates.GetPreviewAsync(templateId);
Console.WriteLine($"Preview body: {preview.Body}");
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
