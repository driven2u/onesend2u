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
| `RegionId` | `Guid` | Region ID |
| `ChannelTypeId` | `Guid` | Channel type ID |
| `NotificationTypeId` | `Guid` | Notification type ID |
| `NotificationSubtypeId` | `Guid` | Notification subtype ID |
| `ProviderId` | `Guid` | Provider ID used for delivery |
| `CreationTime` | `DateTime` | When the message was created |
| `LastModificationTime` | `DateTime?` | Last state change time |
| `ConcurrencyStamp` | `string?` | Optimistic concurrency stamp |

## Listing messages

`GetListAsync` returns `PagedResult<MessageWithDetailsResponse>`. Each item wraps a `Message` (`MessageResponse`) plus lookup data (region, application, channel type, etc.).

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Messages.Models;

var list = await client.Messages.GetListAsync(new GetMessagesRequest
{
    SkipCount  = 0,
    MaxResults = 50
});

Console.WriteLine($"Total: {list.TotalCount}");

foreach (var item in list.Items)
{
    // Core message fields are inside item.Message
    Console.WriteLine(
        $"[{item.Message.Id}] {item.Message.Destination} — {item.Message.MessageProcessState}");
}
```
{{end}}

### Filter parameters

`GetMessagesRequest` supports the following server-side filters:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `FilterText` | `string?` | `null` | Free text search across multiple fields |
| `Destination` | `string?` | `null` | Filter by destination address (phone, email, WhatsApp ID) |
| `MessageProcessState` | `MessageProcessState?` | `null` | Filter by delivery state |
| `Language` | `string?` | `null` | Filter by language code |
| `NotificationSource` | `NotificationSource?` | `null` | Filter by origin: `CPaaS` or `API` |
| `RegionId` | `Guid?` | `null` | Filter by region |
| `ApplicationId` | `Guid?` | `null` | Filter by application |
| `ChannelTypeId` | `Guid?` | `null` | Filter by channel type |
| `NotificationTypeId` | `Guid?` | `null` | Filter by notification type |
| `NotificationSubtypeId` | `Guid?` | `null` | Filter by notification subtype |
| `ProviderId` | `Guid?` | `null` | Filter by provider |
| `DeploymentEnvironmentId` | `Guid?` | `null` | Filter by deployment environment |
| `NotificationId` | `Guid?` | `null` | Filter by parent notification ID |
| `CreationTimeMin` | `DateTime?` | `null` | Minimum creation date |
| `CreationTimeMax` | `DateTime?` | `null` | Maximum creation date |
| `Sorting` | `string?` | `null` | Sort expression (e.g., `"CreationTime desc"`) |
| `SkipCount` | `int` | `0` | Items to skip (pagination) |
| `MaxResultCount` | `int` | `10` | Maximum items to return |

{{if SDK == "csharp"}}
```csharp
// Filter by notification ID and delivery state
var list = await client.Messages.GetListAsync(new GetMessagesRequest
{
    NotificationId     = notificationId,
    MessageProcessState = MessageProcessState.Error,
    SkipCount          = 0,
    MaxResultCount     = 20
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

Returns a `MessageWithDetailsResponse` that wraps the base `MessageResponse` plus lookup data (region, application, channel type, provider, notification type, etc.).

{{if SDK == "csharp"}}
```csharp
var details = await client.Messages.GetWithDetailsAsync(messageId);

// Core message fields are inside details.Message
Console.WriteLine($"Message state: {details.Message.MessageProcessState}");
Console.WriteLine($"Destination: {details.Message.Destination}");

// Lookup data is available as named properties
Console.WriteLine($"Application: {details.Application?.Name}");
Console.WriteLine($"Region: {details.Region?.Code}");
Console.WriteLine($"Channel: {details.ChannelType?.DisplayName}");
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

Console.WriteLine($"Total messages: {page1.TotalCount}"); // TotalCount is the same across pages
```
{{end}}
