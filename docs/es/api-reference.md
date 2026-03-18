# Referencia de API

Tabla completa de los 40 endpoints disponibles en el SDK, agrupados por sub-cliente. URL base: `https://api.onesend2u.com`.

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

### Webhooks — `client.Webhooks` (6 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.Webhooks.GetListAsync(request)` | GET | `/api/app/webhooks` |
| `client.Webhooks.GetAsync(id)` | GET | `/api/app/webhooks/{id}` |
| `client.Webhooks.CreateAsync(request)` | POST | `/api/app/webhooks` |
| `client.Webhooks.UpdateAsync(id, request)` | PUT | `/api/app/webhooks/{id}` |
| `client.Webhooks.DeleteAsync(id)` | DELETE | `/api/app/webhooks/{id}` |
| `client.Webhooks.TestAsync(request)` | POST | `/api/app/webhooks/test` |

**Permisos requeridos:** `Cpaas.Webhooks`

---

### Registros de API — `client.ApiLogs` (2 endpoints)

| Método SDK | Método HTTP | Ruta del endpoint |
|---|---|---|
| `client.ApiLogs.GetListAsync(request)` | GET | `/api/app/notification-api-logs` |
| `client.ApiLogs.GetAsync(id)` | GET | `/api/app/notification-api-logs/{id}` |

**Permisos requeridos:** `Cpaas.ApiLogs`

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

| Valor | Descripción |
|---|---|
| `CPaaS` | Originado desde el portal web CPaaS |
| `Api` | Originado desde llamada a la API (SDK) |

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
| `Sms` | Plantilla de SMS |
| `Email` | Plantilla de Email |
| `WhatsApp` | Plantilla de WhatsApp |

### `TemplateConfigurationApprovalState`

| Valor | Descripción |
|---|---|
| `Pending` | Pendiente de aprobación |
| `Approved` | Aprobada |
| `Rejected` | Rechazada |

### `TemplateConfigurationValidationStatus`

| Valor | Descripción |
|---|---|
| `Valid` | Plantilla válida |
| `Invalid` | Plantilla con errores |
| `NotValidated` | Sin validar |

### `TemplateHeaderType` (WhatsApp)

| Valor | Descripción |
|---|---|
| `Text` | Header de texto |
| `Image` | Header de imagen |
| `Document` | Header de documento |
| `Video` | Header de video |

### `TemplateButtonType` (WhatsApp)

| Valor | Descripción |
|---|---|
| `QuickReply` | Respuesta rápida |
| `CallToAction` | Llamada a la acción (URL, teléfono) |

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
