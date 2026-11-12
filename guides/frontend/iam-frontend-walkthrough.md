# IAM module walkthrough for Frontend Agents

This note describes the implemented IAM module for Dashboard, AR, and Mobile clients. The OpenAPI contract beside this file is the endpoint and payload reference.

## What IAM owns

- Human account sign-in, password changes, access sessions, and logout.
- Three fixed roles: `IT_ADMIN`, `OPERATOR`, and `TECHNICIAN`.
- A fixed permission catalog. A user's effective permissions combine the role baseline with approved additional permissions.
- User administration and coarse user presence.
- Server-side authentication and permission checks. A frontend permission list is only for showing or hiding controls.

IAM does not provide public sign-up, custom roles, or Agent authentication. It does not decide resource-specific rules such as whether a technician is assigned to a particular maintenance activity.

## Main frontend journeys

1. **Sign in.** Dashboard uses `/auth/browser/login`; native clients use `/auth/login`. Both accept `{ identifier, password }`. Successful login returns an access token and `must_change_password`. A forced first-use password change should be the only ordinary screen shown until the password is changed.
2. **Restore a session.** Dashboard posts `/auth/browser/refresh`; the browser sends its HttpOnly cookie and the client supplies the in-memory `X-CSRF-Token` plus `Origin`. Native clients post `/auth/refresh` with the stored `refresh_token`. Refresh tokens rotate, so replace the saved token after every successful native refresh.
3. **Load the current user.** Call `GET /auth/me` with the access token. It returns identity, role, effective permissions, password-change state, and presence preference/status. Permissions help shape the UI; the API remains the authority.
4. **Keep a session active.** Send `POST /auth/me/presence/heartbeat` while the user is active. Set a preference with `PUT /auth/me/presence-preference`. Presence is an availability signal, not proof of authorization.
5. **Manage users.** Authorized administrators list users with opaque cursor pagination, view a user, edit profile fields, replace role or additional permissions, change status, reset a password, or revoke sessions. Create/reset responses contain a one-time temporary password; show it once and do not persist or log it.
6. **Load access options.** `GET /iam/roles` and `GET /iam/permissions` return the fixed catalogs. Use the catalog to populate administration forms rather than maintaining a second hard-coded list.

## Boundaries to remember

- Protected calls use `Authorization: Bearer <access_token>` and return the standard `{ data, meta }` envelope; user lists also include cursor pagination.
- Browser refresh credentials are cookies and never appear in JSON. Browser refresh/logout require both CSRF and Origin checks.
- Access tokens are short-lived; refresh sessions have an absolute expiry. Handle `401` by attempting the appropriate refresh once, then clear local session state and return to sign-in if it fails.
- A `403` means the caller is authenticated but cannot perform that action, or must change a password first. Do not retry it as a token-expiry case.
- Role changes clear additional permissions. Disabling an account or resetting a password can invalidate active sessions. The last active IT Admin is protected by the server.
- Keep passwords, access/refresh tokens, CSRF tokens, and temporary credentials out of logs, analytics, URLs, persistent browser storage, and error-reporting payloads.

For endpoint details, status codes, schemas, and security requirements see [`iam-frontend.openapi.yaml`](../../contracts/openapi/iam-frontend.openapi.yaml). Use the shared [frontend implementation guide](frontend-implementation-guide.md) for general design and coding principles. The backend source contract is `api/v1/iam.yaml`; verify both against the controllers when changing behavior.
