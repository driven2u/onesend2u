````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# OneSend2U SDK — Overview

OneSend2U is a CPaaS (Communications Platform as a Service) platform for sending multi-channel notifications — SMS, Email, and WhatsApp — through multiple providers (Twilio, Infobip, SMTP) from a single unified API.

The **OneSend2U .NET SDK** (`OneSend2U.Sdk`) is a lightweight NuGet package that gives any .NET application typed access to the full OneSend2U API surface. It requires only .NET 10 and the SDK package — no additional frameworks or dependencies.

## What the SDK provides

The SDK wraps **41 API endpoints** across 8 sub-clients, all accessible from a single `OneSend2UClient` entry point:

| Sub-client | Endpoints | Purpose |
|---|---|---|
| `client.Notifications` | 4 | Send and query notifications |
| `client.Templates` | 9 | Template configurations and content |
| `client.Messages` | 3 | Track individual message delivery |
| `client.Webhooks` | 7 | Manage webhook subscriptions |
| `client.ApiLogs` | 2 | Query API request/response logs |
| `client.Contacts` | 5 | Manage contact records |
| `client.ContactsGroups` | 6 | Manage contact groups |
| `client.Consents` | 6 | Manage recipient consents |

## Key features

- **Multi-channel**: SMS, Email, and WhatsApp from one API call
- **Multi-provider**: Twilio, Infobip, SMTP — the platform routes automatically
- **Webhooks**: Real-time delivery status callbacks with HMAC-SHA256 signature verification
- **Consent management**: Built-in support for regulatory compliance
- **Template system**: Dynamic variable substitution with preview and validation
- **Minimal dependencies**: Works in any .NET 10 application with no extra frameworks required

## Authentication

The SDK authenticates using API Keys. Every request sends two HTTP headers:

| Header | Description |
|---|---|
| `X-API-Key` | Your API key (obtain from the admin panel under API Keys) |
| `X-Tenant-Id` | Your tenant identifier |

**Sending notifications** works with any valid API Key. Reading templates, messages, webhooks, contacts, consents, and API logs requires the tenant admin to have assigned the corresponding permissions to the user linked to your API Key.

## Quick start

{{if SDK == "csharp"}}
```csharp
// 1. Install
// dotnet add package OneSend2U.Sdk

// 2. Register
services.AddOneSend2U(options =>
{
    options.BaseUrl = "https://api.onesend2u.com";
    options.ApiKey  = "your-api-key";
    options.TenantId = "your-tenant-id";
});

// 3. Send
var result = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "myapp",
    Country             = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    Recipients          = [new NotificationRecipient { Channel = "sms", Recipient = "+15550001234" }],
    TemplateVariables   = [new Dictionary<string, string> { ["customer_name"] = "Jane Doe" }]
});
```
{{end}}

## Documentation sections

- [Getting Started](getting-started.md) — Installation, configuration, and your first request
- [Notifications](notifications.md) — Sending and querying notifications
- [Templates](templates.md) — Template configurations, validation, and preview
- [Messages](messages.md) — Individual message tracking
- [Webhooks](webhooks.md) — Creating and managing webhook subscriptions
- [Webhook Verification](webhook-verification.md) — Validating incoming webhook payloads
- [Contacts](contacts.md) — Contact and group management
- [Consents](consents.md) — Recipient consent management
- [API Logs](api-logs.md) — API request and response logs
- [Error Handling](error-handling.md) — Exception types, rate limits, and retry
- [API Reference](api-reference.md) — Complete endpoint table and enums
