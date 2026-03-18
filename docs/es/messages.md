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
var resultado = await client.Messages.GetListAsync(new GetMessagesRequest
{
    MessageProcessState = MessageProcessState.Error,
    CreationTimeMin = DateTime.UtcNow.AddDays(-1),
    Sorting = "CreationTime desc",
    MaxResultCount = 50
});

Console.WriteLine($"Total con error: {resultado.TotalCount}");
foreach (var mensaje in resultado.Items)
{
    Console.WriteLine($"{mensaje.Id} — {mensaje.Destination} — {mensaje.MessageProcessState}");
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
// Incluye datos expandidos: aplicación, proveedor, canal, tipo, etc.
var detalle = await client.Messages.GetWithDetailsAsync(mensajeId);
```
{{end}}

## Modelo de respuesta

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

foreach (var msg in fallidos.Items)
{
    Console.WriteLine($"  ID: {msg.Id}");
    Console.WriteLine($"  Destino: {msg.Destination}");
    Console.WriteLine($"  Notificación: {msg.NotificationId}");
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
