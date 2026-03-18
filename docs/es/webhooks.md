````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Webhooks

El sub-cliente `client.Webhooks` permite gestionar webhooks para recibir notificaciones en tiempo real sobre eventos de mensajería. Cubre 6 endpoints con operaciones CRUD completas más un endpoint de prueba.

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista webhooks registrados |
| `GetAsync(id)` | GET | Obtiene un webhook por ID |
| `CreateAsync(request)` | POST | Crea un nuevo webhook |
| `UpdateAsync(id, request)` | PUT | Actualiza un webhook existente |
| `DeleteAsync(id)` | DELETE | Elimina un webhook |
| `TestAsync(request)` | POST | Envía un payload de prueba a un endpoint |

## Crear un webhook

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Webhooks.Models;
using OneSend2U.Sdk.Models.Enums;

var webhook = await client.Webhooks.CreateAsync(new CreateWebhookRequest
{
    Name = "Mi Webhook de Producción",
    TargetEndPoint = "https://mi-app.com/webhooks/onesend2u",
    HttpMethod = "POST",
    Status = true,                                    // Activo
    DeploymentEnvironmentId = environmentId,
    ApplicationIds = [appId],
    EventTypes = [EventType.Status],                  // Eventos de cambio de estado
    Statuses = [MessageProcessState.Success, MessageProcessState.Error],
    NumberOfRetries = 3,
    RetryStrategy = RetryStrategy.ExponentialBackoff,
    RetryDelay = 60,
    Timeout = 30,
    IsSigningEnabled = true,                          // Habilitar firma HMAC-SHA256
    SigningSecretGracePeriodMinutes = 60
});

Console.WriteLine($"Webhook creado: {webhook.Id}");
Console.WriteLine($"Secret de firma: {webhook.SigningSecret}");
```
{{end}}

## Modelo de creación de webhook

{{if SDK == "csharp"}}
```csharp
public class CreateWebhookRequest
{
    public string Name { get; set; }                          // Nombre. Requerido.
    public string TargetEndPoint { get; set; }                // URL destino. Requerido.
    public string HttpMethod { get; set; }                    // "POST" (por defecto)
    public string? Headers { get; set; }                      // Headers personalizados (JSON)
    public int NumberOfRetries { get; set; }                  // Intentos de reenvío. Por defecto: 0.
    public RetryStrategy RetryStrategy { get; set; }          // Estrategia de reintento
    public int RetryDelay { get; set; }                       // Segundos entre reintentos. Por defecto: 60.
    public int Timeout { get; set; }                          // Timeout en segundos. Por defecto: 0.
    public bool Status { get; set; }                          // true = activo. Por defecto: true.
    public Guid DeploymentEnvironmentId { get; set; }         // Entorno de despliegue
    public List<Guid> ApplicationIds { get; set; }            // Aplicaciones a las que aplica
    public List<EventType> EventTypes { get; set; }           // Tipos de evento que activan el webhook
    public List<MessageProcessState> Statuses { get; set; }   // Estados de mensaje que activan el webhook
    public List<string> UserActions { get; set; }             // Acciones de usuario (ej. clicks en botones)
    public bool IsSigningEnabled { get; set; }                // Habilitar firma HMAC-SHA256
    public int SigningSecretGracePeriodMinutes { get; set; }   // Período de gracia de rotación (por defecto: 60)
}
```
{{end}}

## Tipos de eventos disponibles

| Valor enum | Descripción |
|---|---|
| `EventType.Consent` | Cambios en consentimientos de destinatarios |
| `EventType.Status` | Cambios de estado de mensajes |
| `EventType.UserAction` | Acciones de usuario (clics en botones, respuestas) |
| `EventType.ErrorProcessingMessages` | Errores durante el procesamiento de mensajes |

## Estrategias de reintento

| Valor enum | Descripción |
|---|---|
| `RetryStrategy.FixedDelay` | Intervalo fijo entre reintentos |
| `RetryStrategy.ExponentialBackoff` | Intervalo que aumenta exponencialmente |
| `RetryStrategy.LinearBackoff` | Intervalo que aumenta linealmente |
| `RetryStrategy.Jitter` | Intervalo aleatorio |

## Listar webhooks

{{if SDK == "csharp"}}
```csharp
var resultado = await client.Webhooks.GetListAsync(new GetWebhooksRequest
{
    SkipCount = 0,
    MaxResultCount = 20
});

foreach (var wh in resultado.Items)
{
    Console.WriteLine($"{wh.Name} — {wh.TargetEndPoint} — Activo: {wh.Status}");
}
```
{{end}}

## Obtener un webhook por ID

{{if SDK == "csharp"}}
```csharp
var webhook = await client.Webhooks.GetAsync(webhookId);

Console.WriteLine($"Nombre: {webhook.Name}");
Console.WriteLine($"URL: {webhook.TargetEndPoint}");
Console.WriteLine($"Firma habilitada: {webhook.IsSigningEnabled}");
Console.WriteLine($"Eventos: {string.Join(", ", webhook.EventTypes)}");
```
{{end}}

## Actualizar un webhook

Para actualizar es necesario incluir el `ConcurrencyStamp` del webhook actual (control de concurrencia optimista):

{{if SDK == "csharp"}}
```csharp
// Primero obtén el webhook actual
var actual = await client.Webhooks.GetAsync(webhookId);

// Luego actualiza
await client.Webhooks.UpdateAsync(webhookId, new UpdateWebhookRequest
{
    Name = actual.Name,
    TargetEndPoint = "https://mi-app.com/webhooks/nuevo-endpoint",
    HttpMethod = actual.HttpMethod,
    Status = actual.Status,
    DeploymentEnvironmentId = actual.DeploymentEnvironmentId,
    EventTypes = actual.EventTypes,
    Statuses = actual.Statuses,
    NumberOfRetries = actual.NumberOfRetries,
    RetryStrategy = actual.RetryStrategy,
    RetryDelay = actual.RetryDelay,
    ConcurrencyStamp = actual.ConcurrencyStamp   // Requerido para actualizaciones
});
```
{{end}}

## Eliminar un webhook

{{if SDK == "csharp"}}
```csharp
await client.Webhooks.DeleteAsync(webhookId);
Console.WriteLine("Webhook eliminado correctamente.");
```
{{end}}

## Probar un webhook

Envía un payload de prueba a un endpoint sin necesidad de crear el webhook:

{{if SDK == "csharp"}}
```csharp
var testResult = await client.Webhooks.TestAsync(new WebhookTestRequest
{
    TargetEndPoint = "https://mi-app.com/webhooks/test",
    HttpMethod = "POST"
});

Console.WriteLine($"Resultado de prueba: {testResult.StatusCode}");
Console.WriteLine($"Respuesta: {testResult.ResponseBody}");
```
{{end}}

## Rotación de secretos de firma

Cuando el secreto de firma de un webhook es rotado, el sistema mantiene el secreto anterior durante un período de gracia (`SigningSecretGracePeriodMinutes`). Durante este período, el webhook puede ser verificado con cualquiera de los dos secretos.

{{if SDK == "csharp"}}
```csharp
var webhook = await client.Webhooks.GetAsync(webhookId);

if (webhook.HasPreviousSigningSecret)
{
    // Durante el período de gracia, valida con ambos secretos
    var result = WebhookSignatureValidator.Validate(
        payload,
        secrets: [webhook.SigningSecret, previousSecret],
        signatureHeader,
        timestampHeader,
        webhookIdHeader);
}
```
{{end}}

Para más detalles sobre la verificación de firmas, consulta [Verificación de Firmas](webhook-verification.md).
