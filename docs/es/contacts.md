````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Contactos y Grupos de Contactos

El SDK proporciona dos sub-clientes para gestionar contactos y sus agrupaciones:

- `client.Contacts` — CRUD de contactos individuales (5 endpoints)
- `client.ContactsGroups` — CRUD de grupos de contactos (6 endpoints)

## Contactos

### Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista contactos con filtros y paginación |
| `GetAsync(id)` | GET | Obtiene un contacto por ID |
| `CreateAsync(request)` | POST | Crea un nuevo contacto |
| `UpdateAsync(id, request)` | PUT | Actualiza un contacto existente |
| `DeleteAsync(id)` | DELETE | Elimina un contacto |

### Crear un contacto

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Contacts.Models;

var contacto = await client.Contacts.CreateAsync(new CreateContactRequest
{
    Name = "María",
    Surname = "García López",
    Email = "maria.garcia@empresa.com",
    Phone = "+5215512345678"
});

Console.WriteLine($"Contacto creado: {contacto.Id}");
Console.WriteLine($"Nombre completo: {contacto.DisplayName}");
```
{{end}}

### Modelo de creación de contacto

{{if SDK == "csharp"}}
```csharp
public class CreateContactRequest
{
    public string? Name { get; set; }     // Nombre. Requerido.
    public string? Surname { get; set; }  // Apellido.
    public string? Email { get; set; }    // Correo electrónico. Requerido.
    public string? Phone { get; set; }    // Número de teléfono.
}
```
{{end}}

### Modelo de respuesta de contacto

{{if SDK == "csharp"}}
```csharp
public class ContactResponse
{
    public Guid Id { get; set; }
    public string? Name { get; set; }
    public string? Surname { get; set; }
    public string? Email { get; set; }
    public string? Phone { get; set; }
    public string? DisplayName { get; set; }        // Nombre + Apellido (calculado)
    public string? ConcurrencyStamp { get; set; }   // Requerido para actualizaciones
    public DateTime CreationTime { get; set; }
    public DateTime? LastModificationTime { get; set; }
}
```
{{end}}

### Listar contactos

{{if SDK == "csharp"}}
```csharp
var resultado = await client.Contacts.GetListAsync(new GetContactsRequest
{
    FilterText = "garcia",
    SkipCount = 0,
    MaxResultCount = 20
});

Console.WriteLine($"Total: {resultado.TotalCount}");
foreach (var contacto in resultado.Items)
{
    Console.WriteLine($"{contacto.DisplayName} — {contacto.Email} — {contacto.Phone}");
}
```
{{end}}

### Obtener un contacto por ID

{{if SDK == "csharp"}}
```csharp
var contacto = await client.Contacts.GetAsync(contactoId);
Console.WriteLine($"Email: {contacto.Email}");
Console.WriteLine($"Teléfono: {contacto.Phone}");
```
{{end}}

### Actualizar un contacto

Incluye el `ConcurrencyStamp` del contacto actual para evitar conflictos de concurrencia:

{{if SDK == "csharp"}}
```csharp
var actual = await client.Contacts.GetAsync(contactoId);

await client.Contacts.UpdateAsync(contactoId, new UpdateContactRequest
{
    Name = actual.Name,
    Surname = actual.Surname,
    Email = "nuevo.email@empresa.com",
    Phone = actual.Phone,
    ConcurrencyStamp = actual.ConcurrencyStamp  // Requerido
});
```
{{end}}

### Eliminar un contacto

{{if SDK == "csharp"}}
```csharp
await client.Contacts.DeleteAsync(contactoId);
Console.WriteLine("Contacto eliminado.");
```
{{end}}

---

## Grupos de Contactos

### Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista grupos de contactos con filtros y paginación |
| `GetAsync(id)` | GET | Obtiene un grupo por ID |
| `GetWithDetailsAsync(id)` | GET | Obtiene un grupo con sus contactos incluidos |
| `CreateAsync(request)` | POST | Crea un nuevo grupo de contactos |
| `UpdateAsync(id, request)` | PUT | Actualiza un grupo existente |
| `DeleteAsync(id)` | DELETE | Elimina un grupo |

### Crear un grupo de contactos

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.ContactsGroups.Models;

var grupo = await client.ContactsGroups.CreateAsync(new CreateContactsGroupRequest
{
    Name = "Clientes Premium México",
    Code = "NEWSLTTR",   // único por tenant, máx. 10 caracteres
    ContactIds = [contactoId1, contactoId2, contactoId3]
});

Console.WriteLine($"Grupo creado: {grupo.Id} — {grupo.Name} ({grupo.Code})");
```
{{end}}

### Modelo de creación de grupo

{{if SDK == "csharp"}}
```csharp
public class CreateContactsGroupRequest
{
    public string? Name { get; set; }           // Nombre del grupo. Requerido.
    public List<Guid> ContactIds { get; set; }  // IDs de contactos a incluir.
}
```
{{end}}

### Listar grupos de contactos

{{if SDK == "csharp"}}
```csharp
var resultado = await client.ContactsGroups.GetListAsync(new GetContactsGroupsRequest
{
    FilterText = "premium",
    SkipCount = 0,
    MaxResultCount = 10
});

foreach (var grupo in resultado.Items)
{
    Console.WriteLine($"{grupo.Name} ({grupo.Code}) — {grupo.CreationTime}");
}
```
{{end}}

### Obtener un grupo con sus contactos

{{if SDK == "csharp"}}
```csharp
// GetWithDetailsAsync incluye la lista de contactos del grupo
var grupoConContactos = await client.ContactsGroups.GetWithDetailsAsync(grupoId);

Console.WriteLine($"Grupo: {grupoConContactos.Name}");
Console.WriteLine($"Contactos:");
foreach (var contacto in grupoConContactos.Contacts)
{
    Console.WriteLine($"  - {contacto.DisplayName} ({contacto.Email})");
}
```
{{end}}

### Actualizar un grupo

{{if SDK == "csharp"}}
```csharp
var actual = await client.ContactsGroups.GetAsync(grupoId);

await client.ContactsGroups.UpdateAsync(grupoId, new UpdateContactsGroupRequest
{
    Name = "Clientes VIP México",             // Nuevo nombre
    Code = actual.Code,
    ContactIds = [contactoId1, contactoId4],  // Nueva lista de contactos
    ConcurrencyStamp = actual.ConcurrencyStamp
});
```
{{end}}

### Eliminar un grupo

{{if SDK == "csharp"}}
```csharp
await client.ContactsGroups.DeleteAsync(grupoId);
Console.WriteLine("Grupo eliminado.");
```
{{end}}

## Caso de uso: envío a un grupo completo

Los grupos de contactos se pueden usar como base para envíos masivos. Una vez que tienes los IDs de los contactos de un grupo, puedes construir los destinatarios de la notificación:

{{if SDK == "csharp"}}
```csharp
// Obtener contactos del grupo
var grupo = await client.ContactsGroups.GetWithDetailsAsync(grupoId);

// Construir la lista de destinatarios
var destinatarios = grupo.Contacts
    .Where(c => !string.IsNullOrEmpty(c.Phone))
    .Select(c => new NotificationRecipient
    {
        Channel = "sms",
        Recipient = c.Phone
    })
    .ToList();

// Enviar a todos los contactos del grupo
var resultado = await client.Notifications.SendAsync(new SendNotificationRequest
{
    TransactionId = Guid.NewGuid().ToString(),
    Application = "myapp",
    Country = "mx",
    Language = "es",
    NotificationType = "mktg",
    NotificationSubtype = "promo",
    Recipients = destinatarios,
    TemplateVariables = destinatarios.Select(_ => new Dictionary<string, string>
    {
        { "descuento", "20%" }
    }).ToList()
});

Console.WriteLine($"Notificación enviada: {resultado.Status}");
```
{{end}}
