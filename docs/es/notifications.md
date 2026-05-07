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

    // Código de región (alfanumérico, máx. 10 caracteres). Requerido.
    public string Region { get; set; }

    // Código de idioma (ej. "en", "pt-BR"). Requerido. Máx. 5 caracteres.
    public string Language { get; set; }

    // Código de tipo de notificación. Requerido. Máx. 10 caracteres.
    public string NotificationType { get; set; }

    // Código de subtipo de notificación. Requerido. Máx. 10 caracteres.
    public string NotificationSubtype { get; set; }

    // Canales destino. Opcional; si está vacío se usan todos los canales configurados en la plantilla.
    public List<TargetChannel> TargetChannels { get; set; }

    // Destinatarios. Condicional: al menos uno de Recipients o ContactGroupCodes es obligatorio.
    public List<NotificationRecipient> Recipients { get; set; }

    // Códigos de grupos de contacto a resolver en destinatarios. Condicional: al menos uno de Recipients o ContactGroupCodes es obligatorio.
    public List<ContactGroupTarget>? ContactGroupCodes { get; set; }

    // Variables de sustitución de plantilla. Una entrada por destinatario.
    public List<Dictionary<string, string>> TemplateVariables { get; set; }

    // Archivos adjuntos (contenido en Base64). Solo para canales que lo soporten (Email).
    public List<NotificationAttachment> Attachments { get; set; }

    // ID de mensaje externo para correlación con sistemas propios.
    public string? ExternalMessageId { get; set; }

    // Número de secuencia externo.
    public string? ExternalSequenceNumber { get; set; }

    // Overrides de remitente por canal. Claves admitidas: "sms", "email", "whatsapp"
    // (case-insensitive). Cada entrada lleva opcionalmente Address y/o Name.
    // Ver "Override de remitente".
    public Dictionary<string, SenderOverride>? SenderOverrides { get; set; }
}

public class SenderOverride
{
    public string? Address { get; set; }
    public string? Name    { get; set; }
}
```
{{end}}

### Override de remitente

El remitente por defecto se configura a nivel de aplicación (pestaña Communication Channels). Para una petición concreta puedes sobrescribirlo con el mapa **`SenderOverrides`**, indexado por código de canal (`sms`, `email`, `whatsapp`, case-insensitive). Cada entrada lleva opcionalmente `Address` y/o `Name`. Reglas por canal:

| Canal | `Address` | `Name` |
|---|---|---|
| Email | Permitido. El dominio debe estar Verificado en SenderDomain para la cuenta externa de la Connection. En caso contrario se rechaza con `Cpaas:ApplicationChannel:00003`. La parte local (antes del `@`) es libre. | Permitido (texto libre). |
| SMS | Permitido. Debe pertenecer a la lista de senders disponibles de la Connection (`Connection.Senders.Where(IsAvailableForApp)`). En caso contrario se rechaza con `Cpaas:SenderOverride:00011`. | Permitido (best-effort). Solo se aplica si el país destino soporta remitentes alfanuméricos; si no, se descarta silenciosamente y el mensaje se envía con el número de teléfono. |
| WhatsApp | Permitido. Mismo control de pertenencia que SMS; en caso contrario se rechaza con `Cpaas:SenderOverride:00011`. | Rechazado con `Cpaas:SenderOverride:00001`. La Cloud API de WhatsApp asocia el nombre mostrado al número verificado, por lo que no es posible sobrescribirlo en runtime. |

Una entrada con `Address` y `Name` ambos null/blanco se rechaza con `Cpaas:SenderOverride:00012` (`SenderOverrideEmpty`).

El override se aplica a toda la notificación (todos los destinatarios). Prioridad de resolución (más específico gana): petición API → canal de la aplicación → remitente por defecto de la Connection.

```csharp
// Email — override del From para esta petición
req.SenderOverrides = new()
{
    ["email"] = new SenderOverride
    {
        Address = "security@tu-dominio-verificado.com",
        Name    = "Acme Security"
    }
};

// SMS — elige un sender concreto de la Connection y aplica un display name (best-effort)
req.SenderOverrides = new()
{
    ["sms"] = new SenderOverride
    {
        Address = "+34915794174",   // debe existir en Connection.Senders
        Name    = "Acme"            // descartado en países sin alpha sender
    }
};
```

| Código de error | Cuándo |
|---|---|
| `Cpaas:SenderOverride:00001` | Se envió una entrada `whatsapp` — WhatsApp no admite override por petición. |
| `Cpaas:SenderOverride:00011` | El `Address` de SMS no está en la lista de senders disponibles de la Connection. |
| `Cpaas:SenderOverride:00012` | Entrada con `Address` y `Name` ambos vacíos. |
| `Cpaas:SenderOverride:00013` | Clave de canal distinta de `sms` / `email` / `whatsapp`. |
| `Cpaas:ApplicationChannel:00003` | `Address` de Email cuyo dominio no está Verificado para la Connection resuelta. |
| `Cpaas:Integration:00116` | Formato de email inválido. |

#### Migración desde SDK 1.x

`SDK 1.x` exponía los campos planos `SenderAddress` / `SenderName` directamente en `SendNotificationRequest`. SDK `2.0.0` los reemplaza por `SenderOverrides`:

```csharp
// SDK 1.x
req.SenderAddress = "security@verificado.com";
req.SenderName    = "Acme Security";

// SDK 2.0+
req.SenderOverrides = new()
{
    ["email"] = new SenderOverride { Address = "security@verificado.com", Name = "Acme Security" }
};
```

### ContactGroupTarget

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `Code` | `string` | Sí | Código del grupo de contacto a resolver (máx. 10 caracteres) |
| `Channel` | `string?` | No | Filtro de canal (`"SMS"`, `"EMAIL"`, `"WHATSAPP"`). Si se omite, se usan todos los `TargetChannels`. |

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
    Region = "mx",
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

### Envío a Grupos de Contacto

Puedes enviar a todos los miembros de uno o más grupos de contacto especificando sus códigos. Los miembros se resuelven en el servidor — no necesitas consultar el grupo y construir la lista de destinatarios manualmente.

{{if SDK == "csharp"}}
```csharp
// Enviar a un grupo de contacto (todos los TargetChannels)
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    TargetChannels      = [new TargetChannel { Channel = "email" }],
    ContactGroupCodes   =
    [
        new ContactGroupTarget { Code = "VIP" }  // todos los TargetChannels
    ],
    TemplateVariables =
    [
        new Dictionary<string, string>
        {
            ["customer_name"] = "Cliente Preferente",
            ["invoice_number"] = "INV-001"
        }
    ]
});
```
{{end}}

También puedes combinar destinatarios explícitos con grupos de contacto. Los duplicados se eliminan automáticamente por `(canal, destino)`:

{{if SDK == "csharp"}}
```csharp
// Combinar destinatarios explícitos + grupos de contacto con filtro de canal
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    TargetChannels      =
    [
        new TargetChannel { Channel = "email" },
        new TargetChannel { Channel = "sms" }
    ],
    Recipients =
    [
        new NotificationRecipient { Channel = "email", Recipient = "cfo@example.com" }
    ],
    ContactGroupCodes =
    [
        new ContactGroupTarget { Code = "FINANCE", Channel = "EMAIL" },  // solo email
        new ContactGroupTarget { Code = "MGMT" }                         // todos los canales
    ],
    TemplateVariables =
    [
        new Dictionary<string, string>
        {
            ["customer_name"] = "Equipo",
            ["invoice_number"] = "INV-002"
        }
    ]
});
```
{{end}}

> **Nota:** Si un código de grupo de contacto no existe o no tiene miembros, se incluye una advertencia en la respuesta pero la notificación se procesa para los destinatarios válidos (éxito parcial). Los contactos sin los datos del canal requerido (ej: sin teléfono para SMS) se omiten silenciosamente.

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
    Region = "ar",
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

// GetListAsync retorna PagedResult<NotificationWithDetailsResponse>
// Cada item contiene la notificación y sus datos relacionados expandidos
Console.WriteLine($"Total: {pagina.TotalCount}");
foreach (var item in pagina.Items)
{
    // item.Notification contiene los campos base de la notificación
    Console.WriteLine($"{item.Notification.Id} — {item.Notification.Status} — {item.Notification.CreationTime}");
    // item.Application, item.NotificationType, etc. son LookupItem con Id/Name/Code/DisplayName
    Console.WriteLine($"  App: {item.Application?.DisplayName}");
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
// Retorna NotificationWithDetailsResponse con la notificación y sus datos relacionados
var detalle = await client.Notifications.GetWithDetailsAsync(id);

// Datos base de la notificación
Console.WriteLine($"ID: {detalle.Notification.Id}");
Console.WriteLine($"Estado: {detalle.Notification.Status}");
Console.WriteLine($"Mensajes exitosos: {detalle.Notification.NumberOfSuccessMessages}/{detalle.Notification.NumberOfTotalMessages}");

// Datos relacionados expandidos (cada uno es un LookupItem con Id/Name/Code/DisplayName)
Console.WriteLine($"Aplicación: {detalle.Application?.DisplayName}");
Console.WriteLine($"Tipo: {detalle.NotificationType?.DisplayName}");
Console.WriteLine($"Subtipo: {detalle.NotificationSubtype?.DisplayName}");
Console.WriteLine($"País: {detalle.Country?.Code}");
Console.WriteLine($"Entorno: {detalle.DeploymentEnvironment?.Name}");

// ChannelTypes es una lista de LookupItem (puede tener SMS, Email, WhatsApp)
foreach (var canal in detalle.ChannelTypes)
    Console.WriteLine($"  Canal: {canal.DisplayName}");
```
{{end}}

### Modelo `NotificationWithDetailsResponse`

{{if SDK == "csharp"}}
```csharp
public class NotificationWithDetailsResponse
{
    // Datos base de la notificación
    public NotificationResponse Notification { get; set; }

    // Datos relacionados expandidos
    public LookupItem Application { get; set; }
    public LookupItem NotificationType { get; set; }
    public LookupItem NotificationSubtype { get; set; }
    public LookupItem Country { get; set; }   // El request body usa "Region" (código), pero la entidad se almacena como Country/CountryId
    public LookupItem DeploymentEnvironment { get; set; }
    public List<LookupItem> ChannelTypes { get; set; }
}

public class LookupItem
{
    public Guid Id { get; set; }
    public string? Name { get; set; }
    public string? Code { get; set; }
    public string? DisplayName { get; set; }
}

public class NotificationResponse
{
    public Guid Id { get; set; }
    public NotificationSource? Source { get; set; }
    public string? Language { get; set; }
    public NotificationStatus? Status { get; set; }
    public int NumberOfSuccessMessages { get; set; }
    public int NumberOfTotalMessages { get; set; }
    public Guid ApplicationId { get; set; }
    public Guid NotificationTypeId { get; set; }
    public Guid NotificationSubtypeId { get; set; }
    public Guid CountryId { get; set; }   // Campo de entidad; el request de envío usa códigos en "Region"
    public DateTime CreationTime { get; set; }
    public string? ConcurrencyStamp { get; set; }
}
```
{{end}}

## Parámetros de filtro disponibles

| Parámetro | Tipo | Descripción |
|---|---|---|
| `FilterText` | `string?` | Búsqueda libre en múltiples campos |
| `Source` | `NotificationSource?` | Origen: `CPaaS` (1) o `API` (2) |
| `Language` | `string?` | Código de idioma |
| `Status` | `NotificationStatus?` | `Sending`, `Success`, `Error` |
| `ApplicationId` | `Guid?` | Filtrar por aplicación |
| `NotificationTypeId` | `Guid?` | Filtrar por tipo |
| `NotificationSubtypeId` | `Guid?` | Filtrar por subtipo |
| `CountryId` | `Guid?` | Filtrar por país |
| `ChannelTypeId` | `Guid?` | Filtrar por canal |
| `DeploymentEnvironmentId` | `Guid?` | Filtrar por entorno de despliegue |
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
