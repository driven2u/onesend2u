````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Webhook Verification

When signing is enabled on a webhook, OneSend2U computes an HMAC-SHA256 signature over each payload and sends it in the request headers. Verifying this signature proves that the payload originated from OneSend2U and has not been tampered with.

## Signature headers

OneSend2U sends three headers with every signed webhook delivery:

| Header | Description |
|---|---|
| `X-OneSend2U-Webhook-Id` | The webhook ID (GUID without dashes) |
| `X-OneSend2U-Webhook-Timestamp` | Unix timestamp (seconds) when the payload was signed |
| `X-OneSend2U-Webhook-Signature` | `v1={hex-encoded-HMAC-SHA256}` |

## How signatures are computed

The signed payload is constructed by concatenating the webhook ID, timestamp, and raw request body with dots as separators:

```
{webhookId}.{timestamp}.{rawBody}
```

The platform then computes `HMAC-SHA256(key=signingSecret, data=signedPayload)` and formats the result as `v1={lowercase-hex}`.

## Using the SDK validator

The `WebhookSignatureValidator` static class handles all parsing, timing tolerance, and constant-time comparison automatically.

### Single secret validation

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Webhooks;

var result = WebhookSignatureValidator.Validate(
    payload:         rawBodyString,
    secret:          "your-signing-secret",
    signatureHeader: request.Headers["X-OneSend2U-Webhook-Signature"],
    timestampHeader: request.Headers["X-OneSend2U-Webhook-Timestamp"],
    webhookIdHeader: request.Headers["X-OneSend2U-Webhook-Id"],
    toleranceSeconds: 300);   // default: 5-minute replay window

if (!result.IsValid)
{
    Console.WriteLine($"Validation failed: {result.Error} — {result.ErrorMessage}");
    return;
}

// Process the verified event
```
{{end}}

The `toleranceSeconds` parameter (default `300`) defines the maximum age of a timestamp. Requests older than this are rejected to prevent replay attacks.

### Multi-secret validation (secret rotation)

During secret rotation there is a grace period (`SigningSecretGracePeriodMinutes`) where the previous secret remains valid. Use the overload that accepts multiple secrets:

{{if SDK == "csharp"}}
```csharp
var result = WebhookSignatureValidator.Validate(
    payload:         rawBodyString,
    secrets:         [currentSecret, previousSecret],
    signatureHeader: request.Headers["X-OneSend2U-Webhook-Signature"],
    timestampHeader: request.Headers["X-OneSend2U-Webhook-Timestamp"],
    webhookIdHeader: request.Headers["X-OneSend2U-Webhook-Id"]);

if (result.IsValid)
    // Accept the event
```
{{end}}

Secrets are tried in order. If any secret matches, the result is valid.

### Boolean shorthand

For simpler cases where you only need a true/false result:

{{if SDK == "csharp"}}
```csharp
bool valid = WebhookSignatureValidator.IsValid(
    payload:         rawBodyString,
    secret:          "your-signing-secret",
    signatureHeader: signatureHeader,
    timestampHeader: timestampHeader,
    webhookIdHeader: webhookIdHeader);
```
{{end}}

## Validation error types

`WebhookSignatureValidationError` enum values:

| Value | Cause |
|---|---|
| `InvalidParameters` | One or more required parameters are null or empty |
| `InvalidTimestamp` | Timestamp header is not a valid Unix integer |
| `TimestampOutOfTolerance` | Timestamp is older than `toleranceSeconds` |
| `InvalidSignatureFormat` | Signature header does not start with `v1=` or contains invalid hex |
| `InvalidSignature` | HMAC does not match (payload may have been tampered with) |

## ASP.NET Core middleware example

A complete ASP.NET Core endpoint that validates the signature before processing:

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Webhooks;

app.MapPost("/webhooks/onesend2u", async (HttpRequest request) =>
{
    // Read the raw body (required for signature validation)
    request.EnableBuffering();
    using var reader = new StreamReader(request.Body, leaveOpen: true);
    var rawBody = await reader.ReadToEndAsync();
    request.Body.Position = 0;

    var signingSecret = "your-signing-secret-from-configuration";

    var result = WebhookSignatureValidator.Validate(
        payload:         rawBody,
        secret:          signingSecret,
        signatureHeader: request.Headers[WebhookSignatureValidator.SignatureHeaderName].ToString(),
        timestampHeader: request.Headers[WebhookSignatureValidator.TimestampHeaderName].ToString(),
        webhookIdHeader: request.Headers[WebhookSignatureValidator.WebhookIdHeaderName].ToString());

    if (!result.IsValid)
    {
        return Results.Problem(
            title:      "Invalid webhook signature",
            detail:     result.ErrorMessage,
            statusCode: 401);
    }

    // Deserialize and handle the event
    var payload = JsonSerializer.Deserialize<WebhookEvent>(rawBody);
    // ... handle payload ...

    return Results.Ok();
});
```
{{end}}

## Header name constants

The `WebhookSignatureValidator` class exposes header name constants to avoid hardcoding strings:

{{if SDK == "csharp"}}
```csharp
WebhookSignatureValidator.SignatureHeaderName  // "X-OneSend2U-Webhook-Signature"
WebhookSignatureValidator.TimestampHeaderName  // "X-OneSend2U-Webhook-Timestamp"
WebhookSignatureValidator.WebhookIdHeaderName  // "X-OneSend2U-Webhook-Id"
```
{{end}}

## Manual verification (without the SDK)

If you are implementing verification in another language or environment, follow these steps:

1. Extract the three headers: `X-OneSend2U-Webhook-Id`, `X-OneSend2U-Webhook-Timestamp`, `X-OneSend2U-Webhook-Signature`
2. Parse the timestamp as a Unix integer and verify it is within your tolerance window (±300 seconds recommended)
3. Build the signed payload string: `{webhookId}.{timestamp}.{rawBody}`
4. Compute `HMAC-SHA256` using your signing secret (UTF-8 encoded) over the signed payload string (UTF-8 encoded)
5. Hex-encode the HMAC output (lowercase)
6. Prepend `v1=` and compare using a constant-time comparison against the `X-OneSend2U-Webhook-Signature` header value
7. Reject the request if the values do not match or the timestamp is out of tolerance
