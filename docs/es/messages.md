````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Mensajes

El sub-cliente `client.Messages` permite rastrear el estado de los mensajes individuales que se generan al procesar una notificación. Cada notificación puede generar múltiples mensajes (uno por destinatario y por canal).

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista mensajes con filtros y paginación |
| `GetAsync(id)` | GET | Obtiene un mensaje por ID |
| `GetWithDetailsAsync(id)` | GET | Obtiene un mensaje con propiedades de navegación expandidas |

## Ciclo de vida de un mensaje

```
Notificación recibida
        │
        ▼
   [Initial]  ──── sin consentimiento ──▶  [NotConsented]
        │
        ▼
   [Pending]  ──── descartado ──────────▶  [Discarded]
        │
        ▼
   [Sending]
        │
     ┌──┴──┐
     ▼     ▼
[Success] [Error]
     │
     ▼
[Unknown]  (cuando el proveedor no reporta estado final)
```

### Estados del ciclo de vida

| Estado | Descripción |
|---|---|
| `Initial` | Mensaje creado, pendiente de procesamiento |
| `Discarded` | Descartado antes de enviar (ej. canal no configurado) |
| `NotConsented` | El destinatario no ha dado consentimiento |
| `Pending` | En cola para envío |
| `Sending` | Siendo enviado al proveedor |
| `Success` | Enviado exitosamente |
| `Error` | Falló el envío |
| `Unknown` | Estado no reportado por el proveedor |

## Listar mensajes

{{if SDK == "csharp"}}
```csharp
// GetListAsync retorna PagedResult<MessageWithDetailsResponse>
// Cada item contiene el mensaje base más sus datos relacionados expandidos
var resultado = await client.Messages.GetListAsync(new GetMessagesRequest
{
    MessageProcessState = MessageProcessState.Error,
    CreationTimeMin = DateTime.UtcNow.AddDays(-1),
    Sorting = "CreationTime desc",
    MaxResultCount = 50
});

Console.WriteLine($"Total con error: {resultado.TotalCount}");
foreach (var item in resultado.Items)
{
    // item.Message contiene los campos base del mensaje
    Console.WriteLine($"{item.Message.Id} — {item.Message.Destination} — {item.Message.MessageProcessState}");
    // item.Application, item.ChannelType, etc. son LookupItem con Id/Name/Code/DisplayName
    Console.WriteLine($"  App: {item.Application?.DisplayName}  Canal: {item.ChannelType?.DisplayName}");
}
```
{{end}}

## Obtener un mensaje por ID

{{if SDK == "csharp"}}
```csharp
var mensaje = await client.Messages.GetAsync(mensajeId);

Console.WriteLine($"Destino: {mensaje.Destination}");
Console.WriteLine($"Estado: {mensaje.MessageProcessState}");
Console.WriteLine($"Creado: {mensaje.CreationTime}");
Console.WriteLine($"Modificado: {mensaje.LastModificationTime}");
```
{{end}}

## Obtener con propiedades de navegación

{{if SDK == "csharp"}}
```csharp
// Retorna MessageWithDetailsResponse con el mensaje y sus datos relacionados
var detalle = await client.Messages.GetWithDetailsAsync(mensajeId);

// Datos base del mensaje
Console.WriteLine($"Destino: {detalle.Message.Destination}");
Console.WriteLine($"Estado: {detalle.Message.MessageProcessState}");
Console.WriteLine($"Creado: {detalle.Message.CreationTime}");

// Datos relacionados expandidos (LookupItem con Id/Name/Code/DisplayName)
Console.WriteLine($"Aplicación: {detalle.Application?.DisplayName}");
Console.WriteLine($"Canal: {detalle.ChannelType?.DisplayName}");
Console.WriteLine($"Proveedor: {detalle.Provider?.Name}");
Console.WriteLine($"Tipo: {detalle.NotificationType?.DisplayName}");
Console.WriteLine($"Subtipo: {detalle.NotificationSubtype?.DisplayName}");
```
{{end}}

### Modelo `MessageWithDetailsResponse`

{{if SDK == "csharp"}}
```csharp
public class MessageWithDetailsResponse
{
    // Datos base del mensaje
    public MessageResponse Message { get; set; }

    // Datos relacionados expandidos (LookupItem con Id/Name/Code/DisplayName)
    public LookupItem Application { get; set; }
    public LookupItem ChannelType { get; set; }
    public LookupItem NotificationType { get; set; }
    public LookupItem NotificationSubtype { get; set; }
    public LookupItem Country { get; set; }
    public LookupItem Provider { get; set; }
    public LookupItem DeploymentEnvironment { get; set; }
}
```
{{end}}

## Modelo de respuesta base

{{if SDK == "csharp"}}
```csharp
public class MessageResponse
{
    public Guid Id { get; set; }
    public string? Destination { get; set; }          // Dirección destino (teléfono, email, etc.)
    public string? TemplateVariables { get; set; }    // Variables JSON usadas en el mensaje
    public MessageProcessState? MessageProcessState { get; set; }
    public string? Language { get; set; }
    public NotificationSource? NotificationSource { get; set; }
    public Guid CountryId { get; set; }
    public Guid ApplicationId { get; set; }
    public Guid ChannelTypeId { get; set; }
    public Guid NotificationTypeId { get; set; }
    public Guid NotificationSubtypeId { get; set; }
    public Guid NotificationId { get; set; }          // Notificación padre
    public Guid ProviderId { get; set; }
    public Guid? MessageStateId { get; set; }
    public string? ConcurrencyStamp { get; set; }
    public DateTime CreationTime { get; set; }
    public DateTime? LastModificationTime { get; set; }
}
```
{{end}}

## Parámetros de filtro disponibles

| Parámetro | Tipo | Descripción |
|---|---|---|
| `FilterText` | `string?` | Búsqueda libre en múltiples campos |
| `Destination` | `string?` | Filtrar por dirección destino |
| `MessageProcessState` | `MessageProcessState?` | Estado del mensaje |
| `Language` | `string?` | Código de idioma |
| `NotificationSource` | `NotificationSource?` | Origen: `CPaaS` o `Api` |
| `CountryId` | `Guid?` | Filtrar por país |
| `ApplicationId` | `Guid?` | Filtrar por aplicación |
| `ChannelTypeId` | `Guid?` | Filtrar por canal (SMS, Email, WhatsApp) |
| `NotificationTypeId` | `Guid?` | Filtrar por tipo de notificación |
| `NotificationSubtypeId` | `Guid?` | Filtrar por subtipo |
| `ProviderId` | `Guid?` | Filtrar por proveedor |
| `DeploymentEnvironmentId` | `Guid?` | Filtrar por entorno |
| `NotificationId` | `Guid?` | Filtrar por notificación padre |
| `MessageStateId` | `Guid?` | Filtrar por estado de mensaje |
| `TemplateVariables` | `string?` | Filtrar por contenido de variables |
| `CreationTimeMin` | `DateTime?` | Fecha de creación mínima |
| `CreationTimeMax` | `DateTime?` | Fecha de creación máxima |
| `Sorting` | `string?` | Expresión de ordenamiento (ej. `"CreationTime desc"`) |
| `SkipCount` | `int` | Elementos a omitir (paginación) |
| `MaxResultCount` | `int` | Máximo a retornar (por defecto: 10) |

## Ejemplo: monitoreo de mensajes fallidos

{{if SDK == "csharp"}}
```csharp
// Obtener todos los mensajes con error de las últimas 24 horas
var fallidos = await client.Messages.GetListAsync(new GetMessagesRequest
{
    MessageProcessState = MessageProcessState.Error,
    CreationTimeMin = DateTime.UtcNow.AddHours(-24),
    Sorting = "CreationTime desc",
    MaxResultCount = 100
});

Console.WriteLine($"Mensajes fallidos (últimas 24h): {fallidos.TotalCount}");

foreach (var item in fallidos.Items)
{
    // Los campos del mensaje están en item.Message
    Console.WriteLine($"  ID: {item.Message.Id}");
    Console.WriteLine($"  Destino: {item.Message.Destination}");
    Console.WriteLine($"  Notificación: {item.Message.NotificationId}");
    Console.WriteLine();
}
```
{{end}}

## Relación con notificaciones

Una **notificación** es la solicitud de envío (el evento original). Los **mensajes** son las unidades de entrega individuales. La relación es:

```
Notificación (1) ──▶ Mensajes (N)
                      (uno por destinatario × canal)
```

Para obtener los mensajes de una notificación específica, usa el filtro `NotificationId`:

{{if SDK == "csharp"}}
```csharp
var mensajesDeNotif = await client.Messages.GetListAsync(new GetMessagesRequest
{
    NotificationId = notificationId,
    MaxResultCount = 50
});
```
{{end}}
