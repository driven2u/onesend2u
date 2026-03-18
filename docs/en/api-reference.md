# API Reference

Complete reference for all 40 SDK endpoints, enum definitions, and rate limit headers.

## Endpoint table

Base URL: `https://api.onesend2u.com`

### Notifications (4)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.Notifications.SendAsync()` | POST | `/api/app/notifications/send` |
| `client.Notifications.GetListAsync()` | GET | `/api/app/notifications` |
| `client.Notifications.GetAsync(id)` | GET | `/api/app/notifications/{id}` |
| `client.Notifications.GetWithDetailsAsync(id)` | GET | `/api/app/notifications/with-navigation-properties/{id}` |

**Permissions**: `SendAsync` requires any valid API key. The read endpoints require the `Cpaas.Notifications` permission assigned to the API key user.

### Templates (9)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.Templates.GetListAsync()` | GET | `/api/app/template-configurations` |
| `client.Templates.GetAsync(id)` | GET | `/api/app/template-configurations/{id}` |
| `client.Templates.GetWithDetailsAsync(id)` | GET | `/api/app/template-configurations/with-navigation-properties/{id}` |
| `client.Templates.ValidateAsync()` | GET | `/api/app/template-configurations/validate` |
| `client.Templates.GetPreviewAsync(id)` | GET | `/api/app/template-configurations/preview` |
| `client.Templates.GetTemplateAsync(id)` | GET | `/api/app/templates/{id}` |
| `client.Templates.GetTemplateButtonsAsync(id)` | GET | `/api/app/templates/template-buttons` |
| `client.Templates.GetTemplateVariableDefinitionsAsync(id)` | GET | `/api/app/templates/template-variable-definitions` |
| `client.Templates.GetTemplateVariablesWithSectionsAsync(...)` | GET | `/api/app/templates/template-variables-with-sections` |

**Permissions**: Require the `Cpaas.TemplateConfigurations` permission.

### Messages (3)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.Messages.GetListAsync()` | GET | `/api/app/messages` |
| `client.Messages.GetAsync(id)` | GET | `/api/app/messages/{id}` |
| `client.Messages.GetWithDetailsAsync(id)` | GET | `/api/app/messages/with-navigation-properties/{id}` |

**Permissions**: Require the `Cpaas.Messages` permission.

### Webhooks (6)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.Webhooks.GetListAsync()` | GET | `/api/app/webhooks` |
| `client.Webhooks.GetAsync(id)` | GET | `/api/app/webhooks/{id}` |
| `client.Webhooks.CreateAsync()` | POST | `/api/app/webhooks` |
| `client.Webhooks.UpdateAsync(id)` | PUT | `/api/app/webhooks/{id}` |
| `client.Webhooks.DeleteAsync(id)` | DELETE | `/api/app/webhooks/{id}` |
| `client.Webhooks.TestAsync()` | POST | `/api/app/webhooks/test` |

**Permissions**: Require the `Cpaas.Webhooks` permission.

### API Logs (2)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.ApiLogs.GetListAsync()` | GET | `/api/app/notification-api-logs` |
| `client.ApiLogs.GetAsync(id)` | GET | `/api/app/notification-api-logs/{id}` |

**Permissions**: Require the `Cpaas.NotificationApiLogs` permission.

### Contacts (5)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.Contacts.GetListAsync()` | GET | `/api/app/contacts` |
| `client.Contacts.GetAsync(id)` | GET | `/api/app/contacts/{id}` |
| `client.Contacts.CreateAsync()` | POST | `/api/app/contacts` |
| `client.Contacts.UpdateAsync(id)` | PUT | `/api/app/contacts/{id}` |
| `client.Contacts.DeleteAsync(id)` | DELETE | `/api/app/contacts/{id}` |

**Permissions**: Require the `Cpaas.Contacts` permission.

### Contacts Groups (6)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.ContactsGroups.GetListAsync()` | GET | `/api/app/contacts-groups` |
| `client.ContactsGroups.GetAsync(id)` | GET | `/api/app/contacts-groups/{id}` |
| `client.ContactsGroups.GetWithDetailsAsync(id)` | GET | `/api/app/contacts-groups/with-navigation-properties/{id}` |
| `client.ContactsGroups.CreateAsync()` | POST | `/api/app/contacts-groups` |
| `client.ContactsGroups.UpdateAsync(id)` | PUT | `/api/app/contacts-groups/{id}` |
| `client.ContactsGroups.DeleteAsync(id)` | DELETE | `/api/app/contacts-groups/{id}` |

**Permissions**: Require the `Cpaas.ContactsGroups` permission.

### Consents (6)

| SDK Method | HTTP Method | Endpoint |
|---|---|---|
| `client.Consents.GetListAsync()` | GET | `/api/app/consents` |
| `client.Consents.GetAsync(id)` | GET | `/api/app/consents/{id}` |
| `client.Consents.GetWithDetailsAsync(id)` | GET | `/api/app/consents/with-navigation-properties/{id}` |
| `client.Consents.CreateAsync()` | POST | `/api/app/consents` |
| `client.Consents.UpdateAsync(id)` | PUT | `/api/app/consents/{id}` |
| `client.Consents.DeleteAsync(id)` | DELETE | `/api/app/consents/{id}` |

**Permissions**: Require the `Cpaas.Consents` permission.

---

## Enums

### EventType

Used in webhook `EventTypes` filter.

| Value | Description |
|---|---|
| `Consent` | Consent-related events |
| `Status` | Message state change events |
| `UserAction` | User interaction events (e.g., WhatsApp button clicks) |
| `ErrorProcessingMessages` | Errors during message processing |

### MessageProcessState

Message delivery state. Used in webhook `Statuses` filter and `MessageResponse.MessageProcessState`.

| Value | Description |
|---|---|
| `Initial` | Created but not yet processed |
| `Discarded` | Discarded before sending |
| `NotConsented` | Recipient has not consented |
| `Pending` | Queued for sending |
| `Sending` | Currently being sent |
| `Success` | Sent successfully |
| `Error` | Sending failed |
| `Unknown` | State undetermined |

### NotificationStatus

High-level notification status (`NotificationResponse.Status`).

| Value | Description |
|---|---|
| `Sending` | Notification is being sent |
| `Success` | All messages delivered |
| `Error` | One or more messages failed |

### NotificationApiLogStatus

API log entry processing status (`ApiLogResponse.Status`).

| Value | Int | Description |
|---|---|---|
| `Received` | 1 | Request received |
| `ValidationError` | 2 | Request failed validation |
| `Queued` | 3 | Request queued for processing |
| `OK` | 10 | Processed successfully |
| `Warning` | 11 | Processed with warnings |
| `Error` | 12 | Processing failed |

### RetryStrategy

Webhook retry timing strategy.

| Value | Description |
|---|---|
| `FixedDelay` | Fixed delay between retry attempts |

### WebhookSignatureValidationError

Error type returned by `WebhookSignatureValidator.Validate()` on failure.

| Value | Description |
|---|---|
| `InvalidParameters` | Required parameters are null or empty |
| `InvalidTimestamp` | Timestamp header is not a valid Unix integer |
| `TimestampOutOfTolerance` | Timestamp exceeds the tolerance window |
| `InvalidSignatureFormat` | Signature header does not match `v1={hex}` format |
| `InvalidSignature` | HMAC does not match |

---

## Rate limit headers

| Header | Description |
|---|---|
| `X-RateLimit-Limit` | Maximum requests allowed in the current window |
| `X-RateLimit-Remaining` | Requests remaining in the current window |
| `X-RateLimit-Reset` | UTC timestamp when the window resets |
| `Retry-After` | Seconds to wait before retrying (present on 429 responses) |

These headers are parsed automatically by the SDK into `RateLimitInfo` and exposed on `OneSend2UApiException.RateLimit` and `OneSend2URateLimitException.RetryAfterSeconds`.

---

## Authentication headers

| Header | Description |
|---|---|
| `X-API-Key` | Your API key |
| `X-Tenant-Id` | Your tenant identifier |

Both headers are injected automatically by the SDK on every request.

---

## API versioning

The SDK does not send an `api-version` query parameter. The server automatically resolves each endpoint to its latest version. Breaking changes are handled via SDK major version bumps (SemVer).

You can still pin a version manually using Postman or curl with `?api-version=1.0`.
