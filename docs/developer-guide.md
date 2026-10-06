# Developer Guide

Get from zero to your first successful API call in about five minutes.

1. [Get credentials](#1-get-credentials)
2. [Obtain an access token](#2-obtain-an-access-token)
3. [Make your first request](#3-make-your-first-request)
4. [Move through result sets](#4-move-through-result-sets)
5. [Receive webhooks](#5-receive-webhooks)

## 1. Get credentials

Request a client ID and secret in the dashboard under **Settings → API access**. Credentials are scoped per environment — sandbox and production keys are not interchangeable.

## 2. Obtain an access token

```bash
curl -X POST "https://api.example.com/v1/oauth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" \
  -d "scope=orders:read orders:write"
```

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Tokens live for one hour. Cache them and refresh before expiry — do not request a new token per call.

## 3. Make your first request

```bash
curl -X GET "https://api.example.com/v1/orders?limit=1" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

A `200` with a `data` array means the integration works. If something failed, see [error handling](error-handling.md).

## 4. Move through result sets

List endpoints are cursor-paginated. Pass `nextCursor` from the previous response as `cursor` until `hasMore` is `false`.

```bash
cursor=""
while :; do
  url="https://api.example.com/v1/orders?limit=100"
  [ -n "$cursor" ] && url="$url&cursor=$cursor"

  body=$(curl -s -X GET "$url" -H "Authorization: Bearer $ACCESS_TOKEN")

  echo "$body"

  has_more=$(echo "$body" | jq -r '.hasMore')
  [ "$has_more" = "true" ] || break

  cursor=$(echo "$body" | jq -r '.nextCursor')
done
```

Do not compute offsets yourself. The cursor encodes the sort position, so new records will not shift results you have already seen.

## 5. Receive webhooks

Subscribe an HTTPS endpoint under **Settings → Webhooks**, then verify the signature on every delivery before trusting the payload. Full details — event types, HMAC verification, retry policy and deduplication — are in [webhooks.md](webhooks.md).

```js
app.post("/webhooks", express.raw({ type: "application/json" }), (req, res) => {
  try {
    verify(req.body.toString(), req.headers["x-signature"], SECRET);
  } catch {
    return res.status(400).send("bad signature");
  }

  const event = JSON.parse(req.body);
  enqueue(event);      // handle asynchronously
  res.status(200).end(); // acknowledge immediately
});
```

## Rate limits

| Plan | Requests per minute | Burst |
|---|---|---|
| Sandbox | 60 | 10 |
| Production | 600 | 50 |

Exceeding the limit returns `429 rate_limited` with a `Retry-After` header in seconds. Back off for that long, then resume.

## Environments

| Environment | Base URL | Data |
|---|---|---|
| Sandbox | `https://api.sandbox.example.com/v1` | Test data, reset weekly. |
| Production | `https://api.example.com/v1` | Live data. |

Build and test against sandbox first. Sandbox credentials will not authenticate against production — a `401` here usually means mismatched keys, not a malformed request.
