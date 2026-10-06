# Webhooks

Webhooks push order events to your endpoint instead of making you poll. They are declared in [`openapi/openapi.yaml`](../openapi/openapi.yaml) under the root `webhooks` field — the same specification that generates the API reference.

## Events

| Event | Fires when | Payload |
|---|---|---|
| `order.created` | An order is placed. | Full order object, status `pending`. |
| `order.paid` | Payment is captured. | Full order object, status `paid`. |
| `order.shipped` | A shipment is registered. | Full order object, status `shipped`. |
| `order.cancelled` | An order is cancelled. | Full order object, status `cancelled`. |

## Event payload

```json
{
  "id": "evt_000000000311",
  "type": "order.paid",
  "createdAt": "2026-09-14T11:20:33Z",
  "data": {
    "id": "ord_000000000042",
    "status": "paid",
    "currency": "EUR",
    "total": 12900,
    "customerId": "cus_000000000017",
    "createdAt": "2026-09-14T11:20:31Z"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique event identifier. **Deduplicate on this field.** |
| `type` | string | One of the four event types above. |
| `createdAt` | string | When the event occurred, RFC 3339. |
| `data` | object | The order in its state at event time. |

## Subscribing

Register an HTTPS endpoint under **Settings → Webhooks** in the dashboard. Requirements:

- HTTPS with a valid certificate.
- Responds with `2xx` within **5 seconds**.
- Publicly reachable.

## Verifying the signature

Every delivery carries a signature header:

```
X-Signature: t=1757836800,v1=5f8d2c9a...
```

| Part | Meaning |
|---|---|
| `t` | Unix timestamp when the signature was produced. |
| `v1` | HMAC-SHA256 of `t` + `.` + raw request body, hex-encoded. |

Verify before trusting the payload:

1. Concatenate `t`, a `.`, and the **raw** request body. Do not re-serialize the parsed JSON — key order and whitespace differ, and the signature will not match.
2. Compute HMAC-SHA256 using your webhook secret as the key.
3. Compare to `v1` with a constant-time comparison.
4. Reject the delivery if `t` is older than **5 minutes** — this blocks replay attacks.

```js
import crypto from "node:crypto";

function verify(rawBody, header, secret) {
  const parts = Object.fromEntries(header.split(",").map((s) => s.split("=")));
  const { t, v1 } = parts;

  if (Math.abs(Date.now() / 1000 - Number(t)) > 300) {
    throw new Error("Timestamp too old");
  }

  const expected = crypto
    .createHmac("sha256", secret)
    .update(`${t}.${rawBody}`)
    .digest("hex");

  const a = Buffer.from(expected);
  const b = Buffer.from(v1);

  if (a.length !== b.length || !crypto.timingSafeEqual(a, b)) {
    throw new Error("Signature mismatch");
  }

  return true;
}
```

## Responding

| Your response | What happens |
|---|---|
| `2xx` within 5 s | Delivery acknowledged. |
| Any other status, or timeout | Delivery retried. |

**Process asynchronously.** Acknowledge first, then handle the event in a background job. A slow handler causes redelivery of an event you already handled.

## Retry policy

Delivery is **at-least-once**. Retries run for 24 hours with exponential backoff.

| Attempt | Delay after previous |
|---|---|
| 1 | immediate |
| 2 | 30 s |
| 3 | 2 min |
| 4 | 10 min |
| 5 | 1 h |
| 6+ | 6 h, up to 24 h total |

Because delivery is at-least-once, the same `id` can arrive more than once. **Always deduplicate on `id`.** Do not deduplicate on `type` plus timestamp — legitimate distinct events can share both.

## Ordering

Events are not guaranteed to arrive in the order they occurred. `order.created` can land after `order.paid` under retry or network delay. Use the order's `status` field from the payload, or re-fetch the order with `GET /orders/{orderId}`, rather than assuming the arrival order reflects the sequence.
