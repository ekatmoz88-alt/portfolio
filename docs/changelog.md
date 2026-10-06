# Changelog

Documentation versions follow the API version. Every entry states what changed, whether it breaks clients, and what the developer must do.

Format: `Added` · `Changed` · `Deprecated` · `Removed` · `Fixed`.

---

## 1.2.0 — 2026-09-14

**Added**
- Webhook documentation: event types, HMAC signature verification, retry policy, deduplication guidance.
- `webhooks` section in the OpenAPI specification — events are now declared in the spec alongside `paths`.
- `order.shipped` and `order.cancelled` events.

**Changed**
- Error model unified across all endpoints. Every error now returns `error.code`, `error.message`, `error.param` and `error.requestId`.
- `GET /orders/{orderId}` gained the `include` parameter. Previously `items` were always returned.

**Deprecated**
- `GET /orders?page=` offset pagination. Still works until 1.4.0, but `cursor` is the supported path.

**Fixed**
- `total` was documented as a decimal string; it is an integer in minor units. Corrected in the reference and the spec schema.

**Migration note**
- Clients parsing `error.message` must switch to `error.code`. Message wording is not stable.

---

## 1.1.0 — 2026-08-02

**Added**
- `DELETE /orders/{orderId}` for cancelling orders that have not shipped.
- `order_already_shipped` error code with `409` status.
- Rate limit table per plan, with `Retry-After` semantics.

**Changed**
- Access token lifetime reduced from 24 hours to 1 hour. Clients must cache and refresh rather than requesting a token per call.

**Fixed**
- `currency` was documented without the ISO 4217 constraint; the pattern is now enforced in the spec.

---

## 1.0.0 — 2026-07-01

Initial published version.

**Added**
- `GET /orders` with cursor pagination.
- `GET /orders/{orderId}` with optional `include`.
- Order, OrderItem and Error schemas under `components`.
- Developer guide: credentials, token, first request, pagination.
- OpenAPI 3.1 specification validated in CI on every pull request.
