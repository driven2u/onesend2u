# Plantillas

El sub-cliente `client.Templates` cubre 9 endpoints para consultar, validar y previsualizar configuraciones de plantillas.

> Las plantillas son de solo lectura desde el SDK. La creación y edición se realiza desde el portal OneSend2U.

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista configuraciones de plantillas con filtros |
| `GetAsync(id)` | GET | Obtiene una configuración de plantilla por ID |
| `GetWithDetailsAsync(id)` | GET | Obtiene la configuración con propiedades de navegación |
| `ValidateAsync(request)` | GET | Valida que existe una plantilla para los parámetros dados |
| `GetPreviewAsync(request)` | GET | Genera una vista previa del mensaje resultante |
| `GetTemplateAsync(id)` | GET | Obtiene el contenido de una plantilla específica |
| `GetTemplateButtonsAsync(id)` | GET | Obtiene los botones definidos en una plantilla |
| `GetTemplateVariableDefinitionsAsync(id)` | GET | Obtiene las variables disponibles en una plantilla |
| `GetTemplateVariablesWithSectionsAsync(request)` | GET | Obtiene las variables agrupadas por sección |

## Convención de nombres de plantillas

Las plantillas siguen este formato de nomenclatura:

```
{InstanceCode}_{ApplicationCode}_{NotificationTypeCode}_{NotificationSubtypeCode}_{CountryCode}_{LanguageCode}_{ProviderCode}_{ChannelTypeCode}_v{Version}
```

**Ejemplo:**
```
prd_myapp_transactional_invoice_br_pt_twilio_sms_v1
```

| Segmento | Ejemplo | Descripción |
|---|---|---|
| `InstanceCode` | `prd` | Entorno (prd = producción) |
| `ApplicationCode` | `myapp` | Código de la aplicación |
| `NotificationTypeCode` | `transactional` | Tipo de notificación |
| `NotificationSubtypeCode` | `invoice` | Subtipo de notificación |
| `CountryCode` | `br` | Código de país ISO |
| `LanguageCode` | `pt` | Código de idioma |
| `ProviderCode` | `twilio` | Proveedor de mensajería |
| `ChannelTypeCode` | `sms` | Canal de entrega |
| `Version` | `v1` | Versión de la plantilla |

## Listar configuraciones de plantillas

{{if SDK == "csharp"}}
```csharp
var resultado = await client.Templates.GetListAsync(new GetTemplateConfigurationsRequest
{
    FilterText = "invoice",
    SkipCount = 0,
    MaxResultCount = 20
});

foreach (var config in resultado.Items)
{
    Console.WriteLine($"{config.Id} — {config.ValidationStatus}");
}
```
{{end}}

## Validar que existe una plantilla

Antes de enviar, puedes verificar que existe una plantilla configurada para los parámetros dados:

{{if SDK == "csharp"}}
```csharp
var validacion = await client.Templates.ValidateAsync(new TemplateConfigurationValidationRequest
{
    Application = "myapp",
    Country = "br",
    Language = "pt-BR",
    NotificationType = "trans",
    NotificationSubtype = "invoice"
});

if (validacion.IsValid)
{
    Console.WriteLine("Plantilla encontrada. Se puede enviar.");
}
else
{
    Console.WriteLine($"No se encontró plantilla: {validacion.ValidationMessage}");
}
```
{{end}}

### Respuesta de validación

{{if SDK == "csharp"}}
```csharp
public class TemplateConfigurationValidationResponse
{
    public bool IsValid { get; set; }
    public string? ValidationMessage { get; set; }
}
```
{{end}}

## Vista previa del mensaje

Genera el mensaje final con las variables sustituidas, sin enviarlo:

{{if SDK == "csharp"}}
```csharp
var preview = await client.Templates.GetPreviewAsync(new GetTemplateConfigurationsRequest
{
    Application = "myapp",
    Country = "br",
    Language = "pt-BR",
    NotificationType = "trans",
    NotificationSubtype = "invoice"
});

Console.WriteLine($"Mensaje de muestra: {preview.RenderedContent}");
```
{{end}}

## Obtener variables de una plantilla

Para saber qué variables acepta una plantilla antes de enviar:

{{if SDK == "csharp"}}
```csharp
// Por ID de configuración de plantilla
var variables = await client.Templates.GetTemplateVariableDefinitionsAsync(templateConfigId);

foreach (var variable in variables)
{
    Console.WriteLine($"Variable: {variable.Name} — Requerida: {variable.IsRequired}");
}
```
{{end}}

## Obtener variables agrupadas por sección

{{if SDK == "csharp"}}
```csharp
var variablesConSecciones = await client.Templates.GetTemplateVariablesWithSectionsAsync(
    templateConfigId,
    notificationSubtypeId);

foreach (var seccion in variablesConSecciones)
{
    Console.WriteLine($"Sección: {seccion.Section}");
    foreach (var variable in seccion.Variables)
        Console.WriteLine($"  - {variable.Name}");
}
```
{{end}}

## Obtener botones de una plantilla (WhatsApp)

Para plantillas de WhatsApp con botones de respuesta rápida o llamada a acción:

{{if SDK == "csharp"}}
```csharp
var botones = await client.Templates.GetTemplateButtonsAsync(templateId);

foreach (var boton in botones)
{
    Console.WriteLine($"Botón: {boton.Text} ({boton.Type})");
}
```
{{end}}

## Estados de aprobación de plantillas

| Valor enum | Descripción |
|---|---|
| `TemplateConfigurationApprovalState.Pending` | Pendiente de revisión |
| `TemplateConfigurationApprovalState.Approved` | Aprobada y lista para usar |
| `TemplateConfigurationApprovalState.Rejected` | Rechazada |

## Estados de validación

| Valor enum | Descripción |
|---|---|
| `TemplateConfigurationValidationStatus.Valid` | Plantilla válida |
| `TemplateConfigurationValidationStatus.Invalid` | Plantilla con errores |
| `TemplateConfigurationValidationStatus.NotValidated` | Aún no validada |

## Tipos de plantilla

| Valor enum | Descripción |
|---|---|
| `TemplateType.Sms` | SMS |
| `TemplateType.Email` | Correo electrónico |
| `TemplateType.WhatsApp` | WhatsApp |

## Tipos de header (WhatsApp)

| Valor enum | Descripción |
|---|---|
| `TemplateHeaderType.Text` | Header de texto |
| `TemplateHeaderType.Image` | Header de imagen |
| `TemplateHeaderType.Document` | Header de documento |
| `TemplateHeaderType.Video` | Header de video |
