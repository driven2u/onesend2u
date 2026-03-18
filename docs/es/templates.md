````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

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

Antes de enviar, puedes verificar que existe una plantilla configurada para los parámetros dados. La respuesta indica qué canales están configurados y cuáles no.

{{if SDK == "csharp"}}
```csharp
var validacion = await client.Templates.ValidateAsync(new TemplateConfigurationValidationRequest
{
    // Códigos de la aplicación, país, idioma, tipo y subtipo (todos requeridos)
    ApplicationCode = "myapp",
    CountryCode = "br",
    Language = "pt-BR",
    NotificationTypeCode = "trans",
    NotificationSubtypeCode = "invoice",
    DeploymentEnvironmentId = environmentId,
    // Canales a validar; si está vacío valida todos los configurados
    TargetChannels = [new TargetChannel { Channel = "sms" }, new TargetChannel { Channel = "email" }]
});

// Status: None = ninguno configurado, Partial = algunos, All = todos configurados
Console.WriteLine($"Estado: {validacion.Status}");
Console.WriteLine($"Canales configurados: {string.Join(", ", validacion.ConfiguredChannels)}");
Console.WriteLine($"Canales sin configurar: {string.Join(", ", validacion.UnconfiguredChannels)}");
```
{{end}}

### Solicitud de validación

{{if SDK == "csharp"}}
```csharp
public class TemplateConfigurationValidationRequest
{
    public string ApplicationCode { get; set; }           // Código de aplicación. Requerido.
    public string CountryCode { get; set; }               // Código de país ISO. Requerido.
    public string Language { get; set; }                  // Código de idioma. Requerido.
    public string NotificationTypeCode { get; set; }      // Código de tipo. Requerido.
    public string NotificationSubtypeCode { get; set; }   // Código de subtipo. Requerido.
    public Guid DeploymentEnvironmentId { get; set; }     // ID del entorno de despliegue.
    public List<TargetChannel> TargetChannels { get; set; } // Canales a validar.
}
```
{{end}}

### Respuesta de validación

{{if SDK == "csharp"}}
```csharp
public class TemplateConfigurationValidationResponse
{
    // None=ningún canal, Partial=algunos canales, All=todos los canales configurados
    public TemplateConfigurationValidationStatus Status { get; set; }
    public List<string> ConfiguredChannels { get; set; }     // Canales con plantilla configurada
    public List<string> UnconfiguredChannels { get; set; }   // Canales sin plantilla configurada
}
```
{{end}}

## Vista previa del mensaje

Genera el mensaje final con las variables sustituidas, sin enviarlo. Útil para verificar el contenido antes de enviar en producción.

{{if SDK == "csharp"}}
```csharp
var preview = await client.Templates.GetPreviewAsync(new TemplateConfigurationPreviewRequest
{
    ApplicationCode = "myapp",
    CountryCode = "br",
    Language = "pt-BR",
    NotificationTypeCode = "trans",
    NotificationSubtypeCode = "invoice",
    DeploymentEnvironmentId = environmentId,
    TargetChannels = [new TargetChannel { Channel = "sms" }]
});

// Datos de la configuración de plantilla usada
Console.WriteLine($"Transaction ID: {preview.TransactionId}");
Console.WriteLine($"App: {preview.ApplicationCode} — País: {preview.CountryCode}");

// preview.Recipients contiene una lista de destinatarios de muestra
foreach (var recipient in preview.Recipients)
    Console.WriteLine($"  Destinatario: {recipient.Channel} — {recipient.Destination}");

// preview.TargetChannels muestra el contenido renderizado por canal
foreach (var canal in preview.TargetChannels)
{
    Console.WriteLine($"  Canal: {canal.Channel}");
    Console.WriteLine($"  Contenido: {canal.RenderedBody}");
}

// preview.TemplateVariables muestra las variables de plantilla usadas
foreach (var variable in preview.TemplateVariables)
    Console.WriteLine($"  Variable: {variable.Key} = {variable.Value}");
```
{{end}}

### Respuesta de vista previa

{{if SDK == "csharp"}}
```csharp
public class TemplateConfigurationPreviewResponse
{
    public string? TransactionId { get; set; }
    public string? ApplicationCode { get; set; }
    public string? Language { get; set; }
    public string? CountryCode { get; set; }
    public string? NotificationTypeCode { get; set; }
    public string? NotificationSubtypeCode { get; set; }
    public List<PreviewChannel> TargetChannels { get; set; }        // Canales con contenido renderizado
    public List<PreviewRecipient> Recipients { get; set; }          // Destinatarios de muestra
    public List<PreviewTemplateVariable> TemplateVariables { get; set; } // Variables usadas
}

public class PreviewTemplateVariable
{
    public string Key { get; set; }    // Nombre de la variable
    public string Value { get; set; }  // Valor de ejemplo usado en el preview
}
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
| `TemplateConfigurationApprovalState.NotRequired` | No requiere aprobación |
| `TemplateConfigurationApprovalState.UnSubmitted` | No enviada para revisión |
| `TemplateConfigurationApprovalState.Appeal` | En proceso de apelación |
| `TemplateConfigurationApprovalState.Pending` | Pendiente de revisión |
| `TemplateConfigurationApprovalState.Approved` | Aprobada y lista para usar |
| `TemplateConfigurationApprovalState.Rejected` | Rechazada por el proveedor |
| `TemplateConfigurationApprovalState.Paused` | Pausada temporalmente |
| `TemplateConfigurationApprovalState.Disabled` | Deshabilitada |
| `TemplateConfigurationApprovalState.Deleted` | Eliminada |

## Estados de validación de configuración

`TemplateConfigurationValidationStatus` indica qué proporción de canales están configurados:

| Valor enum | Valor numérico | Descripción |
|---|---|---|
| `TemplateConfigurationValidationStatus.None` | 0 | Ningún canal configurado |
| `TemplateConfigurationValidationStatus.Partial` | 1 | Algunos canales configurados |
| `TemplateConfigurationValidationStatus.All` | 2 | Todos los canales configurados |

## Tipos de plantilla

| Valor enum | Descripción |
|---|---|
| `TemplateType.Default` | Plantilla estándar |
| `TemplateType.Carousel` | Plantilla tipo carrusel (WhatsApp) |
| `TemplateType.Coupon` | Plantilla tipo cupón (WhatsApp) |

## Tipos de header (WhatsApp)

| Valor enum | Descripción |
|---|---|
| `TemplateHeaderType.Text` | Header de texto |
| `TemplateHeaderType.Media` | Header con imagen, video o documento |
| `TemplateHeaderType.Location` | Header con ubicación geográfica |

## Tipos de botón (WhatsApp)

| Valor enum | Descripción |
|---|---|
| `TemplateButtonType.Attachment` | Archivo adjunto |
| `TemplateButtonType.Subscribe` | Suscripción |
| `TemplateButtonType.Unsubscribe` | Cancelación de suscripción |
| `TemplateButtonType.Link` | Enlace URL |
| `TemplateButtonType.Call` | Llamada telefónica |
| `TemplateButtonType.UserAction` | Acción de usuario personalizada |
