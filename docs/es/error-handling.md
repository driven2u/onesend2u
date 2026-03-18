# Manejo de Errores y Reintentos

El SDK proporciona una jerarquía de excepciones tipadas para facilitar el manejo granular de errores.

## Jerarquía de excepciones

```
Exception
└── OneSend2UException                    // Base del SDK
    ├── OneSend2UValidationException      // Validación del lado del cliente (antes del HTTP call)
    └── OneSend2UApiException             // Respuesta HTTP de error del servidor
        └── OneSend2URateLimitException   // HTTP 429 Too Many Requests
```

### `OneSend2UException`

Clase base de todas las excepciones del SDK. No se lanza directamente.

### `OneSend2UValidationException`

Se lanza **antes** de realizar la llamada HTTP cuando el SDK detecta un problema en los parámetros del request (por ejemplo, un campo requerido vacío o una URL inválida). No consume cuota de rate limit.

{{if SDK == "csharp"}}
```csharp
public class OneSend2UValidationException : OneSend2UException
{
    public string FieldName { get; }  // Campo que falló la validación
    // Message heredado de Exception contiene la descripción del error
}
```
{{end}}

### `OneSend2UApiException`

Se lanza cuando el servidor responde con un código de estado HTTP de error (4xx, 5xx).

{{if SDK == "csharp"}}
```csharp
public class OneSend2UApiException : OneSend2UException
{
    public HttpStatusCode StatusCode { get; }                         // Código HTTP (400, 403, 404, etc.)
    public string? ErrorCode { get; }                                 // Código de error de la plataforma
    public string? Details { get; }                                   // Detalles del error
    public IReadOnlyList<ValidationError>? ValidationErrors { get; } // Errores de validación del servidor
    public RateLimitInfo? RateLimit { get; }                          // Info de rate limiting
    public string? ResponseBody { get; }                              // Cuerpo de la respuesta raw
}
```
{{end}}

### `OneSend2URateLimitException`

Subclase de `OneSend2UApiException` para respuestas HTTP 429. Incluye información sobre cuándo reintentar.

{{if SDK == "csharp"}}
```csharp
public class OneSend2URateLimitException : OneSend2UApiException
{
    public int? RetryAfterSeconds { get; }  // Segundos de espera del header Retry-After
}
```
{{end}}

## Códigos de estado HTTP comunes

| Código | Descripción | Excepción lanzada |
|---|---|---|
| 400 Bad Request | Request malformado o validación fallida | `OneSend2UApiException` |
| 401 Unauthorized | API Key inválida o ausente | `OneSend2UApiException` |
| 403 Forbidden | Permisos insuficientes para el recurso | `OneSend2UApiException` |
| 404 Not Found | Recurso no encontrado | `OneSend2UApiException` |
| 409 Conflict | Conflicto de concurrencia (ConcurrencyStamp desactualizado) | `OneSend2UApiException` |
| 422 Unprocessable Entity | Error de negocio / regla de dominio | `OneSend2UApiException` |
| 429 Too Many Requests | Rate limit excedido | `OneSend2URateLimitException` |
| 500 Internal Server Error | Error interno del servidor | `OneSend2UApiException` |

## Ejemplo de manejo de errores

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Exceptions;
using System.Net;

public async Task EnviarConManejoDeErroresAsync(SendNotificationRequest request)
{
    try
    {
        var resultado = await client.Notifications.SendAsync(request);
        Console.WriteLine($"Enviado: {resultado.Status}");
    }
    catch (OneSend2UValidationException ex)
    {
        // Error local antes de llamar a la API
        Console.WriteLine($"Campo inválido '{ex.FieldName}': {ex.Message}");
    }
    catch (OneSend2URateLimitException ex)
    {
        // HTTP 429 — esperar antes de reintentar
        var espera = ex.RetryAfterSeconds ?? 60;
        Console.WriteLine($"Rate limit alcanzado. Reintentar en {espera} segundos.");
        Console.WriteLine($"Límite: {ex.RateLimit?.Limit}, Restante: {ex.RateLimit?.Remaining}");
    }
    catch (OneSend2UApiException ex) when (ex.StatusCode == HttpStatusCode.Forbidden)
    {
        // HTTP 403 — permisos insuficientes
        Console.WriteLine("Permisos insuficientes. Verifica los permisos del API Key.");
        Console.WriteLine($"Error code: {ex.ErrorCode}");
    }
    catch (OneSend2UApiException ex) when (ex.StatusCode == HttpStatusCode.BadRequest)
    {
        // HTTP 400 — errores de validación del servidor
        Console.WriteLine($"Request inválido: {ex.Message}");
        if (ex.ValidationErrors != null)
        {
            foreach (var error in ex.ValidationErrors)
                Console.WriteLine($"  - [{string.Join(", ", error.Members ?? [])}]: {error.Message}");
        }
    }
    catch (OneSend2UApiException ex)
    {
        // Cualquier otro error HTTP
        Console.WriteLine($"Error de API {(int)ex.StatusCode}: {ex.Message}");
        Console.WriteLine($"Detalle: {ex.Details}");
        Console.WriteLine($"Body: {ex.ResponseBody}");
    }
}
```
{{end}}

## Rate Limiting

La API de OneSend2U aplica rate limiting por API Key. Los headers de respuesta informan el estado del límite actual:

| Header | Descripción |
|---|---|
| `X-RateLimit-Limit` | Máximo de requests permitidos en la ventana actual |
| `X-RateLimit-Remaining` | Requests restantes en la ventana actual |
| `X-RateLimit-Reset` | Timestamp UTC de reset de la ventana |
| `Retry-After` | Segundos de espera (presente en respuestas 429) |

El SDK parsea automáticamente estos headers y los expone en la propiedad `RateLimit` de la excepción:

{{if SDK == "csharp"}}
```csharp
catch (OneSend2URateLimitException ex)
{
    Console.WriteLine($"Límite: {ex.RateLimit?.Limit}");
    Console.WriteLine($"Restante: {ex.RateLimit?.Remaining}");
    Console.WriteLine($"Reset en: {ex.RateLimit?.Reset}");
    Console.WriteLine($"Reintentar en: {ex.RetryAfterSeconds}s");
}
```
{{end}}

## Resiliencia integrada (modo DI)

Cuando el cliente se registra con `services.AddOneSend2U()`, se configura automáticamente una política de resiliencia usando `Microsoft.Extensions.Http.Resilience`:

### Política de reintentos

- **Reintentos automáticos**: para errores transitorios (timeout, 5xx, errores de red)
- **Circuit breaker**: abre el circuito temporalmente si hay muchos errores consecutivos
- **No reintenta**: 4xx (excepto 429), errores de validación del cliente

> El modo **sin DI** (`new OneSend2UClient(options)`) **no incluye** políticas de resiliencia automáticas. Debes implementar los reintentos manualmente.

### Reintentos manuales con Polly (modo sin DI)

{{if SDK == "csharp"}}
```csharp
using Polly;
using Polly.Retry;

var retryPolicy = new ResiliencePipelineBuilder()
    .AddRetry(new RetryStrategyOptions
    {
        MaxRetryAttempts = 3,
        Delay = TimeSpan.FromSeconds(2),
        BackoffType = DelayBackoffType.Exponential,
        ShouldHandle = new PredicateBuilder()
            .Handle<OneSend2URateLimitException>()
            .Handle<HttpRequestException>()
    })
    .Build();

await retryPolicy.ExecuteAsync(async ct =>
{
    var result = await client.Notifications.SendAsync(request);
    Console.WriteLine($"Enviado: {result.Status}");
});
```
{{end}}

## Información de validación del servidor

Cuando el servidor devuelve errores de validación (HTTP 400), el SDK los expone en `ValidationErrors`:

{{if SDK == "csharp"}}
```csharp
catch (OneSend2UApiException ex) when (ex.StatusCode == HttpStatusCode.BadRequest)
{
    if (ex.ValidationErrors != null)
    {
        foreach (var error in ex.ValidationErrors)
        {
            var campos = string.Join(", ", error.Members ?? []);
            Console.WriteLine($"  Campo(s): {campos}");
            Console.WriteLine($"  Error: {error.Message}");
        }
    }
}
```
{{end}}
