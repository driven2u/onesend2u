# Registros de API

El sub-cliente `client.ApiLogs` permite consultar el historial de llamadas realizadas a la API de OneSend2U. Es la herramienta principal para auditoría, depuración y monitoreo de integraciones.

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista registros de API con filtros y paginación |
| `GetAsync(id)` | GET | Obtiene un registro de API por ID |

## ¿Qué se registra?

Cada llamada al endpoint `/api/app/notifications/send` genera automáticamente un registro de API (`ApiLog`) con la siguiente información:

| Campo | Descripción |
|---|---|
| `CorrelationId` | ID de correlación para rastreo distribuido |
| `TransactionId` | ID de transacción enviado por el cliente |
| `ApplicationCode` | Código de aplicación del request |
| `CountryCode` | Código de país del request |
| `NotificationTypeCode` | Tipo de notificación del request |
| `NotificationSubtypeCode` | Subtipo de notificación del request |
| `LanguageCode` | Idioma del request |
| `Source` | Origen (`Api` para llamadas del SDK) |
| `Endpoint` | Endpoint de API que fue llamado |
| `HttpMethod` | Método HTTP utilizado |
| `ClientIp` | Dirección IP del cliente |
| `RequestPayload` | Payload JSON del request original |
| `Status` | Estado del procesamiento |
| `StatusDetail` | Detalle del estado |
| `MessagesCreated` | Cantidad de mensajes generados |
| `MessagesSuccess` | Cantidad de mensajes enviados exitosamente |
| `MessagesError` | Cantidad de mensajes con error |
| `ReceivedAt` | Fecha y hora de recepción |
| `QueuedAt` | Fecha y hora de encolamiento |
| `CompletedAt` | Fecha y hora de finalización |
| `DurationMs` | Duración total en milisegundos |
| `StatusHistory` | Historial de cambios de estado |

## Estados del registro

| Valor enum | Valor numérico | Descripción |
|---|---|---|
| `NotificationApiLogStatus.Received` | 1 | Request recibido |
| `NotificationApiLogStatus.ValidationError` | 2 | Error de validación del request |
| `NotificationApiLogStatus.Queued` | 3 | Request encolado para procesamiento asíncrono |
| `NotificationApiLogStatus.OK` | 10 | Procesado exitosamente |
| `NotificationApiLogStatus.Warning` | 11 | Procesado con advertencias |
| `NotificationApiLogStatus.Error` | 12 | Procesamiento fallido |

## Listar registros de API

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.ApiLogs.Models;
using OneSend2U.Sdk.Models.Enums;

var resultado = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
{
    Status = NotificationApiLogStatus.Error,
    ReceivedAtMin = DateTime.UtcNow.AddHours(-24),
    ReceivedAtMax = DateTime.UtcNow,
    Sorting = "ReceivedAt desc",
    MaxResultCount = 50
});

Console.WriteLine($"Total errores en las últimas 24h: {resultado.TotalCount}");
foreach (var log in resultado.Items)
{
    Console.WriteLine($"[{log.ReceivedAt:HH:mm:ss}] {log.TransactionId} — {log.Status} — {log.StatusDetail}");
}
```
{{end}}

## Obtener un registro por ID

{{if SDK == "csharp"}}
```csharp
var log = await client.ApiLogs.GetAsync(logId);

Console.WriteLine($"Transaction ID: {log.TransactionId}");
Console.WriteLine($"Estado: {log.Status}");
Console.WriteLine($"Mensajes creados: {log.MessagesCreated}");
Console.WriteLine($"Mensajes exitosos: {log.MessagesSuccess}");
Console.WriteLine($"Mensajes con error: {log.MessagesError}");
Console.WriteLine($"Duración: {log.DurationMs}ms");

// Historial de estados
foreach (var entrada in log.StatusHistory)
{
    Console.WriteLine($"  {entrada.ChangedAt}: {entrada.Status} — {entrada.Detail}");
}
```
{{end}}

## Parámetros de filtro disponibles

| Parámetro | Tipo | Descripción |
|---|---|---|
| `FilterText` | `string?` | Búsqueda libre en múltiples campos |
| `CorrelationId` | `string?` | Filtrar por ID de correlación |
| `TransactionId` | `string?` | Filtrar por ID de transacción |
| `ApplicationCode` | `string?` | Filtrar por código de aplicación |
| `CountryCode` | `string?` | Filtrar por código de país |
| `NotificationTypeCode` | `string?` | Filtrar por tipo de notificación |
| `NotificationSubtypeCode` | `string?` | Filtrar por subtipo |
| `LanguageCode` | `string?` | Filtrar por idioma |
| `Source` | `NotificationSource?` | Filtrar por origen (`Api`, `CPaaS`) |
| `Status` | `NotificationApiLogStatus?` | Filtrar por estado del log |
| `StatusDetail` | `string?` | Filtrar por detalle del estado |
| `ExternalMessageId` | `string?` | Filtrar por ID de mensaje externo |
| `ExternalSequenceNumber` | `string?` | Filtrar por número de secuencia externo |
| `DeploymentEnvironmentId` | `Guid?` | Filtrar por entorno de despliegue |
| `NotificationId` | `Guid?` | Filtrar por notificación generada |
| `ReceivedAtMin` | `DateTime?` | Fecha de recepción mínima |
| `ReceivedAtMax` | `DateTime?` | Fecha de recepción máxima |
| `Sorting` | `string?` | Expresión de ordenamiento (ej. `"ReceivedAt desc"`) |
| `SkipCount` | `int` | Elementos a omitir (paginación) |
| `MaxResultCount` | `int` | Máximo a retornar (por defecto: 10) |

## Ejemplos de uso

### Buscar por Transaction ID

{{if SDK == "csharp"}}
```csharp
var logs = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
{
    TransactionId = "TXN-20260316-001",
    MaxResultCount = 10
});

if (logs.TotalCount == 0)
    Console.WriteLine("No se encontraron logs para ese Transaction ID.");
else
    Console.WriteLine($"Estado: {logs.Items[0].Status}");
```
{{end}}

### Monitoreo de errores recientes

{{if SDK == "csharp"}}
```csharp
// Obtener errores de la última hora para alertas
var erroresRecientes = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
{
    Status = NotificationApiLogStatus.Error,
    ReceivedAtMin = DateTime.UtcNow.AddHours(-1),
    Sorting = "ReceivedAt desc",
    MaxResultCount = 100
});

if (erroresRecientes.TotalCount > 10)
{
    Console.WriteLine($"ALERTA: {erroresRecientes.TotalCount} errores en la última hora.");
}
```
{{end}}

### Calcular tasa de éxito

{{if SDK == "csharp"}}
```csharp
var hoy = DateTime.UtcNow.Date;
var logsHoy = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
{
    ReceivedAtMin = hoy,
    ReceivedAtMax = hoy.AddDays(1),
    MaxResultCount = 1000
});

var exitosos = logsHoy.Items.Count(l => l.Status == NotificationApiLogStatus.OK);
var total = logsHoy.TotalCount;
var tasa = total > 0 ? (double)exitosos / total * 100 : 0;

Console.WriteLine($"Tasa de éxito hoy: {tasa:F1}% ({exitosos}/{total})");
```
{{end}}
