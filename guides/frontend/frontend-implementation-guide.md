# Frontend implementation guide

Use this as a shared baseline for frontend work. Adapt the examples to the product, framework, repository conventions, and feature size; the folder structure is a starting point, not a required layout.

## Product and design principles

- **Start with the user task.** Make the primary action and the information needed to complete it easy to find.
- **Use a clear hierarchy.** Group related information, use concise labels, and make important states and actions visually distinct.
- **Stay consistent.** Reuse established components, spacing, colors, typography, and interaction patterns before adding new ones.
- **Design every state.** Cover loading, empty, success, validation, permission-denied, offline, and unexpected-error cases—not only the ideal path.
- **Make accessibility part of the design.** Use semantic controls, keyboard support, visible focus, sufficient contrast, meaningful labels, and status text that assistive technology can announce.
- **Support real screens.** Check narrow and wide layouts, long text, zoom, and touch targets. Avoid relying on color or hover alone to convey meaning.
- **Keep user control.** Explain destructive or irreversible actions, confirm when consequences are significant, and provide recovery where practical.

## Suggested project structure

Organize around the app's actual features. A small app may need fewer folders; a large app may split features into subdomains.

```text
src/
  app/                 # App shell, routes, global providers
  features/            # User-facing product areas
    <feature>/
      components/      # Feature-specific UI
      pages/            # Route-level composition
      api/              # Feature API adapter and DTO mapping
      state/            # Feature state, only when needed
      validation/       # Input schemas and form rules
      __tests__/        # Focused feature tests
  shared/
    ui/                 # Reusable, product-neutral components
    api/                # HTTP client, common errors, generated types
    hooks/              # Reusable behavior, not feature workflows
    utils/              # Small, well-named general helpers
```

Keep feature behavior with its feature. Move code into `shared` only after it is genuinely reusable and has a clear owner. Follow existing repo conventions when they differ from this example.

## Coding rules

- Use the repository's language, framework, formatting, naming, and import conventions. Keep compiler strictness enabled.
- **Do not use `any`.** Use a concrete type where known; use `unknown` at untrusted boundaries and validate or narrow it before use.
- Give API requests, responses, state, and component props explicit types. Derive types from the API contract or schemas where practical; avoid maintaining duplicate hand-written copies.
- Keep components focused on rendering and user interaction. Put reusable business rules in named functions or feature services, and keep I/O behind an API adapter.
- Prefer simple composition over deeply nested conditionals, oversized components, premature abstraction, or hidden side effects.
- Keep state as local as possible. Promote it to feature or app state only when multiple routes or components need the same source of truth.
- Handle API results explicitly. Model loading, success, empty, and error states; validate forms on the client for feedback and let the server remain authoritative.
- Use stable IDs as list keys. Avoid array indexes when rows can be inserted, removed, reordered, or refreshed.
- Keep generated code separate from handwritten code. Pin generators and options; never hand-edit generated output.
- Remove dead code, debug output, unused dependencies, and commented-out alternatives before delivery.

## API, errors, and security

- Treat OpenAPI or the owning service contract as the source for paths, payloads, status codes, and enums. Do not guess or silently reshape API behavior in the UI.
- Keep HTTP details in a shared client or feature adapter. Normalize errors once, then show messages appropriate to the user's task.
- Use status codes and stable error codes for behavior; do not parse human-readable error text to decide what the app should do.
- The server owns authentication, authorization, validation, and resource rules. UI permission checks improve usability but do not grant access.
- Do not put secrets in URLs, logs, analytics, error reports, screenshots, or persistent storage. Use platform-approved secure storage when credentials must persist.
- Avoid unsafe HTML rendering. Validate external input at boundaries and encode output with the framework's normal escaping behavior.
- Cache only data that is safe and useful to cache. Define invalidation after mutations and exclude credentials and sensitive responses.

## Verification and delivery

- Add focused tests for meaningful behavior: rendering states, validation, user actions, permission-aware UI, and API error paths.
- Run the repository's relevant formatter, linter, type checker, tests, and production build. Report checks that were not available.
- Check keyboard navigation, focus order, responsive layout, and accessible names for changed screens.
- Review the diff for unrelated edits, accidental secrets, stale generated files, and contract changes without corresponding frontend updates.
- Keep changes small enough to review. Describe user-visible behavior, notable design decisions, and verification in the delivery summary.

## Working pattern

1. Read repository guidance, existing screens, design system, and the owning API contract.
2. Identify the user journey, permissions, data states, and responsive/accessibility needs.
3. Reuse existing patterns; agree on unresolved product behavior before inventing it.
4. Implement one coherent feature slice with typed boundaries and explicit states.
5. Verify the slice, review the diff, and update shared contracts or docs when behavior changed.
