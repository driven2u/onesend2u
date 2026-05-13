````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Error Handling

The SDK uses a typed exception hierarchy so you can catch specific error categories and respond appropriately.

## Exception hierarchy

```
OneSend2UException                    (base)
├── OneSend2UApiException             (HTTP errors from the server)
│   └── OneSend2URateLimitException   (HTTP 429 Too Many Requests)
└── OneSend2UValidationException      (client-side validation before HTTP call)
```

All exceptions inherit from `OneSend2UException`, which inherits from `Exception`.

### OneSend2UApiException

Thrown when the API returns an HTTP error response (4xx or 5xx).

| Property | Type | Description |
|---|---|---|
| `StatusCode` | `HttpStatusCode` | HTTP status code (e.g., `403`, `404`, `500`) |
| `ErrorCode` | `string?` | Error code from the response body |
| `Details` | `string?` | Detailed error information |
| `ValidationErrors` | `IReadOnlyList<ValidationError>?` | Field-level validation errors (from 422 responses) |
| `RateLimit` | `RateLimitInfo?` | Rate limit headers parsed from the response |
| `ResponseBody` | `string?` | Raw response body |

### OneSend2URateLimitException

A subclass of `OneSend2UApiException` thrown specifically for HTTP 429 responses.

| Property | Type | Description |
|---|---|---|
| `RetryAfterSeconds` | `int?` | Seconds to wait before retrying (from `Retry-After` header) |

### OneSend2UValidationException

Thrown by the SDK before making an HTTP call when required fields are missing or values exceed limits (e.g., `TransactionId` exceeds 100 characters).

## HTTP status codes

| Status | Exception | Common cause |
|---|---|---|
| `400 Bad Request` | `OneSend2UApiException` | Malformed request body |
| `401 Unauthorized` | `OneSend2UApiException` | Missing or invalid `X-API-Key` |
| `403 Forbidden` | `OneSend2UApiException` | Valid API key but insufficient permissions |
| `404 Not Found` | `OneSend2UApiException` | Entity does not exist or wrong tenant |
| `409 Conflict` | `OneSend2UApiException` | Concurrency conflict (stale `ConcurrencyStamp`) |
| `422 Unprocessable Entity` | `OneSend2UApiException` | Server-side validation failure; check `ValidationErrors` |
| `429 Too Many Requests` | `OneSend2URateLimitException` | Rate limit exceeded; check `RetryAfterSeconds` |
| `500 Internal Server Error` | `OneSend2UApiException` | Unexpected server error |

## Error handling example

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Exceptions;
using OneSend2U.Sdk.Models.Enums;
using OneSend2U.Sdk.Notifications.Models;

try
{
    var response = await client.Notifications.SendAsync(new SendNotificationRequest
    {
        TransactionId       = Guid.NewGuid().ToString(),
        Application         = "billing",
        Region              = "us",
        Language            = "en",
        NotificationType    = "trans",
        NotificationSubtype = "invoice",
        Recipients          = [new NotificationRecipient { Channel = Channel.Sms, Recipient = "+15550001234" }]
    });

    Console.WriteLine($"Sent: {response.Status}");
}
catch (OneSend2UValidationException ex)
{
    // Client-side validation failed — fix the request before retrying
    Console.Error.WriteLine($"Request validation failed: {ex.Message}");
}
catch (OneSend2URateLimitException ex)
{
    // Rate limit hit — wait and retry
    var waitSeconds = ex.RetryAfterSeconds ?? 60;
    Console.Error.WriteLine($"Rate limited. Retry after {waitSeconds}s.");
    await Task.Delay(TimeSpan.FromSeconds(waitSeconds));
    // retry logic here
}
catch (OneSend2UApiException ex) when (ex.StatusCode == System.Net.HttpStatusCode.Forbidden)
{
    // Permissions not assigned to the API key
    Console.Error.WriteLine($"Forbidden: {ex.ErrorCode} — {ex.Message}");
    Console.Error.WriteLine("Ensure the API key user has the required permissions.");
}
catch (OneSend2UApiException ex) when (ex.StatusCode == System.Net.HttpStatusCode.UnprocessableEntity)
{
    // Server-side validation errors
    Console.Error.WriteLine($"Validation errors:");
    foreach (var error in ex.ValidationErrors ?? [])
        Console.Error.WriteLine($"  [{string.Join(", ", error.Members ?? [])}] {error.Message}");
}
catch (OneSend2UApiException ex)
{
    // Other API errors
    Console.Error.WriteLine($"API error {(int)ex.StatusCode}: {ex.Message}");
    Console.Error.WriteLine($"Error code: {ex.ErrorCode}");
    Console.Error.WriteLine($"Details: {ex.Details}");
}
catch (Exception ex)
{
    // Network errors, timeouts, etc.
    Console.Error.WriteLine($"Unexpected error: {ex.Message}");
}
```
{{end}}

## Rate limiting

The OneSend2U API enforces rate limits. When the limit is exceeded, the server returns HTTP 429 and the SDK throws `OneSend2URateLimitException`.

### Rate limit response headers

| Header | Description |
|---|---|
| `X-RateLimit-Limit` | Maximum requests allowed in the current window |
| `X-RateLimit-Remaining` | Requests remaining in the current window |
| `X-RateLimit-Reset` | UTC timestamp when the window resets |
| `Retry-After` | Seconds to wait before retrying |

The SDK parses these headers into `RateLimitInfo`:

{{if SDK == "csharp"}}
```csharp
catch (OneSend2URateLimitException ex)
{
    Console.Error.WriteLine($"Limit:     {ex.RateLimit?.Limit}");
    Console.Error.WriteLine($"Remaining: {ex.RateLimit?.Remaining}");
    Console.Error.WriteLine($"Reset:     {ex.RateLimit?.Reset:O}");
    Console.Error.WriteLine($"Retry in:  {ex.RetryAfterSeconds}s");
}
```
{{end}}

## Built-in resilience (DI mode)

When you register the SDK with `services.AddOneSend2U(...)`, the underlying `HttpClient` is configured with `AddStandardResilienceHandler()` from `Microsoft.Extensions.Http.Resilience`. This provides:

- **Retry**: Automatically retries transient failures (5xx, network errors) with exponential backoff
- **Circuit breaker**: Opens the circuit after repeated failures to prevent cascading load

These policies are transparent to application code. They apply **before** exceptions are thrown — the SDK will retry internally and only throw if all attempts are exhausted.

In **non-DI mode** (`new OneSend2UClient(options)`), no resilience policies are applied. Implement your own retry logic if needed.

### Manual retry with Polly (non-DI mode)

{{if SDK == "csharp"}}
```csharp
using Polly;
using Polly.Retry;

var retryPipeline = new ResiliencePipelineBuilder()
    .AddRetry(new RetryStrategyOptions
    {
        MaxRetryAttempts = 3,
        Delay            = TimeSpan.FromSeconds(2),
        BackoffType      = DelayBackoffType.Exponential,
        ShouldHandle     = new PredicateBuilder()
            .Handle<OneSend2URateLimitException>()
            .Handle<HttpRequestException>()
    })
    .Build();

await retryPipeline.ExecuteAsync(async ct =>
{
    var result = await client.Notifications.SendAsync(request);
    Console.WriteLine($"Sent: {result.Status}");
});
```
{{end}}
