# Error Handling

Every error in this API has the same shape. A client can rely on one parser and one handling path regardless of which endpoint failed.

## Error format

```json
{
  "error": {
    "code": "order_not_found",
    "message": "No order with the given identifier.",
    "param": "orderId",
    "requestId": "req_9f2c41ab77"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `error.code` | string | Machine-readable identifier. Stable — safe to branch on. |
| `error.message` | string | Human-readable explanation. Not safe to parse; wording may change. |
| `error.param` | string | Name of the offending parameter, when the error is caused by one. Optional. |
| `error.requestId` | string | Identifier of the request. Quote it in support tickets. |

**Rule:** branch on `code`, display `message`, log `requestId`.

## HTTP status codes

| Status | Meaning | Retry? |
|---|---|---|
| `400` | Malformed request or invalid parameter. | No — fix the request. |
| `401` | Missing, expired or invalid access token. | No — obtain a new token. |
| `403` | Authenticated, but the token lacks the required scope. | No — request a wider scope. |
| `404` | Resource does not exist or is not visible to this token. | No. |
| `409` | Conflicts with current state, e.g. cancelling a shipped order. | No — resolve the conflict. |
| `422` | Semantically valid request, but business rules reject it. | No. |
| `429` | Rate limit exceeded. | Yes — after `Retry-After`. |
| `500` | Unexpected server error. | Yes — with backoff. |
| `503` | Temporarily unavailable. | Yes — with backoff. |

## Error codes

| Code | Status | Cause | Resolution |
|---|---|---|---|
| `invalid_parameter` | 400 | A parameter is missing, out of range or wrongly formatted. | Check `error.param` against the parameter table for the endpoint. |
| `unauthorized` | 401 | Token missing or expired. | Request a new access token and retry once. |
| `forbidden` | 403 | Token lacks the required scope. | Obtain a token with the scope listed for the endpoint. |
| `order_not_found` | 404 | No order with the given identifier. | Verify `orderId`; it may belong to another environment. |
| `order_already_shipped` | 409 | The order can no longer be cancelled. | Use the refund endpoint instead. |
| `insufficient_stock` | 422 | Requested quantity exceeds available stock. | Reduce quantity or wait for restock. |
| `rate_limited` | 429 | Too many requests in the window. | Wait for `Retry-After`, then retry with backoff. |
| `internal_error` | 500 | Unexpected failure. | Retry with backoff; quote `requestId` if it persists. |

## Handling pattern

Retry only those codes that are worth retrying. Branch on `code`, never on `message`.

```js
async function callApi(url, init) {
  const res = await fetch(url, init);

  if (res.ok) return res.json();

  const { error } = await res.json();

  if (error.code === "rate_limited") {
    const wait = Number(res.headers.get("Retry-After") ?? 1) * 1000;
    await new Promise((r) => setTimeout(r, wait));
    return callApi(url, init);
  }

  if (error.code === "unauthorized") {
    await refreshToken();
    return callApi(url, init);
  }

  throw new ApiError(error);
}
```

## What is safe to rely on

| Element | Guarantee |
|---|---|
| `error.code` | Stable. New codes may be added; existing ones do not change meaning. |
| HTTP status | Stable for a given code. |
| `error.requestId` | Always present. Required for support. |
| `error.message` | Not stable. Wording and language may change — never parse it. |
| `error.param` | Present only when a single parameter caused the error. |
