# IAM frontend implementation guide

This guide **inherits** the shared [Frontend implementation guide](frontend-implementation-guide.md). Apply its design principles, coding rules, adaptable structure, security guidance, and verification workflow to every IAM change. This document adds IAM-specific mappings and constraints; it does not replace or repeat the shared baseline.

## How the shared guide maps to IAM

| Shared guide principle | IAM application |
| --- | --- |
| Organize by feature | Keep sign-in/session UI and user administration in the IAM feature area; use the repository's established structure. |
| Keep I/O behind an API adapter | Put IAM routes, token headers/cookies, envelope handling, and error normalization in an IAM API adapter. Screens should not construct URLs. |
| Use typed contracts | Use [`iam-frontend.openapi.yaml`](../../contracts/openapi/iam-frontend.openapi.yaml) for endpoints, payloads, errors, and enums. Keep generated types isolated from handwritten state and policy. |
| Model all UI states | Handle restoring, signed out, authenticated, password-change-required, loading, empty directory, validation, forbidden, conflict, throttled, and failed-refresh states. |
| Keep server rules authoritative | Permission checks can hide or disable controls for clarity; every action must still handle server `403`. Do not duplicate role-to-permission rules in the UI. |
| Protect sensitive data | Keep access and CSRF tokens in memory where possible; keep native refresh tokens in platform secure storage; never persist or log temporary passwords. |
| Verify focused behavior | Test the client-specific auth flow, password gate, refresh failure, permission-aware UI, cursor paging, and one-time credential display. |

## Session and authentication flows

### Browser Dashboard

- Use `/auth/browser/login`, `/auth/browser/refresh`, and `/auth/browser/logout`.
- Let the browser manage the HttpOnly refresh cookie. Do not read or copy it into JavaScript.
- Keep the access token and current CSRF token in memory. Send `X-CSRF-Token` and the exact `Origin` on refresh and logout; replace the CSRF token after each successful refresh.
- Configure credentialed requests for the API origin. Prefer same-origin deployment through a reverse proxy; validate cookie and CORS behavior on the deployed origin.
- Restore the session by refreshing, then load `/auth/me`. Clear client session state and show sign-in when refresh fails.

### Native clients

- Use `/auth/login` and `/auth/refresh` with JSON tokens.
- Store the rotating refresh token only in platform-protected secure storage; replace it after each successful refresh. Keep access tokens in memory where practical.
- After refresh failure, clear the saved refresh token and require sign-in. Coordinate concurrent refresh requests so only one rotation is in flight.

### Forced password change

- Treat `must_change_password: true` as a distinct session state and route to password change before ordinary application screens.
- Call `POST /auth/change-password`; send `current_password` for a normal change, while a first-use/reset change may omit it. The new password must be at least 12 characters.
- Clear password fields after submission. Do not include credentials in logs, analytics, screenshots, or error reports.

## Principal, permissions, and presence

- Load `GET /auth/me` after sign-in and successful refresh. Replace cached identity and permissions with the latest response.
- Use `effective_permissions` for capability-based UI, centralized behind a helper such as `can('user:manage')`. Do not derive permissions from role names.
- Use `/iam/roles` and `/iam/permissions` to populate administrative choices; do not maintain a duplicate catalog in frontend code.
- A frontend permission check is only a usability aid. The backend resolves current permissions and owning modules enforce resource-level policy.
- Presence from `/auth/me` is a coarse availability signal. It does not establish authorization or eligibility for a particular assignment.
- Send `POST /auth/me/presence/heartbeat` while the app considers the session active. Update preference with `PUT /auth/me/presence-preference`.

## User administration

- List users with `/users?page_size=20`; pass `next_cursor` back unchanged as `cursor`, and stop when it is `null`. The current contract does not define search or sort parameters.
- Use `PATCH /users/{user_id}` for username, email, and display name. Use the dedicated `PUT` endpoints for role, additional permissions, and status.
- Explain that changing a role clears additional permissions. Show the server-returned user after a successful mutation.
- Create and reset-password responses contain a one-time temporary password. Display it once, clear it when the view closes, and keep it out of global state and browser storage.
- On `409`, reload current server state and ask the user to review before retrying. The server protects the last active IT Admin and rejects certain self-administration changes.

## HTTP behavior

- Protected requests use `Authorization: Bearer <access_token>` and standard `{ data, meta }` responses; user-list responses also include cursor pagination.
- Handle `401` by attempting the appropriate refresh once, then clear session state if refresh fails. Do not retry `403` as an expiry case.
- Normalize `400`, `401`, `403`, `404`, `409`, and `429` in the API adapter. Use status or stable error codes for behavior; do not parse human-readable error text.
- Honor `Retry-After` on `429`. Do not automatically replay a non-idempotent mutation after an uncertain network failure.
- Exclude credential-bearing responses from caches and service-worker storage. Preserve request IDs for support diagnostics without including secrets.

## IAM delivery checklist

- [ ] Browser or native transport matches the client type.
- [ ] Refresh rotation, session restoration, logout, and failure cleanup work.
- [ ] Password-change-required state blocks ordinary routes.
- [ ] Permission-aware controls still handle server `403` responses.
- [ ] User administration follows the OpenAPI contract and opaque cursor rules.
- [ ] Temporary credentials are shown once and never persisted or logged.
- [ ] The contract, generated types, and focused IAM behavior checks stay in sync.
