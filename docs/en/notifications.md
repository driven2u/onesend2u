````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Notifications

The `client.Notifications` sub-client covers sending notifications and querying their status.

## Available methods

| Method | Description |
|---|---|
| `SendAsync(request)` | Send a notification to one or more recipients |
| `GetListAsync(request)` | List notifications with optional filters and pagination |
| `GetAsync(id)` | Get a notification by ID |
| `GetWithDetailsAsync(id)` | Get a notification with navigation properties (messages, template, etc.) |

## Sending a notification

### Request model

`SendNotificationRequest` fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `TransactionId` | `string` | Yes | Unique identifier for this request (max 100 chars). Use for idempotency and tracing. |
| `Application` | `string` | Yes | Application code (max 10 chars, e.g., `"billing"`) |
| `Country` | `string` | Yes | ISO 3166-1 alpha-2 country code (max 2 chars, e.g., `"us"`) |
| `Language` | `string` | Yes | Language code (max 5 chars, e.g., `"en"` or `"pt-BR"`) |
| `NotificationType` | `string` | Yes | Notification type code (max 10 chars, e.g., `"trans"`) |
| `NotificationSubtype` | `string` | Yes | Notification subtype code (max 10 chars, e.g., `"invoice"`) |
| `Recipients` | `List<NotificationRecipient>` | Yes | At least one recipient required |
| `TargetChannels` | `List<TargetChannel>` | No | Limit delivery to specific channels. If empty, all channels configured for the template are used. |
| `TemplateVariables` | `List<Dictionary<string, string>>` | No | Variable substitutions per recipient |
| `Attachments` | `List<NotificationAttachment>` | No | Base64-encoded file attachments |
| `ExternalMessageId` | `string?` | No | External correlation ID |
| `ExternalSequenceNumber` | `string?` | No | External ordering sequence number |

### NotificationRecipient

| Field | Type | Description |
|---|---|---|
| `Channel` | `string?` | Channel type: `"sms"`, `"email"`, or `"whatsapp"` |
| `Recipient` | `string?` | Destination address: phone number, email address, or WhatsApp ID |

### NotificationAttachment

| Field | Type | Description |
|---|---|---|
| `FileName` | `string?` | File name including extension |
| `FileContent` | `string?` | File content encoded as Base64 |

### Send example

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Notifications.Models;

var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Country             = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    Recipients =
    [
        new NotificationRecipient { Channel = "sms",   Recipient = "+15550001234" },
        new NotificationRecipient { Channel = "email",  Recipient = "jane@example.com" }
    ],
    TemplateVariables =
    [
        new Dictionary<string, string>
        {
            ["customer_name"]  = "Jane Doe",
            ["invoice_number"] = "INV-001",
            ["amount"]         = "$120.00"
        }
    ]
});

Console.WriteLine($"Status: {response.Status}");
Console.WriteLine($"Created: {response.CreatedAt:O}");

if (response.Warnings?.Count > 0)
    foreach (var w in response.Warnings)
        Console.WriteLine($"Warning: {w}");
```
{{end}}

### Response model

`SendNotificationResponse` fields:

| Field | Type | Description |
|---|---|---|
| `TransactionId` | `string?` | The transaction ID echoed back |
| `Status` | `string?` | Notification status (e.g., `"Accepted"`) |
| `StatusDetail` | `string?` | Additional status detail |
| `CreatedAt` | `DateTime` | When the notification was created on the server |
| `Warnings` | `List<string>?` | Warnings about partially configured channels |

## Querying notifications

### List notifications

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Notifications.Models;

var list = await client.Notifications.GetListAsync(new GetNotificationsRequest
{
    SkipCount  = 0,
    MaxResults = 20
});

Console.WriteLine($"Total: {list.TotalCount}");
foreach (var notification in list.Items)
    Console.WriteLine($"{notification.Id} — {notification.Status}");
```
{{end}}

`GetListAsync` returns `PagedResult<NotificationResponse>` with:

| Field | Type | Description |
|---|---|---|
| `Items` | `List<T>` | Current page of results |
| `TotalCount` | `long` | Total matching records |

### Get a single notification

{{if SDK == "csharp"}}
```csharp
var notification = await client.Notifications.GetAsync(notificationId);
Console.WriteLine($"Status: {notification.Status}");
```
{{end}}

### Get notification with navigation properties

Returns the notification along with its related messages, template configuration, and other associated data.

{{if SDK == "csharp"}}
```csharp
var details = await client.Notifications.GetWithDetailsAsync(notificationId);

foreach (var message in details.Messages ?? [])
    Console.WriteLine($"Message {message.Id}: {message.MessageProcessState}");
```
{{end}}
