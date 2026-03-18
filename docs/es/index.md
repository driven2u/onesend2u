````json
//[doc-params]
{
    "SDK": ["csharp"]
}
````

# OneSend2U SDK — Documentación en Español

Bienvenido a la documentación oficial del SDK de **OneSend2U**, la plataforma CPaaS (Communications Platform as a Service) de Driven2U para el envío de notificaciones multicanal.

## ¿Qué es OneSend2U?

OneSend2U es una plataforma de comunicaciones que permite enviar mensajes a través de múltiples canales desde una sola integración: **SMS**, **Email** y **WhatsApp**. Soporta múltiples proveedores (Twilio, Infobip, SMTP) con selección automática según la configuración de cada plantilla.

## ¿Qué ofrece el SDK?

El paquete NuGet `OneSend2U.Sdk` es un cliente .NET tipado que cubre los **41 endpoints** de la API REST de OneSend2U. Está diseñado para cualquier aplicación .NET 10+ sin dependencias adicionales de frameworks externos.

### Características principales

| Característica | Descripción |
|---|---|
| **SMS** | Envío de mensajes de texto vía Twilio e Infobip |
| **Email** | Envío de correos vía SendGrid, Infobip y SMTP |
| **WhatsApp** | Mensajería WhatsApp Business vía Twilio e Infobip |
| **Multi-proveedor** | Selección automática de proveedor por plantilla |
| **Webhooks** | Recepción de eventos de estado en tiempo real |
| **Firma HMAC-SHA256** | Verificación de autenticidad de payloads entrantes |
| **Resiliencia integrada** | Retry automático y circuit breaker (modo DI) |
| **Sin dependencias externas** | Compatible con cualquier aplicación .NET 10+ |
| **41 endpoints** | Notificaciones, plantillas, mensajes, webhooks, contactos, consentimientos y logs |

## Modelo de autenticación

El SDK utiliza **autenticación por API Key** mediante dos headers HTTP:

| Header | Descripción |
|---|---|
| `X-API-Key` | Clave de API generada desde el portal OneSend2U |
| `X-Tenant-Id` | Identificador del tenant (organización) |

No se requiere OAuth ni tokens de sesión. Es el modelo ideal para integraciones máquina a máquina.

## Sub-clientes disponibles

El cliente principal `OneSend2UClient` expone los siguientes sub-clientes:

| Sub-cliente | Endpoints | Descripción |
|---|---|---|
| `client.Notifications` | 4 | Envío y consulta de notificaciones |
| `client.Templates` | 9 | Configuraciones de plantillas |
| `client.Messages` | 3 | Seguimiento de mensajes individuales |
| `client.Webhooks` | 7 | Gestión de webhooks |
| `client.ApiLogs` | 2 | Registros de llamadas a la API |
| `client.Contacts` | 5 | Gestión de contactos |
| `client.ContactsGroups` | 6 | Gestión de grupos de contactos |
| `client.Consents` | 6 | Gestión de consentimientos |

## Enlace rápido de instalación

{{if SDK == "csharp"}}
```bash
dotnet add package OneSend2U.Sdk
```
{{end}}

## Navegación de la documentación

- [Primeros Pasos](getting-started.md) — Instalación, configuración y primer envío
- [Notificaciones](notifications.md) — Envío y consulta de notificaciones
- [Plantillas](templates.md) — Configuración y validación de plantillas
- [Mensajes](messages.md) — Seguimiento de estado de mensajes
- [Webhooks](webhooks.md) — Creación y gestión de webhooks
- [Verificación de Firmas](webhook-verification.md) — Validación HMAC-SHA256
- [Contactos y Grupos](contacts.md) — Gestión de contactos
- [Consentimientos](consents.md) — Gestión de consentimientos GDPR
- [Registros de API](api-logs.md) — Auditoría de llamadas
- [Manejo de Errores](error-handling.md) — Excepciones y reintentos
- [Referencia de API](api-reference.md) — Tabla completa de endpoints y enums
