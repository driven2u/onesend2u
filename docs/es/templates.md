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
{InstanceCode}_{ApplicationCode}_{NotificationTypeCode}_{NotificationSubtypeCode}_{RegionCode}_{LanguageCode}_{ProviderCode}_{ChannelTypeCode}_v{Version}
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
| `RegionCode` | `br` | Código de región (alfanumérico, máx. 10 caracteres) |
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

foreach (var item in resultado.Items)
{
    // item es TemplateConfigurationWithDetailsResponse: TemplateConfiguration + LookupItems
    Console.WriteLine($"{item.TemplateConfiguration?.Id} — {item.TemplateConfiguration?.Name} — {item.TemplateConfiguration?.ApprovalState}");
    Console.WriteLine($"  Provider: {item.Provider?.Name}, País: {item.Country?.Code}, App: {item.Application?.Code}");
}
```
{{end}}

## Validar que existe una plantilla

Antes de enviar, puedes verificar que existe una plantilla configurada para los parámetros dados. La respuesta indica qué canales están configurados y cuáles no.

{{if SDK == "csharp"}}
```csharp
var validacion = await client.Templates.ValidateAsync(new TemplateConfigurationValidationRequest
{
    // Códigos de la aplicación, región, idioma, tipo y subtipo (todos requeridos)
    ApplicationCode = "myapp",
    RegionCode = "br",
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
    public string RegionCode { get; set; }                // Código de región (alfanumérico, máx. 10 caracteres). Requerido.
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

## Vista previa de configuración de plantilla

Devuelve datos de muestra (transaction ID generado, códigos, destinatarios y variables con valores de ejemplo) que puedes usar para enviar una notificación de prueba a partir de una configuración de plantilla. El método recibe directamente el **ID de la configuración** (no un objeto request).

`GetPreviewAsync(Guid templateConfigurationId)` devuelve `TemplateConfigurationPreviewResponse`:

| Campo | Tipo | Descripción |
|---|---|---|
| `TransactionId` | `string` | ID de transacción generado para esta vista previa |
| `ApplicationCode` | `string` | Código de aplicación |
| `Language` | `string` | Código de idioma |
| `RegionCode` | `string` | Código de región |
| `NotificationTypeCode` / `NotificationSubtypeCode` | `string` | Códigos de tipo/subtipo |
| `TargetChannels` | `List<PreviewChannel>` | Cada item solo expone `Channel` (código del canal) |
| `Recipients` | `List<PreviewRecipient>` | Cada item expone `Channel` y `Recipient` (dirección de muestra) |
| `TemplateVariables` | `List<PreviewTemplateVariable>` | Cada item expone `Key` y `Value` (valor de muestra) |

{{if SDK == "csharp"}}
```csharp
var preview = await client.Templates.GetPreviewAsync(templateConfigurationId);

Console.WriteLine($"Transaction ID: {preview.TransactionId}");
Console.WriteLine($"App: {preview.ApplicationCode} — Región: {preview.RegionCode}");

foreach (var canal in preview.TargetChannels)
    Console.WriteLine($"  Canal: {canal.Channel}");

foreach (var recipient in preview.Recipients)
    Console.WriteLine($"  {recipient.Channel}: {recipient.Recipient}");

foreach (var variable in preview.TemplateVariables)
    Console.WriteLine($"  {variable.Key} = {variable.Value}");
```
{{end}}

## Obtener variables de una plantilla

Para saber qué variables acepta una plantilla antes de enviar. Los valores de `Name` que devuelve este método son las claves a usar en `SendNotificationRequest.TemplateVariables`.

`GetTemplateVariableDefinitionsAsync(Guid templateId)` devuelve `List<TemplateVariableDefinitionResponse>` con: `Id`, `Name`, `SampleValue`, `TemplateId`.

{{if SDK == "csharp"}}
```csharp
var variables = await client.Templates.GetTemplateVariableDefinitionsAsync(templateId);

foreach (var variable in variables)
    Console.WriteLine($"Variable: {variable.Name}  (ejemplo: {variable.SampleValue})");
```
{{end}}

## Obtener variables con secciones

Devuelve las definiciones de variables enriquecidas con las secciones de la plantilla en las que aparecen (`Header`, `Body`, `Footer`, etc.).

`GetTemplateVariablesWithSectionsAsync(Guid templateConfigurationId, Guid providerId)` devuelve una lista plana `List<TemplateVariableDefinitionWithSectionsResponse>`. Cada item es un `TemplateVariableDefinitionResponse` con dos campos extra:

| Campo | Tipo | Descripción |
|---|---|---|
| `Sections` | `List<TemplateSection>?` | Secciones donde se usa la variable (`Subject`, `Header`, `Body`, `Footer`, `Buttons`) |
| `NotificationVariablesType` | `NotificationVariablesType` | `Template`, `Auto`, `Property`, `TemplateName`, `TemplateLanguage`, `Optional` |

{{if SDK == "csharp"}}
```csharp
var variables = await client.Templates.GetTemplateVariablesWithSectionsAsync(
    templateConfigurationId,
    providerId);

foreach (var v in variables)
{
    var secciones = string.Join(", ", v.Sections ?? []);
    Console.WriteLine($"{v.Name}  tipo={v.NotificationVariablesType}  secciones=[{secciones}]");
}
```
{{end}}

## Obtener botones de una plantilla (WhatsApp)

Devuelve `List<TemplateButtonResponse>`. Cada item tiene `Id`, `Label`, `Value`, `ButtonType`, `Order`, `TemplateId`, `ConcurrencyStamp`, además de la lista de variables que usa el botón.

{{if SDK == "csharp"}}
```csharp
var botones = await client.Templates.GetTemplateButtonsAsync(templateId);

foreach (var boton in botones)
    Console.WriteLine($"Botón: {boton.Label} ({boton.ButtonType}) → {boton.Value}");
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
