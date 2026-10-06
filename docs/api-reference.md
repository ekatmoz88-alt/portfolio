# API Reference

Extracted from [`openapi/openapi.yaml`](../openapi/openapi.yaml) — the specification is the source of truth.

Every endpoint follows the same layout, so a developer knows where to look regardless of which endpoint they open:

1. **Purpose** — what the endpoint does and when to use it.
2. **Request** — method, path, parameters table.
3. **Response** — schema table with types and nullability.
4. **Errors** — status codes this endpoint can return, mapped to the shared [error model](error-handling.md).
5. **Example** — copy-paste request and a real response body.

---

## List orders

Returns orders in reverse chronological order, paginated by cursor.

`GET /orders`

🔒 Scope: `orders:read`

### Request parameters

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `status` | query | string | no | Filters by status. Repeat for multiple values. One of `pending`, `paid`, `shipped`, `cancelled`, `refunded`. |
| `limit` | query | integer | no | Items per page, 1–100. Default `25`. |
| `cursor` | query | string | no | Cursor returned as `nextCursor` by the previous page. |

### Response `200 OK`

| Field | Type | Nullable | Description |
|---|---|---|---|
| `data` | array of object | no | Orders on this page. Fields follow the order schema below. |
| `nextCursor` | string | yes | Pass as `cursor` for the next page. `null` on the last page. |
| `hasMore` | boolean | no | `true` when further pages exist. |

### Errors

| Status | Code | When |
|---|---|---|
| `400` | `invalid_parameter` | `limit` out of range. |
| `401` | `unauthorized` | Missing or invalid access token. |
| `403` | `forbidden` | Token lacks the `orders:read` scope. |
| `429` | `rate_limited` | Too many requests. See [rate limits](developer-guide.md#rate-limits). |

### Example

```bash
curl -X GET "https://api.example.com/v1/orders?status=paid&limit=2" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

```json
{
  "data": [
    {
      "id": "ord_000000000042",
      "status": "paid",
      "currency": "EUR",
      "total": 12900,
      "customerId": "cus_000000000017",
      "createdAt": "2026-09-14T11:20:31Z"
    },
    {
      "id": "ord_000000000041",
      "status": "paid",
      "currency": "USD",
      "total": 4500,
      "customerId": "cus_000000000017",
      "createdAt": "2026-09-13T08:04:12Z"
    }
  ],
  "nextCursor": "eyJvZmZzZXQiOjJ9",
  "hasMore": true
}
```

---

## Retrieve an order

Returns a single order by its identifier.

`GET /orders/{orderId}`

🔒 Scope: `orders:read`

### Request parameters

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `orderId` | path | string | yes | Order identifier, format `ord_` + 12 characters. |
| `include` | query | string[] | no | Expands related resources. Allowed: `items`, `customer`, `payments`. |

### Response `200 OK`

| Field | Type | Nullable | Description |
|---|---|---|---|
| `id` | string | no | Order identifier. |
| `status` | string | no | One of `pending`, `paid`, `shipped`, `cancelled`, `refunded`. |
| `currency` | string | no | ISO 4217 code, three uppercase letters. |
| `total` | integer | no | Order total in minor units (cents). |
| `customerId` | string | no | Identifier of the customer who placed the order. |
| `items` | array of object | no | Present only when `include=items`. |
| `items[].sku` | string | no | Stock keeping unit. |
| `items[].quantity` | integer | no | Number of units. |
| `items[].unitPrice` | integer | no | Price per unit in minor units. |
| `createdAt` | string | no | Creation timestamp, RFC 3339. |

### Errors

| Status | Code | When |
|---|---|---|
| `401` | `unauthorized` | Missing or invalid access token. |
| `403` | `forbidden` | Token lacks the `orders:read` scope. |
| `404` | `order_not_found` | No order with the given `orderId`. |
| `429` | `rate_limited` | Too many requests. |

### Example

```bash
curl -X GET "https://api.example.com/v1/orders/ord_000000000042?include=items" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

```json
{
  "id": "ord_000000000042",
  "status": "paid",
  "currency": "EUR",
  "total": 12900,
  "customerId": "cus_000000000017",
  "items": [
    { "sku": "KB-88-BLK", "quantity": 1, "unitPrice": 12900 }
  ],
  "createdAt": "2026-09-14T11:20:31Z"
}
```

---

## Cancel an order

Cancels an order that has not shipped yet. Returns `409 order_already_shipped` once a shipment is registered — use the refund endpoint instead.

`DELETE /orders/{orderId}`

🔒 Scope: `orders:write`

### Request parameters

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `orderId` | path | string | yes | Order identifier, format `ord_` + 12 characters. |

### Response `200 OK`

Returns the cancelled order. Fields follow the order schema above; `status` is now `cancelled`.

### Errors

| Status | Code | When |
|---|---|---|
| `401` | `unauthorized` | Missing or invalid access token. |
| `403` | `forbidden` | Token lacks the `orders:write` scope. |
| `404` | `order_not_found` | No order with the given `orderId`. |
| `409` | `order_already_shipped` | The order can no longer be cancelled. Use refund instead. |

### Example

```bash
curl -X DELETE "https://api.example.com/v1/orders/ord_000000000040" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

```json
{
  "id": "ord_000000000040",
  "status": "cancelled",
  "currency": "EUR",
  "total": 3200,
  "customerId": "cus_000000000017",
  "createdAt": "2026-09-12T19:44:05Z"
}
```
