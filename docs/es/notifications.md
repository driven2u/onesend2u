````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Notificaciones

El sub-cliente `client.Notifications` cubre 4 endpoints para enviar y consultar notificaciones.

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `SendAsync(request)` | POST | Envía una notificación multicanal |
| `GetListAsync(request)` | GET | Lista notificaciones con filtros y paginación |
| `GetAsync(id)` | GET | Obtiene una notificación por ID |
| `GetWithDetailsAsync(id)` | GET | Obtiene una notificación con todas las propiedades de navegación |

## Modelo de solicitud de envío

{{if SDK == "csharp"}}
```csharp
public class SendNotificationRequest
{
    // Identificador único de la transacción. Requerido. Máx. 100 caracteres.
    public string TransactionId { get; set; }

    // Código de aplicación. Requerido. Máx. 10 caracteres.
    public string Application { get; set; }

    // Código de país (ISO 3166-1 alpha-2). Requerido. Máx. 2 caracteres.
    public string Country { get; set; }

    // Código de idioma (ej. "en", "pt-BR"). Requerido. Máx. 5 caracteres.
    public string Language { get; set; }

    // Código de tipo de notificación. Requerido. Máx. 10 caracteres.
    public string NotificationType { get; set; }

    // Código de subtipo de notificación. Requerido. Máx. 10 caracteres.
    public string NotificationSubtype { get; set; }

    // Canales destino. Opcional; si está vacío se usan todos los canales configurados en la plantilla.
    public List<TargetChannel> TargetChannels { get; set; }

    // Destinatarios. Requerido, al menos 1.
    public List<NotificationRecipient> Recipients { get; set; }

    // Variables de sustitución de plantilla. Una entrada por destinatario.
    public List<Dictionary<string, string>> TemplateVariables { get; set; }

    // Archivos adjuntos (contenido en Base64). Solo para canales que lo soporten (Email).
    public List<NotificationAttachment> Attachments { get; set; }

    // ID de mensaje externo para correlación con sistemas propios.
    public string? ExternalMessageId { get; set; }

    // Número de secuencia externo.
    public string? ExternalSequenceNumber { get; set; }
}
```
{{end}}

### Canales disponibles

| Valor | Canal |
|---|---|
| `"sms"` | Mensaje de texto SMS |
| `"email"` | Correo electrónico |
| `"whatsapp"` | WhatsApp Business |

## Enviar una notificación

{{if SDK == "csharp"}}
```csharp
var result = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId = Guid.NewGuid().ToString(),
    Application = "myapp",
    Country = "mx",
    Language = "es",
    NotificationType = "trans",
    NotificationSubtype = "welcome",
    Recipients =
    [
        new NotificationRecipient { Channel = "sms",   Recipient = "+5215512345678" },
        new NotificationRecipient { Channel = "email",  Recipient = "usuario@ejemplo.com" }
    ],
    TemplateVariables =
    [
        new Dictionary<string, string>
        {
            { "nombre", "María García" },
            { "codigo_verificacion", "847291" }
        }
    ]
});

Console.WriteLine($"Estado: {result.Status}");        // "Accepted"
Console.WriteLine($"Creado: {result.CreatedAt}");
```
{{end}}

### Respuesta de envío

{{if SDK == "csharp"}}
```csharp
public class SendNotificationResponse
{
    public string? TransactionId { get; set; }     // Echo del TransactionId enviado
    public string? Status { get; set; }            // Estado inicial ("Accepted")
    public string? StatusDetail { get; set; }      // Detalle del estado
    public DateTime CreatedAt { get; set; }        // Fecha de creación en el servidor
    public List<string>? Warnings { get; set; }    // Advertencias sobre canales no configurados
}
```
{{end}}

### Envío con adjunto (Email)

{{if SDK == "csharp"}}
```csharp
var result = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId = Guid.NewGuid().ToString(),
    Application = "myapp",
    Country = "ar",
    Language = "es",
    NotificationType = "trans",
    NotificationSubtype = "invoice",
    TargetChannels = [new TargetChannel { Channel = "email" }],
    Recipients = [new NotificationRecipient { Channel = "email", Recipient = "cliente@empresa.com" }],
    TemplateVariables = [new Dictionary<string, string> { { "numero_factura", "F-0001" } }],
    Attachments =
    [
        new NotificationAttachment
        {
            FileName = "factura-001.pdf",
            FileContent = Convert.ToBase64String(pdfBytes)
        }
    ]
});
```
{{end}}

## Consultar notificaciones

### Listar con filtros

{{if SDK == "csharp"}}
```csharp
var pagina = await client.Notifications.GetListAsync(new GetNotificationsRequest
{
    FilterText = "invoice",
    Status = NotificationStatus.Success,
    CreationTimeMin = DateTime.UtcNow.AddDays(-7),
    CreationTimeMax = DateTime.UtcNow,
    Sorting = "CreationTime desc",
    SkipCount = 0,
    MaxResultCount = 20
});

Console.WriteLine($"Total: {pagina.TotalCount}");
foreach (var notif in pagina.Items)
{
    Console.WriteLine($"{notif.Id} — {notif.Status} — {notif.CreationTime}");
}
```
{{end}}

### Obtener por ID

{{if SDK == "csharp"}}
```csharp
var notif = await client.Notifications.GetAsync(id);
Console.WriteLine($"Mensajes exitosos: {notif.NumberOfSuccessMessages}/{notif.NumberOfTotalMessages}");
```
{{end}}

### Obtener con propiedades de navegación

{{if SDK == "csharp"}}
```csharp
// Incluye datos expandidos: aplicación, tipo, subtipo, país, etc.
var detalle = await client.Notifications.GetWithDetailsAsync(id);
```
{{end}}

## Parámetros de filtro disponibles

| Parámetro | Tipo | Descripción |
|---|---|---|
| `FilterText` | `string?` | Búsqueda libre en múltiples campos |
| `Source` | `NotificationSource?` | Origen: `CPaaS` o `Api` |
| `Language` | `string?` | Código de idioma |
| `Status` | `NotificationStatus?` | `Sending`, `Success`, `Error` |
| `ApplicationId` | `Guid?` | Filtrar por aplicación |
| `NotificationTypeId` | `Guid?` | Filtrar por tipo |
| `NotificationSubtypeId` | `Guid?` | Filtrar por subtipo |
| `CountryId` | `Guid?` | Filtrar por país |
| `ChannelTypeId` | `Guid?` | Filtrar por canal |
| `CreationTimeMin` | `DateTime?` | Fecha de creación mínima |
| `CreationTimeMax` | `DateTime?` | Fecha de creación máxima |
| `Sorting` | `string?` | Expresión de ordenamiento (ej. `"CreationTime desc"`) |
| `SkipCount` | `int` | Elementos a omitir (paginación) |
| `MaxResultCount` | `int` | Máximo de elementos a retornar (por defecto: 10) |

## Resultado paginado

Todos los métodos `GetListAsync` retornan `PagedResult<T>`:

{{if SDK == "csharp"}}
```csharp
public class PagedResult<T>
{
    public List<T> Items { get; set; }
    public long TotalCount { get; set; }
}
```
{{end}}
