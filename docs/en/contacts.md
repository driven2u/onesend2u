````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Contacts

The SDK provides two sub-clients for managing contacts and contact groups:

- `client.Contacts` — individual contact records
- `client.ContactsGroups` — groups that organize contacts

## Contacts

### Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List contacts with optional filters and pagination |
| `GetAsync(id)` | Get a contact by ID |
| `CreateAsync(request)` | Create a new contact |
| `UpdateAsync(id, request)` | Update an existing contact |
| `DeleteAsync(id)` | Delete a contact |

### Contact model

`ContactResponse` fields:

| Field | Type | Description |
|---|---|---|
| `Id` | `Guid` | Contact ID |
| `Name` | `string?` | First name |
| `Surname` | `string?` | Last name |
| `Email` | `string?` | Email address |
| `Phone` | `string?` | Phone number (E.164 format recommended) |
| `DisplayName` | `string?` | Computed display name (`Name + Surname`) |
| `ConcurrencyStamp` | `string?` | Required for update operations |
| `CreationTime` | `DateTime` | When the contact was created |
| `LastModificationTime` | `DateTime?` | Last update time |

### Creating a contact

`Name` and `Email` are required.

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Contacts.Models;

var contact = await client.Contacts.CreateAsync(new CreateContactRequest
{
    Name    = "Jane",
    Surname = "Doe",
    Email   = "jane.doe@example.com",
    Phone   = "+15550001234"
});

Console.WriteLine($"Created contact: {contact.Id} — {contact.DisplayName}");
```
{{end}}

### Listing contacts

{{if SDK == "csharp"}}
```csharp
var list = await client.Contacts.GetListAsync(new GetContactsRequest
{
    SkipCount      = 0,
    MaxResultCount = 50
});

Console.WriteLine($"Total: {list.TotalCount}");
foreach (var contact in list.Items)
    Console.WriteLine($"{contact.Id} — {contact.DisplayName} — {contact.Email}");
```
{{end}}

### Getting a contact by ID

{{if SDK == "csharp"}}
```csharp
var contact = await client.Contacts.GetAsync(contactId);
Console.WriteLine($"Name: {contact.DisplayName}");
Console.WriteLine($"Phone: {contact.Phone}");
```
{{end}}

### Updating a contact

Pass the `ConcurrencyStamp` from the `GetAsync` response to enable optimistic concurrency:

{{if SDK == "csharp"}}
```csharp
var existing = await client.Contacts.GetAsync(contactId);

await client.Contacts.UpdateAsync(contactId, new UpdateContactRequest
{
    Name             = existing.Name,
    Surname          = existing.Surname,
    Email            = "new.email@example.com",
    Phone            = existing.Phone,
    ConcurrencyStamp = existing.ConcurrencyStamp
});
```
{{end}}

### Deleting a contact

{{if SDK == "csharp"}}
```csharp
await client.Contacts.DeleteAsync(contactId);
Console.WriteLine("Contact deleted.");
```
{{end}}

---

## Contacts Groups

### Available methods

| Method | Description |
|---|---|
| `GetListAsync(request)` | List contact groups with optional filters and pagination |
| `GetAsync(id)` | Get a contact group by ID |
| `GetWithDetailsAsync(id)` | Get a contact group with its member contacts |
| `CreateAsync(request)` | Create a new contact group |
| `UpdateAsync(id, request)` | Update an existing contact group |
| `DeleteAsync(id)` | Delete a contact group |

### Creating a contact group

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.ContactsGroups.Models;

var group = await client.ContactsGroups.CreateAsync(new CreateContactsGroupRequest
{
    Name        = "Newsletter Subscribers",
    Code        = "NEWSLTTR",   // unique per tenant, max 10 chars
    Description = "Opted-in customers for the monthly newsletter",
    ContactIds  = [contactId1, contactId2]
});

Console.WriteLine($"Group created: {group.Id} — {group.Name} ({group.Code})");
```
{{end}}

### Listing contact groups

{{if SDK == "csharp"}}
```csharp
var list = await client.ContactsGroups.GetListAsync(new GetContactsGroupsRequest
{
    SkipCount      = 0,
    MaxResultCount = 20
});

foreach (var group in list.Items)
    Console.WriteLine($"{group.Id} — {group.Name} ({group.Code})");
```
{{end}}

### Getting a group with members

{{if SDK == "csharp"}}
```csharp
var details = await client.ContactsGroups.GetWithDetailsAsync(groupId);

// details.ContactsGroup holds the group itself; details.Contacts is the member list
Console.WriteLine($"Group: {details.ContactsGroup?.Name}");
foreach (var contact in details.Contacts ?? [])
    Console.WriteLine($"  Member: {contact.DisplayName}");
```
{{end}}

### Updating a contact group

{{if SDK == "csharp"}}
```csharp
var existing = await client.ContactsGroups.GetAsync(groupId);

await client.ContactsGroups.UpdateAsync(groupId, new UpdateContactsGroupRequest
{
    Name             = existing.Name,
    Code             = existing.Code,
    Description      = "Updated description",
    ContactIds       = [contactId1, contactId2, contactId3],
    ConcurrencyStamp = existing.ConcurrencyStamp
});
```
{{end}}

### Deleting a contact group

{{if SDK == "csharp"}}
```csharp
await client.ContactsGroups.DeleteAsync(groupId);
Console.WriteLine("Group deleted.");
```
{{end}}
