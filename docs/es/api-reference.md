# Referencia de API

Tabla completa de los 42 endpoints disponibles en el SDK, agrupados por sub-cliente. URL base: `https://api.onesend2u.com`.

## Endpoints por sub-cliente

### Notificaciones — `client.Notifications` (4 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Notifications.SendAsync(request)` | POST | `/api/app/notifications/send` |
| `client.Notifications.GetListAsync(request)` | GET | `/api/app/notifications` |
| `client.Notifications.GetAsync(id)` | GET | `/api/app/notifications/{id}` |
| `client.Notifications.GetWithDetailsAsync(id)` | GET | `/api/app/notifications/with-navigation-properties/{id}` |

**Notas de autenticación:**
- `SendAsync` — requiere solo una API Key válida (sin permisos adicionales)
- Los demás métodos — requieren el permiso `Cpaas.Notifications` asignado al API Key

---

### Plantillas — `client.Templates` (9 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Templates.GetListAsync(request)` | GET | `/api/app/template-configurations` |
| `client.Templates.GetAsync(id)` | GET | `/api/app/template-configurations/{id}` |
| `client.Templates.GetWithDetailsAsync(id)` | GET | `/api/app/template-configurations/with-navigation-properties/{id}` |
| `client.Templates.ValidateAsync(request)` | GET | `/api/app/template-configurations/validate` |
| `client.Templates.GetPreviewAsync(request)` | GET | `/api/app/template-configurations/preview` |
| `client.Templates.GetTemplateAsync(id)` | GET | `/api/app/templates/{id}` |
| `client.Templates.GetTemplateButtonsAsync(id)` | GET | `/api/app/templates/template-buttons` |
| `client.Templates.GetTemplateVariableDefinitionsAsync(id)` | GET | `/api/app/templates/template-variable-definitions` |
| `client.Templates.GetTemplateVariablesWithSectionsAsync(...)` | GET | `/api/app/templates/template-variables-with-sections` |

**Permisos requeridos:** `Cpaas.TemplateConfigurations`

---

### Mensajes — `client.Messages` (3 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Messages.GetListAsync(request)` | GET | `/api/app/messages` |
| `client.Messages.GetAsync(id)` | GET | `/api/app/messages/{id}` |
| `client.Messages.GetWithDetailsAsync(id)` | GET | `/api/app/messages/with-navigation-properties/{id}` |

**Permisos requeridos:** `Cpaas.Messages`

---

### Webhooks — `client.Webhooks` (7 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Webhooks.GetListAsync(request)` | GET | `/api/app/webhooks` |
| `client.Webhooks.GetAsync(id)` | GET | `/api/app/webhooks/{id}` |
| `client.Webhooks.CreateAsync(request)` | POST | `/api/app/webhooks` |
| `client.Webhooks.UpdateAsync(id, request)` | PUT | `/api/app/webhooks/{id}` |
| `client.Webhooks.DeleteAsync(id)` | DELETE | `/api/app/webhooks/{id}` |
| `client.Webhooks.TestAsync(webhook)` | POST | `/api/app/webhooks/test` |
| `client.Webhooks.RotateSigningSecretAsync(id)` | POST | `/api/app/webhooks/{id}/rotate-signing-secret` |

**Permisos requeridos:** `Cpaas.Webhooks`

---

### Registros de API — `client.ApiLogs` (2 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.ApiLogs.GetListAsync(request)` | GET | `/api/app/notification-api-logs` |
| `client.ApiLogs.GetAsync(id)` | GET | `/api/app/notification-api-logs/{id}` |

**Permisos requeridos:** `Cpaas.NotificationApiLogs`

---

### Contactos — `client.Contacts` (5 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Contacts.GetListAsync(request)` | GET | `/api/app/contacts` |
| `client.Contacts.GetAsync(id)` | GET | `/api/app/contacts/{id}` |
| `client.Contacts.CreateAsync(request)` | POST | `/api/app/contacts` |
| `client.Contacts.UpdateAsync(id, request)` | PUT | `/api/app/contacts/{id}` |
| `client.Contacts.DeleteAsync(id)` | DELETE | `/api/app/contacts/{id}` |

**Permisos requeridos:** `Cpaas.Contacts`

---

### Grupos de Contactos — `client.ContactsGroups` (6 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.ContactsGroups.GetListAsync(request)` | GET | `/api/app/contacts-groups` |
| `client.ContactsGroups.GetAsync(id)` | GET | `/api/app/contacts-groups/{id}` |
| `client.ContactsGroups.GetWithDetailsAsync(id)` | GET | `/api/app/contacts-groups/with-navigation-properties/{id}` |
| `client.ContactsGroups.CreateAsync(request)` | POST | `/api/app/contacts-groups` |
| `client.ContactsGroups.UpdateAsync(id, request)` | PUT | `/api/app/contacts-groups/{id}` |
| `client.ContactsGroups.DeleteAsync(id)` | DELETE | `/api/app/contacts-groups/{id}` |

**Permisos requeridos:** `Cpaas.ContactsGroups`

---

### Consentimientos — `client.Consents` (6 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Consents.GetListAsync(request)` | GET | `/api/app/consents` |
| `client.Consents.GetAsync(id)` | GET | `/api/app/consents/{id}` |
| `client.Consents.GetWithDetailsAsync(id)` | GET | `/api/app/consents/with-navigation-properties/{id}` |
| `client.Consents.CreateAsync(request)` | POST | `/api/app/consents` |
| `client.Consents.UpdateAsync(id, request)` | PUT | `/api/app/consents/{id}` |
| `client.Consents.DeleteAsync(id)` | DELETE | `/api/app/consents/{id}` |

**Permisos requeridos:** `Cpaas.Consents`

---

## Enums del SDK

### `NotificationStatus`

| Valor | Descripción |
|---|---|
| `Sending` | Notificación en proceso de envío |
| `Success` | Notificación enviada exitosamente |
| `Error` | Error al enviar la notificación |

### `MessageProcessState`

| Valor | Descripción |
|---|---|
| `Initial` | Creado, pendiente de procesamiento |
| `Discarded` | Descartado antes de enviar |
| `NotConsented` | Sin consentimiento del destinatario |
| `Pending` | En cola para envío |
| `Sending` | Siendo enviado al proveedor |
| `Success` | Enviado exitosamente |
| `Error` | Error en el envío |
| `Unknown` | Estado desconocido |

### `NotificationSource`

| Valor | Valor numérico | Descripción |
|---|---|---|
| `CPaaS` | 1 | Originado desde el portal web |
| `API` | 2 | Originado desde llamada a la API (SDK) |

### `NotificationApiLogStatus`

| Valor | Valor numérico | Descripción |
|---|---|---|
| `Received` | 1 | Request recibido |
| `ValidationError` | 2 | Error de validación |
| `Queued` | 3 | Encolado para procesamiento |
| `OK` | 10 | Procesado exitosamente |
| `Warning` | 11 | Procesado con advertencias |
| `Error` | 12 | Error de procesamiento |

### `EventType` (Webhooks)

| Valor | Descripción |
|---|---|
| `Consent` | Cambios de consentimiento |
| `Status` | Cambios de estado de mensajes |
| `UserAction` | Acciones de usuario (clics, respuestas) |
| `ErrorProcessingMessages` | Errores de procesamiento |

### `RetryStrategy` (Webhooks)

| Valor | Descripción |
|---|---|
| `FixedDelay` | Intervalo fijo entre reintentos |
| `ExponentialBackoff` | Intervalo exponencialmente creciente |
| `LinearBackoff` | Intervalo linealmente creciente |
| `Jitter` | Intervalo aleatorio |

### `TemplateType`

| Valor | Descripción |
|---|---|
| `Default` | Plantilla estándar |
| `Carousel` | Plantilla tipo carrusel (WhatsApp) |
| `Coupon` | Plantilla tipo cupón (WhatsApp) |

### `TemplateConfigurationApprovalState`

| Valor | Descripción |
|---|---|
| `NotRequired` | No requiere aprobación del proveedor |
| `UnSubmitted` | No enviada para revisión |
| `Appeal` | En proceso de apelación |
| `Pending` | Pendiente de aprobación |
| `Approved` | Aprobada y lista para usar |
| `Rejected` | Rechazada por el proveedor |
| `Paused` | Pausada temporalmente |
| `Disabled` | Deshabilitada |
| `Deleted` | Eliminada |

### `TemplateConfigurationValidationStatus`

| Valor | Valor numérico | Descripción |
|---|---|---|
| `None` | 0 | Ningún canal configurado |
| `Partial` | 1 | Algunos canales configurados |
| `All` | 2 | Todos los canales configurados |

### `TemplateConfigurationCategory`

| Valor | Descripción |
|---|---|
| `Utility` | Mensajes de utilidad (confirmaciones, alertas, actualizaciones) |
| `Marketing` | Mensajes de marketing y promociones |
| `Authentication` | Mensajes de autenticación (OTP, verificación) |

### `TemplateSection`

Identifica la sección de una plantilla donde aparece una variable:

| Valor | Descripción |
|---|---|
| `Subject` | Asunto (Email) |
| `Header` | Encabezado del mensaje |
| `Body` | Cuerpo principal del mensaje |
| `Footer` | Pie del mensaje |
| `Buttons` | Botones de acción (WhatsApp) |

### `TemplateMarketingCategory`

Subcategoría para plantillas de marketing (WhatsApp):

| Valor | Descripción |
|---|---|
| `Default` | Marketing estándar |
| `Catalog` | Catálogo de productos |
| `Flow` | Flujo interactivo |

### `TemplateServiceCategory`

Subcategoría para plantillas de utilidad (WhatsApp):

| Valor | Descripción |
|---|---|
| `Default` | Utilidad estándar |
| `Flow` | Flujo interactivo |

### `TemplateAuthenticationCategory`

Subcategoría para plantillas de autenticación (WhatsApp):

| Valor | Descripción |
|---|---|
| `Default` | Autenticación estándar |

### `NotificationVariablesType`

Indica cómo se resuelve el valor de una variable de plantilla:

| Valor | Descripción |
|---|---|
| `Template` | Valor definido en la plantilla |
| `Auto` | Resuelto automáticamente por la plataforma |
| `Property` | Tomado de una propiedad del destinatario |
| `TemplateName` | Nombre de la plantilla |
| `TemplateLanguage` | Idioma de la plantilla |
| `Optional` | Variable opcional; si no se provee se omite |

### `TemplateHeaderType` (WhatsApp)

| Valor | Descripción |
|---|---|
| `Text` | Header de texto |
| `Media` | Header con imagen, video o documento |
| `Location` | Header con ubicación geográfica |

### `TemplateButtonType` (WhatsApp)

| Valor | Descripción |
|---|---|
| `Attachment` | Archivo adjunto |
| `Subscribe` | Suscripción a comunicaciones |
| `Unsubscribe` | Cancelación de suscripción |
| `Link` | Enlace URL |
| `Call` | Llamada telefónica |
| `UserAction` | Acción de usuario personalizada |

---

## Headers de autenticación

Todos los endpoints requieren los siguientes headers:

| Header | Descripción | Ejemplo |
|---|---|---|
| `X-API-Key` | Clave de API | `sk_prefix.your-api-key` |
| `X-Tenant-Id` | ID del tenant | `your-tenant-id` |

## Headers de rate limit en respuestas

| Header | Descripción |
|---|---|
| `X-RateLimit-Limit` | Límite máximo de requests en la ventana |
| `X-RateLimit-Remaining` | Requests restantes en la ventana actual |
| `X-RateLimit-Reset` | Timestamp UTC de reset de la ventana |
| `Retry-After` | Segundos de espera (solo en respuestas 429) |

## Versionado de API

El SDK **no envía** el parámetro `?api-version` por defecto. El servidor resuelve automáticamente la versión más reciente de cada endpoint.

Si necesitas fijar una versión específica (para testing o compatibilidad), puedes añadir el parámetro manualmente en los requests:

```
GET /api/app/notifications?api-version=1.0
```

Los cambios que rompen compatibilidad se gestionan mediante incrementos de versión major en el SDK (SemVer).
