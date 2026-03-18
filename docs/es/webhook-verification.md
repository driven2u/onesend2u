````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Verificación de Firmas de Webhook

Cuando un webhook tiene la firma habilitada (`IsSigningEnabled = true`), OneSend2U firma cada payload enviado usando **HMAC-SHA256**. El SDK incluye la clase estática `WebhookSignatureValidator` para verificar estas firmas.

## Cómo funciona la firma

Para cada request firmado, OneSend2U envía tres headers HTTP:

| Header | Descripción |
|---|---|
| `X-OneSend2U-Webhook-Id` | ID del webhook (GUID sin guiones) |
| `X-OneSend2U-Webhook-Timestamp` | Timestamp Unix (segundos) del momento de firma |
| `X-OneSend2U-Webhook-Signature` | `v1={hmac-sha256-en-hex}` calculado sobre `{webhookId}.{timestamp}.{body}` |

El payload firmado tiene el formato:

```
{webhookId}.{timestamp}.{rawBody}
```

La firma HMAC-SHA256 se calcula con el secreto de firma del webhook como clave y este string como mensaje, y el resultado se codifica en hexadecimal con el prefijo `v1=`.

## Uso del SDK: `WebhookSignatureValidator.Validate()`

### Verificación con un único secreto

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Webhooks;

var result = WebhookSignatureValidator.Validate(
    payload:          rawBody,                 // Cuerpo del request como string
    secret:           webhookSecret,           // Secreto de firma (texto plano)
    signatureHeader:  request.Headers["X-OneSend2U-Webhook-Signature"],
    timestampHeader:  request.Headers["X-OneSend2U-Webhook-Timestamp"],
    webhookIdHeader:  request.Headers["X-OneSend2U-Webhook-Id"],
    toleranceSeconds: 300                      // Ventana de tolerancia (por defecto: 5 minutos)
);

if (!result.IsValid)
{
    Console.WriteLine($"Firma inválida: {result.Error} — {result.ErrorMessage}");
    return;
}

// Payload auténtico, procesar...
```
{{end}}

### Verificación simplificada con `IsValid()`

{{if SDK == "csharp"}}
```csharp
bool esValido = WebhookSignatureValidator.IsValid(
    payload:         rawBody,
    secret:          webhookSecret,
    signatureHeader: signatureHeader,
    timestampHeader: timestampHeader,
    webhookIdHeader: webhookIdHeader
);
```
{{end}}

### Verificación durante rotación de secretos

Durante la rotación de secretos, acepta tanto el secreto actual como el anterior:

{{if SDK == "csharp"}}
```csharp
var result = WebhookSignatureValidator.Validate(
    payload:         rawBody,
    secrets:         [currentSecret, previousSecret],  // Prueba en orden
    signatureHeader: signatureHeader,
    timestampHeader: timestampHeader,
    webhookIdHeader: webhookIdHeader
);
```
{{end}}

## Ejemplo: middleware ASP.NET Core

Implementación completa de un endpoint receptor de webhooks con verificación:

{{if SDK == "csharp"}}
```csharp
[ApiController]
[Route("webhooks")]
public class WebhookController : ControllerBase
{
    private readonly string _webhookSecret;

    public WebhookController(IConfiguration configuration)
    {
        _webhookSecret = configuration["OneSend2U:WebhookSecret"]!;
    }

    [HttpPost("onesend2u")]
    public async Task<IActionResult> Receive()
    {
        // Leer el cuerpo raw (importante: usar EnableBuffering si se lee más de una vez)
        Request.EnableBuffering();
        using var reader = new StreamReader(Request.Body, leaveOpen: true);
        var rawBody = await reader.ReadToEndAsync();
        Request.Body.Position = 0;

        // Leer headers de firma
        var signatureHeader = Request.Headers[WebhookSignatureValidator.SignatureHeaderName].ToString();
        var timestampHeader = Request.Headers[WebhookSignatureValidator.TimestampHeaderName].ToString();
        var webhookIdHeader = Request.Headers[WebhookSignatureValidator.WebhookIdHeaderName].ToString();

        // Verificar la firma
        var result = WebhookSignatureValidator.Validate(
            payload:         rawBody,
            secret:          _webhookSecret,
            signatureHeader: signatureHeader,
            timestampHeader: timestampHeader,
            webhookIdHeader: webhookIdHeader
        );

        if (!result.IsValid)
        {
            // Registrar el intento de acceso inválido
            return Unauthorized(new { error = result.ErrorMessage });
        }

        // Deserializar y procesar el evento
        var evento = JsonSerializer.Deserialize<WebhookEvent>(rawBody);
        await ProcessEventAsync(evento);

        return Ok();
    }
}
```
{{end}}

## Nombres de los headers como constantes

El SDK expone los nombres de los headers como constantes:

{{if SDK == "csharp"}}
```csharp
WebhookSignatureValidator.SignatureHeaderName  // "X-OneSend2U-Webhook-Signature"
WebhookSignatureValidator.TimestampHeaderName  // "X-OneSend2U-Webhook-Timestamp"
WebhookSignatureValidator.WebhookIdHeaderName  // "X-OneSend2U-Webhook-Id"
```
{{end}}

## Resultado de validación

{{if SDK == "csharp"}}
```csharp
public class WebhookSignatureValidationResult
{
    public bool IsValid { get; }
    public WebhookSignatureValidationError? Error { get; }
    public string? ErrorMessage { get; }
}
```
{{end}}

### Posibles errores

| Valor enum | Descripción | Causa común |
|---|---|---|
| `InvalidParameters` | Parámetros nulos o vacíos | Falta algún header o el secreto |
| `InvalidTimestamp` | Timestamp no es un Unix timestamp válido | Header `X-OneSend2U-Webhook-Timestamp` malformado |
| `TimestampOutOfTolerance` | Timestamp fuera de la ventana de tolerancia | Request demasiado antiguo (> 5 min) o ataque de replay |
| `InvalidSignatureFormat` | Formato de firma incorrecto | No comienza con `v1=` o contiene caracteres hexadecimales inválidos |
| `InvalidSignature` | La firma no coincide | Secreto incorrecto o payload modificado |

## Verificación manual sin el SDK

Si no puedes usar el SDK, puedes verificar la firma manualmente:

```python
# Python
import hmac
import hashlib
import time

def verify_signature(payload: str, secret: str, signature: str, timestamp: str, webhook_id: str, tolerance: int = 300) -> bool:
    # Verificar antigüedad del timestamp
    now = int(time.time())
    if abs(now - int(timestamp)) > tolerance:
        return False

    # Construir el payload firmado
    signed_payload = f"{webhook_id}.{timestamp}.{payload}"

    # Calcular HMAC-SHA256
    expected = hmac.new(
        secret.encode("utf-8"),
        signed_payload.encode("utf-8"),
        hashlib.sha256
    ).hexdigest()

    # Comparar con tiempo constante
    received = signature.removeprefix("v1=")
    return hmac.compare_digest(expected, received)
```

```javascript
// Node.js
const crypto = require('crypto');

function verifySignature(payload, secret, signature, timestamp, webhookId, toleranceSeconds = 300) {
    const now = Math.floor(Date.now() / 1000);
    if (Math.abs(now - parseInt(timestamp)) > toleranceSeconds) return false;

    const signedPayload = `${webhookId}.${timestamp}.${payload}`;
    const expected = crypto
        .createHmac('sha256', secret)
        .update(signedPayload, 'utf8')
        .digest('hex');

    const received = signature.replace('v1=', '');
    return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(received));
}
```

## Consideraciones de seguridad

- **Usa siempre la ventana de tolerancia** (`toleranceSeconds`) para rechazar requests antiguos y evitar ataques de replay.
- **Nunca compares strings directamente** para verificar la firma — usa comparación en tiempo constante (`CryptographicOperations.FixedTimeEquals` en C#, `hmac.compare_digest` en Python, `crypto.timingSafeEqual` en Node.js).
- **Guarda el secreto de forma segura** — usa variables de entorno o un gestor de secretos (Azure Key Vault, AWS Secrets Manager), nunca lo incluyas en el código fuente.
- **Lee el cuerpo raw** antes de cualquier deserialización. El cálculo de la firma usa el cuerpo exactamente como fue recibido.
