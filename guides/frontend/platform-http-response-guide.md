# Core HTTP response guide for Frontend Agents

This guide explains how frontend clients consume the platform's common HTTP responses. The normative reusable schema is [`core-api-baseline.openapi.yaml`](../../contracts/openapi/core-api-baseline.openapi.yaml); endpoint contracts define their own data shapes and any raw-response exceptions.

## Standard success responses

Ordinary controller results are wrapped by the HTTP response interceptor:

```json
{
  "data": { "id": "asset_123" },
  "meta": {
    "request_id": "req_01JEXAMPLE",
    "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736"
  }
}
```

- `data` contains the endpoint's result. A single result may be an object or `null`; list endpoints return an array.
- `meta.request_id` is always attached by the standard response path. `meta.trace_id` appears when an active trace is available.
- The response also carries the correlation ID in the `X-Request-Id` header.
- A paginated list adds `pagination`. Cursor and page pagination use different shapes; follow the endpoint contract and treat cursors as opaque.
- A `204 No Content` response has no JSON body. Do not attempt to unwrap `data` from it.

## Validation errors

Errors use `Content-Type: application/problem+json` and the Core Problem Details shape. For validation failures, `errors` gives field-level details:

```json
{
  "type": "about:blank",
  "title": "Request validation failed",
  "status": 400,
  "detail": "One or more request fields are invalid.",
  "instance": "/api/v1/assets",
  "code": "VALIDATION_ERROR",
  "request_id": "req_01JEXAMPLE",
  "errors": [
    {
      "field": ["items", 0, "quantity"],
      "code": "OUT_OF_RANGE",
      "message": "Quantity must be greater than zero."
    }
  ]
}
```

- `field` is an array of path segments: object keys are strings and array indexes are integers. It is not a JSON Pointer string.
- `field.code` is one of `INVALID_TYPE`, `INVALID_FORMAT`, `OUT_OF_RANGE`, `UNKNOWN_FIELD`, or `INVALID_VALUE`.
- `message` is suitable for field-level feedback, but keep behavior keyed to status and stable codes rather than message text.
- The top-level `code` identifies the API error category. The HTTP status and body `status` agree.

## Other errors and client behavior

- Use the top-level `status` and `code` to choose behavior. Use `detail` as a message only when appropriate for the user; unexpected server errors receive a generic detail outside development.
- Keep `request_id` for support diagnostics. Never send credentials, passwords, or sensitive form values with diagnostics.
- A `401` means authentication is missing, invalid, or expired; a `403` means the request is not allowed. Endpoint contracts may specify an auth refresh flow, but do not retry a `403` as if it were expiry.
- Validation schemas are strict on many endpoints. An unknown request field can produce `UNKNOWN_FIELD`; send only fields in the contract.
- Honor endpoint-specific headers such as `Retry-After` and `Cache-Control: no-store`.

## Exceptions to the standard envelope

- `@RawResponse()` marks routes whose controller owns the response format. Health routes are examples; readiness `503` keeps Terminus's JSON shape instead of Problem Details.
- `204` routes remain bodyless.
- Do not infer the response shape from another endpoint. Check that endpoint's OpenAPI contract for raw payloads, headers, and status behavior.

## Pagination reminder

- `cursor` responses return `{ "type": "cursor", "next_cursor": "..." }`; `null` means there is no next page. Pass cursors back unchanged.
- `page` responses include `page`, `page_size`, `has_next`, and `has_prev`; `total_items` and `total_pages` may be omitted or `null`.
- The platform query schemas default `page_size` to 20, cap it at 100, and reject unknown query keys. Endpoints may narrow supported query parameters.
