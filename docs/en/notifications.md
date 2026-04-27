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
| `GetWithDetailsAsync(id)` | Get a notification with navigation properties (application, type, subtype, etc.) |

## Sending a notification

### Request model

`SendNotificationRequest` fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `TransactionId` | `string` | Yes | Unique identifier for this request (max 100 chars). Use for idempotency and tracing. |
| `Application` | `string` | Yes | Application code (max 10 chars, e.g., `"billing"`) |
| `Region` | `string` | Yes | Region code (alphanumeric, max 10 characters, e.g., `"us"`) |
| `Language` | `string` | Yes | Language code (max 5 chars, e.g., `"en"` or `"pt-BR"`) |
| `NotificationType` | `string` | Yes | Notification type code (max 10 chars, e.g., `"trans"`) |
| `NotificationSubtype` | `string` | Yes | Notification subtype code (max 10 chars, e.g., `"invoice"`) |
| `Recipients` | `List<NotificationRecipient>` | Conditional | Recipients list. At least one of `Recipients` or `ContactGroupCodes` is required. |
| `ContactGroupCodes` | `List<ContactGroupTarget>?` | Conditional | Contact group codes to resolve into recipients. At least one of `Recipients` or `ContactGroupCodes` is required. |
| `TargetChannels` | `List<TargetChannel>` | No | Limit delivery to specific channels. If empty, all channels configured for the template are used. |
| `TemplateVariables` | `List<Dictionary<string, string>>` | No | Variable substitutions per recipient |
| `Attachments` | `List<NotificationAttachment>` | No | Base64-encoded file attachments |
| `ExternalMessageId` | `string?` | No | External correlation ID |
| `ExternalSequenceNumber` | `string?` | No | External ordering sequence number |
| `SenderAddress` | `string?` | No | Sender address override (Email only). Domain must be Verified for the resolved Connection. See [Sender override](#sender-override). |
| `SenderName` | `string?` | No | Sender display name override (Email + SMS). See [Sender override](#sender-override). |

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

### ContactGroupTarget

| Field | Type | Required | Description |
|---|---|---|---|
| `Code` | `string` | Yes | Contact group code to resolve into recipients (max 10 chars) |
| `Channel` | `string?` | No | Channel filter (`"SMS"`, `"EMAIL"`, `"WHATSAPP"`). If omitted, all `TargetChannels` are used. |

### TargetChannel

| Field | Type | Description |
|---|---|---|
| `Channel` | `string?` | Channel type: `"sms"`, `"email"`, or `"whatsapp"` |

### Send example

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Notifications.Models;

var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
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

### Sending to Contact Groups

You can send to all members of one or more contact groups by specifying their codes. Group members are resolved server-side — no need to query the group and build the recipients list manually.

{{if SDK == "csharp"}}
```csharp
// Send to a contact group (all TargetChannels)
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    TargetChannels      = [new TargetChannel { Channel = "email" }],
    ContactGroupCodes   =
    [
        new ContactGroupTarget { Code = "VIP" }  // all TargetChannels
    ],
    TemplateVariables =
    [
        new Dictionary<string, string>
        {
            ["customer_name"] = "Valued Customer",
            ["invoice_number"] = "INV-001"
        }
    ]
});
```
{{end}}

You can also mix explicit recipients with contact groups. Duplicates are automatically removed by `(channel, destination)`:

{{if SDK == "csharp"}}
```csharp
// Mix explicit recipients + contact groups with channel filter
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    TargetChannels      =
    [
        new TargetChannel { Channel = "email" },
        new TargetChannel { Channel = "sms" }
    ],
    Recipients =
    [
        new NotificationRecipient { Channel = "email", Recipient = "cfo@example.com" }
    ],
    ContactGroupCodes =
    [
        new ContactGroupTarget { Code = "FINANCE", Channel = "EMAIL" },  // email only
        new ContactGroupTarget { Code = "MGMT" }                         // all channels
    ],
    TemplateVariables =
    [
        new Dictionary<string, string>
        {
            ["customer_name"] = "Team",
            ["invoice_number"] = "INV-002"
        }
    ]
});
```
{{end}}

> **Note:** If a contact group code does not exist or has no members, a warning is included in the response but the notification is still processed for any valid recipients (partial success). Contacts without the required channel data (e.g., no phone number for SMS) are silently skipped.

### Response model

`SendNotificationResponse` fields:

| Field | Type | Description |
|---|---|---|
| `TransactionId` | `string?` | The transaction ID echoed back |
| `Status` | `string?` | Notification status (e.g., `"Accepted"`) |
| `StatusDetail` | `string?` | Additional status detail |
| `CreatedAt` | `DateTime` | When the notification was created on the server |
| `Warnings` | `List<string>?` | Warnings about partially configured channels |

### Sender override

The default sender per channel is configured at the application level (Communication Channels tab). For one-off requests you can override it with the `SenderAddress` and `SenderName` fields. Per-channel policies apply:

| Channel | `SenderAddress` | `SenderName` |
|---|---|---|
| Email | Allowed. The domain must be Verified in SenderDomain for the connection's external account; otherwise rejected with `Cpaas:ApplicationChannel:00003`. | Allowed. |
| SMS | Rejected with `Cpaas:SenderOverride:00001`. | Allowed (best-effort). Applied only when the destination country supports alphanumeric senders; silently discarded otherwise — the message is still delivered using the phone-number sender. |
| WhatsApp | Rejected with `Cpaas:SenderOverride:00001`. | Rejected with `Cpaas:SenderOverride:00002`. |

The override applies to the whole notification (all recipients in the request). Resolution priority (most specific wins): API request → Template configuration → Application channel default → Connection default sender.

{{if SDK == "csharp"}}
```csharp
// Email — override From address and name for this request only
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "reset_pwd",
    TargetChannels      = [new TargetChannel { Channel = "email" }],
    Recipients          = [new NotificationRecipient { Channel = "email", Recipient = "jane@example.com" }],
    SenderAddress       = "security@your-verified-domain.com",
    SenderName          = "Acme Security",
    TemplateVariables   = [new Dictionary<string, string> { ["reset_link"] = "https://..." }]
});
```

```csharp
// SMS — only the display name can be overridden; SenderAddress is rejected for SMS
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "otp",
    TargetChannels      = [new TargetChannel { Channel = "sms" }],
    Recipients          = [new NotificationRecipient { Channel = "sms", Recipient = "+15550001234" }],
    SenderName          = "Acme",
    TemplateVariables   = [new Dictionary<string, string> { ["code"] = "482910" }]
});
```
{{end}}

| Error code | When |
|---|---|
| `Cpaas:SenderOverride:00001` | Sent `SenderAddress` to a channel that does not allow it (SMS, WhatsApp). |
| `Cpaas:SenderOverride:00002` | Sent `SenderName` to a channel that does not allow it (WhatsApp). |
| `Cpaas:ApplicationChannel:00003` | Sent an Email `SenderAddress` whose domain is not Verified for the resolved Connection. |
| `Cpaas:Integration:00116` | Sent a malformed Email address. |

### Sending with attachments (Email)

{{if SDK == "csharp"}}
```csharp
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    TargetChannels      = [new TargetChannel { Channel = "email" }],
    Recipients          = [new NotificationRecipient { Channel = "email", Recipient = "jane@example.com" }],
    TemplateVariables   = [new Dictionary<string, string> { ["invoice_number"] = "INV-001" }],
    Attachments =
    [
        new NotificationAttachment
        {
            FileName    = "invoice-001.pdf",
            FileContent = Convert.ToBase64String(pdfBytes)
        }
    ]
});
```
{{end}}

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

`GetListAsync` returns `PagedResult<NotificationWithDetailsResponse>` with:

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

Returns the notification along with its related lookup data (application, type, subtype, region, environment, and channel types).

`GetWithDetailsAsync` returns `NotificationWithDetailsResponse` with:

| Field | Type | Description |
|---|---|---|
| `Notification` | `NotificationResponse` | The base notification data |
| `Application` | `LookupItem` | Application lookup |
| `NotificationType` | `LookupItem` | Notification type lookup |
| `NotificationSubtype` | `LookupItem` | Notification subtype lookup |
| `Region` | `LookupItem` | Region lookup |
| `DeploymentEnvironment` | `LookupItem` | Deployment environment lookup |
| `ChannelTypes` | `List<LookupItem>` | Channel types used |

`NotificationResponse` fields:

| Field | Type | Description |
|---|---|---|
| `Id` | `Guid` | Notification ID |
| `Source` | `NotificationSource?` | Origin: `CPaaS` or `API` |
| `Language` | `string?` | Language code |
| `Status` | `NotificationStatus?` | `Sending`, `Success`, or `Error` |
| `NumberOfSuccessMessages` | `int` | Count of successfully delivered messages |
| `NumberOfTotalMessages` | `int` | Total message count |
| `ApplicationId` | `Guid` | Application ID |
| `NotificationTypeId` | `Guid` | Notification type ID |
| `NotificationSubtypeId` | `Guid` | Notification subtype ID |
| `RegionId` | `Guid` | Region ID |
| `CreationTime` | `DateTime` | When the notification was created |
| `ConcurrencyStamp` | `string?` | Optimistic concurrency stamp |

`LookupItem` fields:

| Field | Type | Description |
|---|---|---|
| `Id` | `Guid` | Entity ID |
| `Name` | `string?` | Display name |
| `Code` | `string?` | Short code |
| `DisplayName` | `string?` | Full display name |

{{if SDK == "csharp"}}
```csharp
var details = await client.Notifications.GetWithDetailsAsync(notificationId);

// Access the base notification data through details.Notification
Console.WriteLine($"Status: {details.Notification.Status}");
Console.WriteLine($"Messages sent: {details.Notification.NumberOfSuccessMessages}/{details.Notification.NumberOfTotalMessages}");

// Access related lookup data
Console.WriteLine($"Application: {details.Application?.Name}");
Console.WriteLine($"Region: {details.Region?.Code}");
Console.WriteLine($"Environment: {details.DeploymentEnvironment?.Name}");

// List the channels used
foreach (var channel in details.ChannelTypes ?? [])
    Console.WriteLine($"Channel: {channel.DisplayName}");
```
{{end}}

## Filter parameters

`GetNotificationsRequest` supports the following filters:

| Parameter | Type | Description |
|---|---|---|
| `FilterText` | `string?` | Free text search across multiple fields |
| `Source` | `NotificationSource?` | Origin: `Api` or `Platform` |
| `Language` | `string?` | Language code |
| `Status` | `NotificationStatus?` | `Sending`, `Success`, `Error` |
| `ApplicationId` | `Guid?` | Filter by application |
| `NotificationTypeId` | `Guid?` | Filter by notification type |
| `NotificationSubtypeId` | `Guid?` | Filter by notification subtype |
| `RegionId` | `Guid?` | Filter by region |
| `ChannelTypeId` | `Guid?` | Filter by channel |
| `DeploymentEnvironmentId` | `Guid?` | Filter by deployment environment |
| `CreationTimeMin` | `DateTime?` | Minimum creation date |
| `CreationTimeMax` | `DateTime?` | Maximum creation date |
| `Sorting` | `string?` | Sort expression (e.g., `"CreationTime desc"`) |
| `SkipCount` | `int` | Items to skip (pagination) |
| `MaxResultCount` | `int` | Maximum items to return (default: 10) |
