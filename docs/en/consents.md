````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Consents

The `client.Consents` sub-client manages recipient consent records. Consent tracking allows you to comply with regulations such as GDPR, LGPD, and TCPA by recording whether a recipient has opted in or out of receiving communications on a specific channel or for a specific application and notification type.

When a message is processed, the platform checks the consent status for the recipient. If the recipient has not consented (or has explicitly opted out), the message is placed in the `NotConsented` state and not sent.

## Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List consents with optional filters and pagination |
| `GetAsync(id)` | Get a consent record by ID |
| `GetWithDetailsAsync(id)` | Get a consent record with navigation properties |
| `CreateAsync(request)` | Create a new consent record |
| `UpdateAsync(id, request)` | Update an existing consent record |
| `DeleteAsync(id)` | Delete a consent record |

## Consent model

`ConsentResponse` fields:

| Field | Type | Description |
|---|---|---|
| `Id` | `Guid` | Consent ID |
| `Recipient` | `string?` | Recipient address (phone, email, etc.) |
| `IsConsented` | `bool` | `true` if the recipient has consented; `false` if opted out |
| `RegionId` | `Guid?` | Region scope (optional) |
| `ApplicationId` | `Guid?` | Application scope (optional) |
| `NotificationTypeId` | `Guid?` | Notification type scope (optional) |
| `NotificationSubtypeId` | `Guid?` | Notification subtype scope (optional) |
| `ChannelTypeId` | `Guid?` | Channel scope (optional) |
| `DeploymentEnvironmentId` | `Guid?` | Environment scope (optional) |
| `ConcurrencyStamp` | `string?` | Required for update operations |
| `CreationTime` | `DateTime` | When the consent was recorded |
| `LastModificationTime` | `DateTime?` | Last update time |

Scoping fields are optional. A consent record without a scope applies broadly. A record with `ChannelTypeId` scoped to SMS applies only to SMS messages for that recipient.

## Creating a consent record

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Consents.Models;

var consent = await client.Consents.CreateAsync(new CreateConsentRequest
{
    Recipient   = "+15550001234",
    IsConsented = true,
    ChannelTypeId             = smsChannelTypeId,
    ApplicationId             = billingApplicationId,
    DeploymentEnvironmentId   = productionEnvironmentId
});

Console.WriteLine($"Consent created: {consent.Id}");
Console.WriteLine($"Consented: {consent.IsConsented}");
```
{{end}}

To record an opt-out, set `IsConsented = false`:

{{if SDK == "csharp"}}
```csharp
var optOut = await client.Consents.CreateAsync(new CreateConsentRequest
{
    Recipient   = "+15550001234",
    IsConsented = false,
    ChannelTypeId = smsChannelTypeId
});
```
{{end}}

## Listing consents

{{if SDK == "csharp"}}
```csharp
var list = await client.Consents.GetListAsync(new GetConsentsRequest
{
    SkipCount  = 0,
    MaxResults = 50
});

Console.WriteLine($"Total: {list.TotalCount}");
foreach (var consent in list.Items)
    Console.WriteLine($"{consent.Recipient} — Consented: {consent.IsConsented}");
```
{{end}}

## Getting a consent by ID

{{if SDK == "csharp"}}
```csharp
var consent = await client.Consents.GetAsync(consentId);
Console.WriteLine($"Recipient: {consent.Recipient}");
Console.WriteLine($"Consented: {consent.IsConsented}");
```
{{end}}

## Getting a consent with navigation properties

Returns the consent record along with its related region, application, channel type, and environment objects.

{{if SDK == "csharp"}}
```csharp
var details = await client.Consents.GetWithDetailsAsync(consentId);
```
{{end}}

## Updating a consent record

Use this to change the consent status (e.g., when a recipient opts out via a preference center).

{{if SDK == "csharp"}}
```csharp
var existing = await client.Consents.GetAsync(consentId);

await client.Consents.UpdateAsync(consentId, new UpdateConsentRequest
{
    Recipient             = existing.Recipient,
    IsConsented           = false,   // opt-out
    ChannelTypeId         = existing.ChannelTypeId,
    ApplicationId         = existing.ApplicationId,
    DeploymentEnvironmentId = existing.DeploymentEnvironmentId,
    ConcurrencyStamp      = existing.ConcurrencyStamp
});
```
{{end}}

## Deleting a consent record

{{if SDK == "csharp"}}
```csharp
await client.Consents.DeleteAsync(consentId);
Console.WriteLine("Consent record deleted.");
```
{{end}}

## Regulatory compliance notes

- **GDPR** (EU): Requires a lawful basis for processing personal data. Explicit consent is one basis. Record and retain consent timestamps.
- **LGPD** (Brazil): Similar to GDPR — explicit consent required for marketing communications.
- **TCPA** (US): Prior express written consent required for automated SMS and calls to mobile numbers.

The OneSend2U consent system records and enforces opt-in/opt-out at the platform level. Always consult your legal team for jurisdiction-specific requirements.
