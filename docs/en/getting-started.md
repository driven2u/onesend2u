````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Getting Started

## Prerequisites

Before you begin, you need:

- **.NET 10** or later
- An **API Key** — obtain one from the OneSend2U admin panel under **Settings → API Keys**
- Your **Tenant ID** — shown in the admin panel under **Settings → Tenant**

## Installation

{{if SDK == "csharp"}}
Install the NuGet package from NuGet.org:

```bash
dotnet add package OneSend2U.Sdk
```
{{end}}

## DI registration (recommended)

Add the client to your dependency injection container during startup. This mode includes automatic **retry** and **circuit breaker** resilience policies via `Microsoft.Extensions.Http.Resilience`.

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk;

// In Program.cs or your module setup:
services.AddOneSend2U(options =>
{
    options.BaseUrl  = "https://api.onesend2u.com";
    options.ApiKey   = "your-api-key";
    options.TenantId = "your-tenant-id";
});
```
{{end}}

### Configuration options

| Option | Type | Default | Description |
|---|---|---|---|
| `BaseUrl` | `string` | *(required)* | Base URL of the OneSend2U API |
| `ApiKey` | `string` | `""` | API key sent as `X-API-Key` header |
| `TenantId` | `string` | `""` | Tenant identifier sent as `X-Tenant-Id` header |
| `Timeout` | `TimeSpan` | `30s` | HTTP request timeout |

### Reading options from configuration

{{if SDK == "csharp"}}
```csharp
services.AddOneSend2U(options =>
    builder.Configuration.GetSection("OneSend2U").Bind(options));
```
{{end}}

```json
{
  "OneSend2U": {
    "BaseUrl": "https://api.onesend2u.com",
    "ApiKey": "your-api-key",
    "TenantId": "your-tenant-id"
  }
}
```

## Non-DI usage

For scripts, console applications, or situations where you do not have a DI container, instantiate the client directly. In this mode there are **no resilience policies**.

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk;

await using var client = new OneSend2UClient(new OneSend2UClientOptions
{
    BaseUrl  = "https://api.onesend2u.com",
    ApiKey   = "your-api-key",
    TenantId = "your-tenant-id"
});

// Use client.Notifications, client.Webhooks, etc.
```
{{end}}

Call `Dispose()` (or use `await using`) when done. In DI mode the lifetime is managed by the container.

## Your first notification

The following example shows a complete service that sends an SMS notification:

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk;
using OneSend2U.Sdk.Models.Enums;
using OneSend2U.Sdk.Notifications.Models;

public class InvoiceNotificationService(OneSend2UClient client)
{
    public async Task<SendNotificationResponse> NotifyCustomerAsync(
        string phoneNumber,
        string customerName,
        string invoiceNumber)
    {
        var response = await client.Notifications.SendAsync(new SendNotificationRequest
        {
            TransactionId       = Guid.NewGuid().ToString(),  // unique per call
            Application         = "billing",
            Region              = "us",
            Language            = "en",
            NotificationType    = "trans",
            NotificationSubtype = "invoice",
            Recipients =
            [
                new NotificationRecipient
                {
                    Channel   = Channel.Sms,
                    Recipient = phoneNumber
                }
            ],
            TemplateVariables =
            [
                new Dictionary<string, string>
                {
                    ["customer_name"]   = customerName,
                    ["invoice_number"]  = invoiceNumber
                }
            ]
        });

        Console.WriteLine($"Status: {response.Status}");

        if (response.Warnings?.Count > 0)
        {
            foreach (var warning in response.Warnings)
                Console.WriteLine($"Warning: {warning}");
        }

        return response;
    }
}
```
{{end}}

Inject the service and call it:

{{if SDK == "csharp"}}
```csharp
// Registration (Program.cs)
services.AddOneSend2U(options => { ... });
services.AddScoped<InvoiceNotificationService>();

// Usage
var svc = serviceProvider.GetRequiredService<InvoiceNotificationService>();
await svc.NotifyCustomerAsync("+15550001234", "Jane Doe", "INV-001");
```
{{end}}

## Using the client in a controller

With DI registration, inject `OneSend2UClient` directly into your services or controllers:

{{if SDK == "csharp"}}
```csharp
public class NotifyController(OneSend2UClient client) : ControllerBase
{
    [HttpPost("notify")]
    public async Task<IActionResult> Notify([FromBody] NotifyRequest request)
    {
        var result = await client.Notifications.SendAsync(/* ... */);
        return Ok(result);
    }
}
```
{{end}}

## Next steps

- [Notifications](notifications.md) — full send request model, querying
- [Templates](templates.md) — template validation and preview before sending
- [Webhooks](webhooks.md) — receive real-time delivery status callbacks
- [Error Handling](error-handling.md) — handling API errors and rate limits
