````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Consentimientos

El sub-cliente `client.Consents` gestiona los consentimientos de destinatarios para recibir comunicaciones. Cubre 6 endpoints con CRUD completo.

## Contexto: cumplimiento normativo

Los consentimientos permiten cumplir con regulaciones de privacidad y comunicaciones:

- **GDPR** (Reglamento General de Protección de Datos — Europa)
- **LGPD** (Lei Geral de Proteção de Dados — Brasil)
- **CAN-SPAM / TCPA** (Estados Unidos)
- Regulaciones de telecomunicaciones locales

Antes de enviar una notificación, OneSend2U puede verificar si el destinatario ha dado consentimiento. Si el destinatario no ha consentido, el mensaje resultante tendrá el estado `NotConsented` en lugar de ser enviado.

## Métodos disponibles

| Método SDK | HTTP | Descripción |
|---|---|---|
| `GetListAsync(request)` | GET | Lista consentimientos con filtros y paginación |
| `GetAsync(id)` | GET | Obtiene un consentimiento por ID |
| `GetWithDetailsAsync(id)` | GET | Obtiene un consentimiento con propiedades de navegación |
| `CreateAsync(request)` | POST | Registra un nuevo consentimiento |
| `UpdateAsync(id, request)` | PUT | Actualiza un consentimiento existente |
| `DeleteAsync(id)` | DELETE | Elimina un consentimiento |

## Modelo de consentimiento

Un consentimiento vincula un destinatario con un contexto específico:

| Campo | Descripción |
|---|---|
| `Recipient` | Dirección del destinatario (teléfono, email) |
| `IsConsented` | `true` = ha consentido, `false` = ha revocado |
| `RegionId` | Región a la que aplica el consentimiento |
| `ApplicationId` | Aplicación a la que aplica |
| `NotificationTypeId` | Tipo de notificación para el que aplica |
| `NotificationSubtypeId` | Subtipo de notificación para el que aplica |
| `ChannelTypeId` | Canal al que aplica (SMS, Email, WhatsApp) |
| `DeploymentEnvironmentId` | Entorno de despliegue |

Los campos de contexto son opcionales: cuantos más estén definidos, más específico es el consentimiento.

## Registrar un consentimiento

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk.Consents.Models;

// Consentimiento genérico para todos los canales y tipos
var consentimiento = await client.Consents.CreateAsync(new CreateConsentRequest
{
    Recipient = "+5215512345678",
    IsConsented = true
});

Console.WriteLine($"Consentimiento registrado: {consentimiento.Id}");
```
{{end}}

### Consentimiento específico por canal y tipo

{{if SDK == "csharp"}}
```csharp
// Consentimiento solo para SMS de tipo marketing
var consentimiento = await client.Consents.CreateAsync(new CreateConsentRequest
{
    Recipient = "+5215512345678",
    IsConsented = true,
    ApplicationId = appId,
    NotificationTypeId = marketingTypeId,
    ChannelTypeId = smsChannelId
});
```
{{end}}

### Revocar un consentimiento

{{if SDK == "csharp"}}
```csharp
// Registrar la revocación del consentimiento
var revocacion = await client.Consents.CreateAsync(new CreateConsentRequest
{
    Recipient = "+5215512345678",
    IsConsented = false,   // El destinatario ha retirado su consentimiento
    ApplicationId = appId
});
```
{{end}}

## Modelo de creación

{{if SDK == "csharp"}}
```csharp
public class CreateConsentRequest
{
    public string? Recipient { get; set; }              // Dirección del destinatario. Requerido.
    public bool IsConsented { get; set; }               // true = consentido. Por defecto: true.
    public Guid? RegionId { get; set; }
    public Guid? ApplicationId { get; set; }
    public Guid? NotificationTypeId { get; set; }
    public Guid? NotificationSubtypeId { get; set; }
    public Guid? ChannelTypeId { get; set; }
    public Guid? DeploymentEnvironmentId { get; set; }
}
```
{{end}}

## Listar consentimientos

{{if SDK == "csharp"}}
```csharp
var resultado = await client.Consents.GetListAsync(new GetConsentsRequest
{
    FilterText = "+521551",
    IsConsented = true,
    SkipCount = 0,
    MaxResultCount = 50
});

Console.WriteLine($"Total consentimientos activos: {resultado.TotalCount}");
foreach (var c in resultado.Items)
{
    Console.WriteLine($"{c.Recipient} — Consented: {c.IsConsented} — {c.CreationTime}");
}
```
{{end}}

## Obtener un consentimiento por ID

{{if SDK == "csharp"}}
```csharp
var consentimiento = await client.Consents.GetAsync(consentimientoId);

Console.WriteLine($"Destinatario: {consentimiento.Recipient}");
Console.WriteLine($"Consentido: {consentimiento.IsConsented}");
Console.WriteLine($"Creado: {consentimiento.CreationTime}");
```
{{end}}

## Actualizar un consentimiento

{{if SDK == "csharp"}}
```csharp
var actual = await client.Consents.GetAsync(consentimientoId);

await client.Consents.UpdateAsync(consentimientoId, new UpdateConsentRequest
{
    Recipient = actual.Recipient,
    IsConsented = false,                         // Revocar consentimiento
    ConcurrencyStamp = actual.ConcurrencyStamp   // Requerido
});
```
{{end}}

## Obtener con propiedades de navegación

{{if SDK == "csharp"}}
```csharp
// Incluye datos expandidos: aplicación, canal, tipo, subtipo, etc.
var detalle = await client.Consents.GetWithDetailsAsync(consentimientoId);
```
{{end}}

## Eliminar un consentimiento

{{if SDK == "csharp"}}
```csharp
await client.Consents.DeleteAsync(consentimientoId);
```
{{end}}

## Flujo recomendado: captura de consentimiento

{{if SDK == "csharp"}}
```csharp
// Al registrar un usuario nuevo en tu sistema
public async Task RegisterUserAsync(string phone, string email, bool acceptsMarketing)
{
    // Siempre registrar consentimiento para notificaciones transaccionales
    await client.Consents.CreateAsync(new CreateConsentRequest
    {
        Recipient = phone,
        IsConsented = true,
        NotificationTypeId = transactionalTypeId,
        ChannelTypeId = smsChannelId
    });

    // Solo si el usuario aceptó marketing
    if (acceptsMarketing)
    {
        await client.Consents.CreateAsync(new CreateConsentRequest
        {
            Recipient = email,
            IsConsented = true,
            NotificationTypeId = marketingTypeId,
            ChannelTypeId = emailChannelId
        });
    }
}

// Al procesar una solicitud de baja
public async Task UnsubscribeAsync(string phone)
{
    // Registrar la revocación del consentimiento de marketing
    await client.Consents.CreateAsync(new CreateConsentRequest
    {
        Recipient = phone,
        IsConsented = false,
        NotificationTypeId = marketingTypeId
    });

    Console.WriteLine($"Consentimiento de marketing revocado para {phone}");
}
```
{{end}}

## Modelo de respuesta

{{if SDK == "csharp"}}
```csharp
public class ConsentResponse
{
    public Guid Id { get; set; }
    public string? Recipient { get; set; }
    public bool IsConsented { get; set; }
    public Guid? RegionId { get; set; }
    public Guid? ApplicationId { get; set; }
    public Guid? NotificationTypeId { get; set; }
    public Guid? NotificationSubtypeId { get; set; }
    public Guid? ChannelTypeId { get; set; }
    public Guid? DeploymentEnvironmentId { get; set; }
    public string? ConcurrencyStamp { get; set; }
    public DateTime CreationTime { get; set; }
    public DateTime? LastModificationTime { get; set; }
}
```
{{end}}
