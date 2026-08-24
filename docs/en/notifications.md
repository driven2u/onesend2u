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

## Channels

Channels are represented by the typed `Channel` enum in `OneSend2U.Sdk.Models.Enums`:

| Enum member | Wire code |
|---|---|
| `Channel.Sms` | `sms` |
| `Channel.Email` | `email` |
| `Channel.WhatsApp` | `whatsapp` |

The enum serializes to/from the lowercase wire codes shown above (case-insensitive on read), so the JSON payloads sent and received by the API are unchanged. Use the enum members anywhere a `Channel?` property appears:

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Models.Enums;
using OneSend2U.Sdk.Notifications.Models;

new NotificationRecipient { Channel = Channel.Sms, Recipient = "+34915794174" };
new TargetChannel        { Channel = Channel.Email };
new ContactGroupTarget   { Code = "VIP", Channel = Channel.WhatsApp };
```
{{end}}

For dictionary keys (e.g., `SenderOverrides`) the key type is still `string` — `System.Text.Json` does not serialize enum keys. Use the `ChannelCodes` constants to keep the wire codes consistent:

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Models.Enums;

SenderOverrides = new Dictionary<string, SenderOverride>
{
    [ChannelCodes.Sms]   = new() { Address = "+34915794174" },
    [ChannelCodes.Email] = new() { Address = "security@verified.com" }
};
```
{{end}}

`ChannelCodes` also exposes helpers — `ChannelCodes.ToWireCode(Channel)` for query strings and `ChannelCodes.FromWireCode(string?)` for case-insensitive parsing of incoming codes.

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
| `Objects` | `Dictionary<string, object>?` | No | Structured data for email templates — nested objects and arrays. See [Structured data](#structured-data). |
| `Attachments` | `List<NotificationAttachment>` | No | Base64-encoded file attachments |
| `ExternalMessageId` | `string?` | No | External correlation ID |
| `ExternalSequenceNumber` | `string?` | No | External ordering sequence number |
| `SenderOverrides` | `Dictionary<string, SenderOverride>?` | No | Per-channel sender overrides. Keys: `ChannelCodes.Sms`, `ChannelCodes.Email`, `ChannelCodes.WhatsApp` (string keys, case-insensitive). See [Sender override](#sender-override). |

### NotificationRecipient

| Field | Type | Description |
|---|---|---|
| `Channel` | `Channel?` | Channel type — see [Channels](#channels) |
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
| `Channel` | `Channel?` | No | Channel filter — see [Channels](#channels). If omitted, all `TargetChannels` are used. |

### TargetChannel

| Field | Type | Description |
|---|---|---|
| `Channel` | `Channel?` | Channel type — see [Channels](#channels) |

### Send example

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Models.Enums;
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
        new NotificationRecipient { Channel = Channel.Sms,   Recipient = "+15550001234" },
        new NotificationRecipient { Channel = Channel.Email, Recipient = "jane@example.com" }
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
    TargetChannels      = [new TargetChannel { Channel = Channel.Email }],
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
        new TargetChannel { Channel = Channel.Email },
        new TargetChannel { Channel = Channel.Sms }
    ],
    Recipients =
    [
        new NotificationRecipient { Channel = Channel.Email, Recipient = "cfo@example.com" }
    ],
    ContactGroupCodes =
    [
        new ContactGroupTarget { Code = "FINANCE", Channel = Channel.Email },  // email only
        new ContactGroupTarget { Code = "MGMT" }                               // all channels
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

The default sender per channel is configured at the application level (Communication Channels tab). For one-off requests you can override it with the **`SenderOverrides`** map — keyed by channel code (`ChannelCodes.Sms`, `ChannelCodes.Email`, `ChannelCodes.WhatsApp`; the wire format is the lowercase code, case-insensitive on read). Each entry carries an optional `Address` and/or `Name`. Per-channel rules:

| Channel | `Address` | `Name` |
|---|---|---|
| Email | Allowed. The domain must be Verified in SenderDomain for the connection's external account; otherwise rejected with `Cpaas:ApplicationChannel:00003`. The local-part is free. | Allowed (free text). |
| SMS | Allowed. Must belong to the connection's available sender list (`Connection.Senders.Where(IsAvailableForApp)`); otherwise rejected with `Cpaas:SenderOverride:00011`. | Allowed (best-effort). Applied only when the destination country supports alphanumeric senders; silently discarded otherwise — the message is still delivered using the phone-number sender. |
| WhatsApp | Allowed. Same list-membership check as SMS; otherwise rejected with `Cpaas:SenderOverride:00011`. | Rejected with `Cpaas:SenderOverride:00001`. WhatsApp Cloud API ties the displayed name to the verified phone number, so a runtime override is impossible. |

An entry with both `Address` and `Name` null/whitespace is rejected with `Cpaas:SenderOverride:00012` (`SenderOverrideEmpty`).

The override applies to the whole notification (all recipients in the request). Resolution priority (most specific wins): API request → Application channel default → Connection default sender.

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Models.Enums;

// Email — override From address and name for this request only
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "reset_pwd",
    TargetChannels      = [new TargetChannel { Channel = Channel.Email }],
    Recipients          = [new NotificationRecipient { Channel = Channel.Email, Recipient = "jane@example.com" }],
    SenderOverrides     = new()
    {
        [ChannelCodes.Email] = new SenderOverride
        {
            Address = "security@your-verified-domain.com",
            Name    = "Acme Security"
        }
    },
    TemplateVariables   = [new Dictionary<string, string> { ["reset_link"] = "https://..." }]
});
```

```csharp
// SMS — pick a specific sender from the connection's available list and apply a display name
var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "otp",
    TargetChannels      = [new TargetChannel { Channel = Channel.Sms }],
    Recipients          = [new NotificationRecipient { Channel = Channel.Sms, Recipient = "+15550001234" }],
    SenderOverrides     = new()
    {
        [ChannelCodes.Sms] = new SenderOverride
        {
            Address = "+34915794174",   // must exist in Connection.Senders[].Address
            Name    = "Acme"            // best-effort: dropped on countries that don't support alpha
        }
    },
    TemplateVariables   = [new Dictionary<string, string> { ["code"] = "482910" }]
});
```
{{end}}

| Error code | When |
|---|---|
| `Cpaas:SenderOverride:00001` | Sent a `whatsapp` entry — WhatsApp does not allow per-request overrides. |
| `Cpaas:SenderOverride:00011` | Sent an SMS `Address` not present in the connection's available sender list. |
| `Cpaas:SenderOverride:00012` | Sent an entry with both `Address` and `Name` empty. |
| `Cpaas:SenderOverride:00013` | Used a channel key other than `sms` / `email` / `whatsapp`. |
| `Cpaas:ApplicationChannel:00003` | Sent an Email `Address` whose domain is not Verified for the resolved Connection. |
| `Cpaas:Integration:00116` | Sent a malformed Email address. |

#### Migration from SDK 1.x

`SDK 1.x` exposed flat `SenderAddress` / `SenderName` fields on `SendNotificationRequest`. SDK `2.0.0` replaces them with `SenderOverrides`:

```csharp
using OneSend2U.Sdk.Models.Enums;

// SDK 1.x
req.SenderAddress = "security@verified.com";
req.SenderName    = "Acme Security";

// SDK 2.0+
req.SenderOverrides = new()
{
    [ChannelCodes.Email] = new SenderOverride { Address = "security@verified.com", Name = "Acme Security" }
};
```

> **SDK 2.1 note:** the `Channel` properties (`NotificationRecipient.Channel`, `TargetChannel.Channel`, `ContactGroupTarget.Channel`) are now the typed `Channel?` enum instead of `string?`. Wire format on the API is unchanged. `SenderOverrides` keys are still `string` (System.Text.Json limitation); use `ChannelCodes` constants instead of string literals.

### Sending with attachments (Email)

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Models.Enums;

var response = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId       = Guid.NewGuid().ToString(),
    Application         = "billing",
    Region              = "us",
    Language            = "en",
    NotificationType    = "trans",
    NotificationSubtype = "invoice",
    TargetChannels      = [new TargetChannel { Channel = Channel.Email }],
    Recipients          = [new NotificationRecipient { Channel = Channel.Email, Recipient = "jane@example.com" }],
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

## Structured data

`TemplateVariables` carries flat values. When an email needs data with shape — an order with its lines, a
user with an address — send it in `Objects`.

```csharp
var request = new SendNotificationRequest
{
    // ... application, region, language, type, subtype, recipients ...
    TemplateVariables = [new Dictionary<string, string> { ["customerName"] = "Alice" }],
    Objects = new Dictionary<string, object>
    {
        ["user"]  = new { name = "Alice", isPremium = true, address = new { city = "Madrid" } },
        ["order"] = new
        {
            total = 150.00m,
            lines = new[]
            {
                new { productName = "T-Shirt", quantity = 2, unitPrice = 25.00m },
                new { productName = "Mug",     quantity = 1, unitPrice = 10.00m }
            }
        }
    }
};
```

The template reads it with dot notation, loops and conditions:

```
Hello {{ customerName }},

{{ for line in order.lines }}
  {{ line.quantity }} x {{ line.productName }} — {{ line.unitPrice | math.format "0.00" }}
{{ end }}

Total: {{ order.total | math.format "0.00" }}

{{ if user.isPremium }}Thank you for being a premium member.{{ end }}
```

Worth knowing:

- **Email only.** SMS and WhatsApp are rendered by the provider from positional parameters, so they ignore
  this section.
- **A root key of `Objects` satisfies a template variable of the same name**, so a template that expects
  `order` is served by `Objects["order"]` without also listing it in `TemplateVariables`.
- **On a name collision the flat variable wins**, which keeps templates written before this feature
  behaving exactly as they did.
- **Values are HTML-escaped**, so content coming from your systems cannot inject markup into the email.
- **A missing field renders as empty** rather than failing the send: `{{ user.address.postcode }}` on a
  payload without a postcode yields nothing. Use `{{ user.nickname ?? "there" }}` to supply a default.
- The payload is capped at 64 KB.

## Querying notifications

### List notifications

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Notifications.Models;

var list = await client.Notifications.GetListAsync(new GetNotificationsRequest
{
    SkipCount      = 0,
    MaxResultCount = 20
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

Returns the notification along with its related lookup data (application, type, subtype, country, environment, and channel types).

`GetWithDetailsAsync` returns `NotificationWithDetailsResponse` with:

| Field | Type | Description |
|---|---|---|
| `Notification` | `NotificationResponse` | The base notification data |
| `Application` | `LookupItem` | Application lookup |
| `NotificationType` | `LookupItem` | Notification type lookup |
| `NotificationSubtype` | `LookupItem` | Notification subtype lookup |
| `Country` | `LookupItem` | Country lookup. The send request body uses `Region` (string code), but the entity is stored as `Country`/`CountryId`. |
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
| `CountryId` | `Guid` | Country ID (entity-level field; the send request uses `Region` codes) |
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
Console.WriteLine($"Country: {details.Country?.Code}");
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
| `Source` | `NotificationSource?` | Origin: `CPaaS` (1) or `API` (2) |
| `Language` | `string?` | Language code |
| `Status` | `NotificationStatus?` | `Sending`, `Success`, `Error` |
| `ApplicationId` | `Guid?` | Filter by application |
| `NotificationTypeId` | `Guid?` | Filter by notification type |
| `NotificationSubtypeId` | `Guid?` | Filter by notification subtype |
| `CountryId` | `Guid?` | Filter by country |
| `ChannelTypeId` | `Guid?` | Filter by channel |
| `DeploymentEnvironmentId` | `Guid?` | Filter by deployment environment |
| `CreationTimeMin` | `DateTime?` | Minimum creation date |
| `CreationTimeMax` | `DateTime?` | Maximum creation date |
| `Sorting` | `string?` | Sort expression (e.g., `"CreationTime desc"`) |
| `SkipCount` | `int` | Items to skip (pagination) |
| `MaxResultCount` | `int` | Maximum items to return (default: 10) |
