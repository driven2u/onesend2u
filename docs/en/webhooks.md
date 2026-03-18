````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Webhooks

The `client.Webhooks` sub-client provides full CRUD operations for webhook subscriptions. Webhooks allow the OneSend2U platform to push real-time event notifications to your application's HTTP endpoint when message states change, errors occur, or user actions are detected.

## Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List webhooks with optional filters and pagination |
| `GetAsync(id)` | Get a webhook by ID |
| `CreateAsync(request)` | Create a new webhook |
| `UpdateAsync(id, request)` | Update an existing webhook |
| `DeleteAsync(id)` | Delete a webhook |
| `TestAsync(request)` | Send a test payload to a webhook endpoint |

## Webhook events

Webhooks are triggered by combinations of **event types** and **message states**:

### EventType enum

| Value | Description |
|---|---|
| `Status` | Message state changes (use with `Statuses` filter) |
| `UserAction` | User interactions such as button clicks in WhatsApp |
| `Consent` | Consent-related events |
| `ErrorProcessingMessages` | Errors during message processing |

### MessageProcessState (Statuses filter)

| Value | Description |
|---|---|
| `Initial` | Message created |
| `Pending` | Queued for sending |
| `Sending` | Being sent to provider |
| `Success` | Delivered to provider |
| `Error` | Delivery failed |
| `Discarded` | Discarded before sending |
| `NotConsented` | Recipient has not consented |
| `Unknown` | State undetermined |

## Creating a webhook

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Models.Enums;
using OneSend2U.Sdk.Webhooks.Models;

var webhook = await client.Webhooks.CreateAsync(new CreateWebhookRequest
{
    Name           = "Delivery Status Webhook",
    TargetEndPoint = "https://yourapp.example.com/webhooks/onesend2u",
    HttpMethod     = "POST",
    Status         = true,          // active
    NumberOfRetries = 3,
    RetryStrategy  = RetryStrategy.FixedDelay,
    RetryDelay     = 60,            // seconds between retries
    IsSigningEnabled = true,        // enable HMAC-SHA256 signature
    SigningSecretGracePeriodMinutes = 60,
    EventTypes = [EventType.Status],
    Statuses   = [MessageProcessState.Success, MessageProcessState.Error],
    DeploymentEnvironmentId = deploymentEnvId
});

Console.WriteLine($"Webhook created: {webhook.Id}");
Console.WriteLine($"Signing enabled: {webhook.IsSigningEnabled}");
```
{{end}}

### CreateWebhookRequest fields

| Field | Type | Default | Description |
|---|---|---|---|
| `Name` | `string` | *(required)* | Webhook name |
| `TargetEndPoint` | `string` | *(required)* | Target URL to receive events |
| `HttpMethod` | `string` | `"POST"` | HTTP method for delivery |
| `Headers` | `string?` | `null` | Custom headers as a JSON string |
| `NumberOfRetries` | `int` | `0` | Number of retry attempts on failure |
| `RetryStrategy` | `RetryStrategy` | `FixedDelay` | Retry timing strategy (see table below) |
| `RetryDelay` | `int` | `60` | Seconds between retries |
| `Timeout` | `int` | `0` | Request timeout in seconds (0 = no timeout) |
| `Status` | `bool` | `true` | Whether the webhook is active |
| `DeploymentEnvironmentId` | `Guid` | *(required)* | Environment scope |
| `ApplicationIds` | `List<Guid>` | `[]` | Limit to specific applications |
| `EventTypes` | `List<EventType>` | `[]` | Event types that trigger this webhook |
| `Statuses` | `List<MessageProcessState>` | `[]` | Message states that trigger this webhook |
| `UserActions` | `List<string>` | `[]` | User action identifiers |
| `IsSigningEnabled` | `bool` | `false` | Enable HMAC-SHA256 payload signing |
| `SigningSecretGracePeriodMinutes` | `int` | `60` | Grace period for secret rotation |

### Retry strategies

| Value | Description |
|---|---|
| `FixedDelay` | Fixed delay between retry attempts |
| `ExponentialBackoff` | Exponentially increasing delay |
| `LinearBackoff` | Linearly increasing delay |
| `Jitter` | Randomized delay |

## Updating a webhook

Pass the `ConcurrencyStamp` from the `GetAsync` response to prevent conflicting updates:

{{if SDK == "csharp"}}
```csharp
var existing = await client.Webhooks.GetAsync(webhookId);

await client.Webhooks.UpdateAsync(webhookId, new UpdateWebhookRequest
{
    Name             = existing.Name!,
    TargetEndPoint   = "https://yourapp.example.com/webhooks/v2",
    HttpMethod       = existing.HttpMethod ?? "POST",
    Status           = existing.Status,
    NumberOfRetries  = existing.NumberOfRetries,
    RetryStrategy    = existing.RetryStrategy,
    RetryDelay       = existing.RetryDelay,
    EventTypes       = existing.EventTypes,
    Statuses         = existing.Statuses,
    ConcurrencyStamp = existing.ConcurrencyStamp,
    DeploymentEnvironmentId = existing.DeploymentEnvironmentId
});
```
{{end}}

## Deleting a webhook

{{if SDK == "csharp"}}
```csharp
await client.Webhooks.DeleteAsync(webhookId);
Console.WriteLine("Webhook deleted.");
```
{{end}}

## Testing a webhook

Sends a synthetic test event to the webhook endpoint without creating a real notification.

{{if SDK == "csharp"}}
```csharp
var testResult = await client.Webhooks.TestAsync(new TestWebhookRequest
{
    WebhookId = webhookId
});

Console.WriteLine($"Test status code: {testResult.StatusCode}");
Console.WriteLine($"Response body: {testResult.ResponseBody}");
```
{{end}}

## Secret rotation

When you rotate a signing secret, the previous secret remains valid for the configured grace period (`SigningSecretGracePeriodMinutes`). During this window, validate incoming payloads against **both** secrets:

{{if SDK == "csharp"}}
```csharp
var webhook = await client.Webhooks.GetAsync(webhookId);

// During grace period, HasPreviousSigningSecret will be true.
// Validate against both secrets to avoid dropping valid events.
var result = WebhookSignatureValidator.Validate(
    payload:         requestBody,
    secrets:         [webhook.SigningSecret, previousSecret],
    signatureHeader: request.Headers["X-OneSend2U-Webhook-Signature"],
    timestampHeader: request.Headers["X-OneSend2U-Webhook-Timestamp"],
    webhookIdHeader: request.Headers["X-OneSend2U-Webhook-Id"]);
```
{{end}}

See [Webhook Verification](webhook-verification.md) for the full signature validation guide.
