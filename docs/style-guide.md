# Documentation Style Guide

## Voice & Tone
- Active voice, present tense, imperative mood ("Send a request", not "A request should be sent").
- Neutral, professional, concise.

## Formatting
- Headers: H1 (page title), H2 (sections), H3 (sub-sections).
- Code blocks: language specified (`json`, `bash`, `yaml`).
- Tables: aligned, used for params/errors only.

## API Terminology
- Endpoint, route, path, parameter, payload, schema, status code.
- Use "request body" / "response body" (not "input/output data").
- IDs: `order_id` (snake_case in JSON), `orderId` (camelCase if specified by spec).

## Code Examples
- Always include: cURL + JSON response (success & error).
- Mask PII: use `{{API_KEY}}`, `user_***`.

## Localization
- Avoid idioms; use clear verbs.
- Measure units, dates: ISO 8601.
