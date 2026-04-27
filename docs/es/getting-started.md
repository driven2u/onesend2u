````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# Primeros Pasos

## Prerequisitos

- .NET 10.0 o superior
- Una API Key válida de OneSend2U
- El ID de tu tenant (organización)

Para obtener tus credenciales, accede al portal de OneSend2U y ve a **Configuración → API Keys**.

## Instalación

{{if SDK == "csharp"}}
```bash
dotnet add package OneSend2U.Sdk
```
{{end}}

## Registro con Inyección de Dependencias (recomendado)

El modo DI incluye automáticamente políticas de **retry** y **circuit breaker** mediante `Microsoft.Extensions.Http.Resilience`.

{{if SDK == "csharp"}}
```csharp
// Program.cs o Startup.cs
builder.Services.AddOneSend2U(options =>
{
    options.BaseUrl = "https://api.onesend2u.com";
    options.ApiKey = "sk_prefix.your-api-key";
    options.TenantId = "your-tenant-id";
});
```
{{end}}

### Opciones de configuración

| Opción | Tipo | Por defecto | Descripción |
|---|---|---|---|
| `BaseUrl` | `string` | *(requerido)* | URL base de la API |
| `ApiKey` | `string` | `""` | Clave de API para el header `X-API-Key` |
| `TenantId` | `string` | `""` | ID del tenant para el header `X-Tenant-Id` |
| `Timeout` | `TimeSpan` | `30s` | Tiempo máximo de espera por solicitud HTTP |

### Carga desde configuración

{{if SDK == "csharp"}}
```csharp
// appsettings.json
{
  "OneSend2U": {
    "BaseUrl": "https://api.onesend2u.com",
    "ApiKey": "sk_prefix.your-api-key",
    "TenantId": "your-tenant-id"
  }
}
```

```csharp
// Program.cs
builder.Services.AddOneSend2U(options =>
{
    builder.Configuration.GetSection("OneSend2U").Bind(options);
});
```
{{end}}

## Uso sin Inyección de Dependencias

Para scripts o aplicaciones simples donde no se requiere DI. Este modo **no incluye** retry ni circuit breaker automáticos.

{{if SDK == "csharp"}}
```csharp
var client = new OneSend2UClient(new OneSend2UClientOptions
{
    BaseUrl = "https://api.onesend2u.com",
    ApiKey = "sk_prefix.your-api-key",
    TenantId = "your-tenant-id"
});

// Usar el cliente directamente
var result = await client.Notifications.SendAsync(request);
```
{{end}}

## Primera notificación: ejemplo completo

El siguiente ejemplo envía un SMS con variables de plantilla:

{{if SDK == "csharp"}}
```csharp
using OneSend2U.Sdk;
using OneSend2U.Sdk.Notifications.Models;
using OneSend2U.Sdk.Exceptions;

public class NotificationService(OneSend2UClient client)
{
    public async Task SendInvoiceNotificationAsync(string phoneNumber, string customerName, string invoiceNumber)
    {
        try
        {
            var result = await client.Notifications.SendAsync(new SendNotificationRequest
            {
                TransactionId = Guid.NewGuid().ToString(),  // ID único para rastreo
                Application = "myapp",                       // Código de aplicación (max 10 chars)
                Region = "br",                               // Código de región (alfanumérico, máx. 10 caracteres)
                Language = "pt-BR",                          // Código de idioma (max 5 chars)
                NotificationType = "trans",                  // Tipo de notificación (max 10 chars)
                NotificationSubtype = "invoice",             // Subtipo (max 10 chars)
                Recipients =
                [
                    new NotificationRecipient
                    {
                        Channel = "sms",
                        Recipient = phoneNumber
                    }
                ],
                TemplateVariables =
                [
                    new Dictionary<string, string>
                    {
                        { "customer_name", customerName },
                        { "invoice_number", invoiceNumber }
                    }
                ]
            });

            Console.WriteLine($"Estado: {result.Status}");
            Console.WriteLine($"Transaction ID: {result.TransactionId}");

            if (result.Warnings?.Count > 0)
            {
                foreach (var warning in result.Warnings)
                    Console.WriteLine($"Advertencia: {warning}");
            }
        }
        catch (OneSend2URateLimitException ex)
        {
            Console.WriteLine($"Rate limit alcanzado. Reintenta en {ex.RetryAfterSeconds}s.");
        }
        catch (OneSend2UApiException ex)
        {
            Console.WriteLine($"Error de API {(int)ex.StatusCode}: {ex.Message}");
        }
    }
}
```
{{end}}

## Inyección en servicios

Con el registro DI, inyecta `OneSend2UClient` directamente en tus servicios o controladores:

{{if SDK == "csharp"}}
```csharp
// Con DI, el cliente se inyecta automáticamente
public class MyController(OneSend2UClient client) : ControllerBase
{
    [HttpPost("notify")]
    public async Task<IActionResult> Notify([FromBody] NotifyRequest request)
    {
        var result = await client.Notifications.SendAsync(/* ... */);
        return Ok(result);
    }
}
```
{{end}}
