````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Messages

The `client.Messages` sub-client provides read-only access to individual messages. A **message** represents a single delivery attempt for one recipient on one channel. A notification can produce multiple messages (one per recipient per channel).

## Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List messages with optional filters and pagination |
| `GetAsync(id)` | Get a message by ID |
| `GetWithDetailsAsync(id)` | Get a message with full navigation properties |

## Message lifecycle

A message transitions through the following states (`MessageProcessState` enum):

| State | Description |
|---|---|
| `Initial` | Created but not yet processed |
| `Pending` | Queued for sending |
| `Sending` | Currently being sent to the provider |
| `Success` | Delivered to the provider successfully |
| `Error` | Sending failed |
| `Discarded` | Discarded before sending (e.g., duplicate) |
| `NotConsented` | Recipient has not consented to receive messages |
| `Unknown` | State cannot be determined |

## Message response model

`MessageResponse` fields:

| Field | Type | Description |
|---|---|---|
| `Id` | `Guid` | Message ID |
| `Destination` | `string?` | Destination address (phone, email, WhatsApp ID) |
| `MessageProcessState` | `MessageProcessState?` | Current delivery state |
| `Language` | `string?` | Language code used |
| `NotificationSource` | `NotificationSource?` | Origin: `Api` or `Platform` |
| `TemplateVariables` | `string?` | Serialized variable values used |
| `NotificationId` | `Guid` | Parent notification ID |
| `ApplicationId` | `Guid` | Application ID |
| `CountryId` | `Guid` | Country ID |
| `ChannelTypeId` | `Guid` | Channel type ID |
| `NotificationTypeId` | `Guid` | Notification type ID |
| `NotificationSubtypeId` | `Guid` | Notification subtype ID |
| `ProviderId` | `Guid` | Provider ID used for delivery |
| `CreationTime` | `DateTime` | When the message was created |
| `LastModificationTime` | `DateTime?` | Last state change time |
| `ConcurrencyStamp` | `string?` | Optimistic concurrency stamp |

## Listing messages

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Messages.Models;

var list = await client.Messages.GetListAsync(new GetMessagesRequest
{
    SkipCount  = 0,
    MaxResults = 50
});

Console.WriteLine($"Total: {list.TotalCount}");

foreach (var message in list.Items)
{
    Console.WriteLine(
        $"[{message.Id}] {message.Destination} — {message.MessageProcessState}");
}
```
{{end}}

### Filtering

`GetMessagesRequest` supports server-side filtering. Refer to the platform admin panel for supported filter fields. Common filters include notification ID and message state.

{{if SDK == "csharp"}}
```csharp
var list = await client.Messages.GetListAsync(new GetMessagesRequest
{
    SkipCount  = 0,
    MaxResults = 20
    // Add filter properties as supported by GetMessagesRequest
});
```
{{end}}

## Getting a message by ID

{{if SDK == "csharp"}}
```csharp
var message = await client.Messages.GetAsync(messageId);
Console.WriteLine($"State: {message.MessageProcessState}");
Console.WriteLine($"Destination: {message.Destination}");
Console.WriteLine($"Created: {message.CreationTime:O}");
```
{{end}}

## Getting a message with navigation properties

Returns the message with its related notification, template, provider details, and status history.

{{if SDK == "csharp"}}
```csharp
var details = await client.Messages.GetWithDetailsAsync(messageId);
Console.WriteLine($"Message state: {details.MessageProcessState}");
```
{{end}}

## Pagination

All list methods return `PagedResult<T>`:

{{if SDK == "csharp"}}
```csharp
var page1 = await client.Messages.GetListAsync(new GetMessagesRequest
{
    SkipCount  = 0,
    MaxResults = 20
});

var page2 = await client.Messages.GetListAsync(new GetMessagesRequest
{
    SkipCount  = 20,
    MaxResults = 20
});

Console.WriteLine($"Total messages: {page1.TotalCount}");
```
{{end}}
