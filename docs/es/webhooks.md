````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Webhooks

El sub-cliente `client.Webhooks` permite gestionar webhooks para recibir notificaciones en tiempo real sobre eventos de mensajería. Cubre 7 endpoints con operaciones CRUD completas, un endpoint de prueba y rotación de secreto de firma.

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista webhooks registrados |
| `GetAsync(id)` | GET | Obtiene un webhook por ID |
| `CreateAsync(request)` | POST | Crea un nuevo webhook |
| `UpdateAsync(id, request)` | PUT | Actualiza un webhook existente |
| `DeleteAsync(id)` | DELETE | Elimina un webhook |
| `TestAsync(webhook)` | POST | Envía un payload de prueba usando un webhook existente |
| `RotateSigningSecretAsync(id)` | POST | Genera un nuevo secreto de firma HMAC-SHA256 |

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

Envía un payload de prueba usando un webhook ya registrado. El método recibe el objeto `WebhookResponse` completo (obtenido con `GetAsync`):

{{if SDK == "csharp"}}
```csharp
// Primero obtén el webhook que quieres probar
var webhook = await client.Webhooks.GetAsync(webhookId);

// Luego envía el payload de prueba al endpoint configurado en ese webhook
var testResult = await client.Webhooks.TestAsync(webhook);

Console.WriteLine($"HTTP Status: {testResult.StatusCode}");
Console.WriteLine($"Razón: {testResult.ReasonPhrase}");
Console.WriteLine($"Cuerpo (formateado): {testResult.FormattedContent}");
Console.WriteLine($"Cuerpo (raw): {testResult.RawContent}");

// Si hubo un error de conexión, ErrorMessage lo indica
if (testResult.ErrorMessage != null)
    Console.WriteLine($"Error: {testResult.ErrorMessage}");
```
{{end}}

### Respuesta de prueba de webhook

{{if SDK == "csharp"}}
```csharp
public class WebhookTestResponse
{
    public int? StatusCode { get; set; }          // Código HTTP de la respuesta del endpoint
    public string? ReasonPhrase { get; set; }     // Frase de estado HTTP (ej. "OK", "Not Found")
    public string? FormattedContent { get; set; } // Cuerpo de respuesta con formato legible
    public string? RawContent { get; set; }       // Cuerpo de respuesta sin procesar
    public string? ErrorMessage { get; set; }     // Mensaje de error si no se pudo conectar
}
```
{{end}}

## Modelo de respuesta del webhook

{{if SDK == "csharp"}}
```csharp
public class WebhookResponse
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string TargetEndPoint { get; set; }
    public string HttpMethod { get; set; }
    public string? Headers { get; set; }
    public string? Body { get; set; }                             // Cuerpo personalizado del payload
    public int NumberOfRetries { get; set; }
    public RetryStrategy RetryStrategy { get; set; }
    public int RetryDelay { get; set; }
    public int Timeout { get; set; }
    public bool Status { get; set; }
    public Guid DeploymentEnvironmentId { get; set; }
    public List<Guid> ApplicationIds { get; set; }
    public List<EventType> EventTypes { get; set; }
    public List<MessageProcessState> Statuses { get; set; }
    public List<string> UserActions { get; set; }
    public bool IsSigningEnabled { get; set; }
    public string? SigningSecret { get; set; }                // Secreto actual (cifrado en reposo)
    public bool HasPreviousSigningSecret { get; set; }        // true durante el período de gracia tras una rotación
    public DateTime? SigningSecretRotatedAt { get; set; }     // Última rotación
    public int SigningSecretGracePeriodMinutes { get; set; }
    public string? ConcurrencyStamp { get; set; }
}
```
{{end}}

## Rotar secreto de firma

Genera un nuevo secreto HMAC-SHA256 para un webhook. El secreto anterior sigue siendo válido durante el período de gracia (`SigningSecretGracePeriodMinutes`), lo que permite rotar sin interrumpir el servicio:

{{if SDK == "csharp"}}
```csharp
var rotated = await client.Webhooks.RotateSigningSecretAsync(webhookId);
Console.WriteLine($"Nuevo secreto: {rotated.SigningSecret}");
// El secreto anterior sigue siendo válido durante el período de gracia
```
{{end}}

## Rotación de secretos de firma — período de gracia

Cuando el secreto es rotado, el sistema acepta firmas generadas con cualquiera de los dos secretos durante el tiempo configurado en `SigningSecretGracePeriodMinutes`.

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
