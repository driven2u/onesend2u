````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# API Logs

The `client.ApiLogs` sub-client provides read-only access to notification API log entries. An API log entry is created for every call to the send notification endpoint and records the full lifecycle of that request: receipt, queuing, processing, and final outcome.

API logs are useful for auditing, debugging failed requests, and monitoring throughput.

## Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List API log entries with optional filters and pagination |
| `GetAsync(id)` | Get a single API log entry by ID |

## What is logged

Each API log entry captures:

- The full request payload sent to the send endpoint
- Transaction ID, application, region, language, notification type/subtype
- Source of the request (`CPaaS` or `API`)
- Client IP address and HTTP method
- Processing status and status detail
- Timing information: when received, queued, and completed
- Total processing duration in milliseconds
- Number of messages created, succeeded, and failed
- Status change history

## API log response model

`ApiLogResponse` fields:

| Field | Type | Description |
|---|---|---|
| `Id` | `Guid` | Log entry ID |
| `CorrelationId` | `string?` | Internal correlation ID for tracing |
| `TransactionId` | `string?` | Transaction ID provided by the client |
| `ApplicationCode` | `string?` | Application code from the request |
| `RegionCode` | `string?` | Region code from the request |
| `NotificationTypeCode` | `string?` | Notification type code |
| `NotificationSubtypeCode` | `string?` | Notification subtype code |
| `LanguageCode` | `string?` | Language code |
| `Source` | `NotificationSource` | `CPaaS` (1) or `API` (2) |
| `Endpoint` | `string?` | API endpoint called |
| `HttpMethod` | `string?` | HTTP method used |
| `ClientIp` | `string?` | Client IP address |
| `RequestPayload` | `string?` | Raw request JSON |
| `Status` | `NotificationApiLogStatus` | Current processing status |
| `StatusDetail` | `string?` | Status detail message |
| `ExternalMessageId` | `string?` | External correlation ID from request |
| `ExternalSequenceNumber` | `string?` | External sequence number from request |
| `NotificationId` | `Guid?` | ID of the notification created |
| `DeploymentEnvironmentId` | `Guid?` | Environment ID |
| `MessagesCreated` | `int?` | Number of messages created |
| `MessagesSuccess` | `int?` | Number of messages delivered successfully |
| `MessagesError` | `int?` | Number of messages that failed |
| `ReceivedAt` | `DateTime` | When the request was received |
| `QueuedAt` | `DateTime?` | When the request was queued |
| `CompletedAt` | `DateTime?` | When processing completed |
| `DurationMs` | `int?` | Total processing time in milliseconds |
| `StatusHistory` | `List<ApiLogStatusEntryResponse>` | Status change history |

### NotificationApiLogStatus enum

| Value | Description |
|---|---|
| `Received` (1) | Request received |
| `ValidationError` (2) | Request failed validation |
| `Queued` (3) | Request queued for processing |
| `OK` (10) | Processed successfully |
| `Warning` (11) | Processed with warnings |
| `Error` (12) | Processing failed |

## Listing API logs

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.ApiLogs.Models;

var list = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
{
    SkipCount      = 0,
    MaxResultCount = 50
});

Console.WriteLine($"Total: {list.TotalCount}");

foreach (var log in list.Items)
{
    Console.WriteLine(
        $"[{log.ReceivedAt:O}] {log.TransactionId} — {log.Status} — {log.DurationMs}ms");
}
```
{{end}}

## Getting an API log by ID

{{if SDK == "csharp"}}
```csharp
var log = await client.ApiLogs.GetAsync(logId);

Console.WriteLine($"Transaction ID:  {log.TransactionId}");
Console.WriteLine($"Status:          {log.Status}");
Console.WriteLine($"Received:        {log.ReceivedAt:O}");
Console.WriteLine($"Completed:       {log.CompletedAt:O}");
Console.WriteLine($"Duration:        {log.DurationMs}ms");
Console.WriteLine($"Messages OK:     {log.MessagesSuccess}");
Console.WriteLine($"Messages Error:  {log.MessagesError}");

foreach (var entry in log.StatusHistory)
    Console.WriteLine($"  [{entry.Timestamp:O}] {entry.Status} — {entry.StatusDetail}  (source: {entry.Source})");
```
{{end}}

## Filtering by date range

{{if SDK == "csharp"}}
```csharp
var list = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
{
    SkipCount       = 0,
    MaxResultCount  = 100,
    Status          = NotificationApiLogStatus.Error,
    ReceivedAtMin   = DateTime.UtcNow.AddDays(-1),
    ReceivedAtMax   = DateTime.UtcNow,
    Sorting         = "ReceivedAt desc"
});
```
{{end}}

## Pagination

{{if SDK == "csharp"}}
```csharp
const int pageSize = 50;
int skip = 0;
long total;

do
{
    var page = await client.ApiLogs.GetListAsync(new GetApiLogsRequest
    {
        SkipCount      = skip,
        MaxResultCount = pageSize
    });

    total = page.TotalCount;

    foreach (var log in page.Items)
        Console.WriteLine($"{log.TransactionId} — {log.Status}");

    skip += pageSize;
}
while (skip < total);
```
{{end}}
