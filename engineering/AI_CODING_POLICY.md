# AI-Assisted Development Policy

**Applies to:** All AR-IMMS contributors.

## 1. Developer Accountability

AI tools may assist with development, but **the developer is fully responsible for the submitted work**.

- Understand the task and relevant project documentation before coding.
- Review and understand all AI-generated changes.
- Be able to explain the implementation and important technical decisions.
- Own debugging, testing, review feedback, and integration issues related to the changes.

**AI-generated code that the author cannot explain or verify is not acceptable.**

## 2. AI Usage Boundaries

AI must not independently introduce or change:

- Business rules or expected behavior.
- Architecture, domain models, or module boundaries.
- Public APIs, contracts, or database schemas.
- Authentication, authorization, or security policies.
- Dependencies or infrastructure components.

Changes affecting these areas require approval from the responsible technical owner.

When requirements are unclear or conflict with existing documentation, **ask rather than assume**.

## 3. Scope & Code Quality

- Keep changes focused on the assigned issue.
- Do not introduce unrelated refactoring, unnecessary abstractions, or dependencies.
- Follow existing project conventions and architecture.
- Prefer small, cohesive, maintainable modules.
- **Source files should generally stay within 300–400 lines of code.** Larger files require a reasonable justification; generated files and other justified exceptions are excluded.
- PRs have no fixed LOC limit, but must remain focused and reviewable.

## 4. Verification

Before requesting PR review, developers must:

- Review their own diff and remove unintended changes.
- Run applicable lint, type checks, tests, and build.
- Verify the issue's acceptance criteria, including relevant failure cases.
- Provide test results or reproducible verification steps.
- Disclose known limitations and unresolved issues.

**Code that runs is not necessarily correct. Passing tests alone does not guarantee correctness.**

## 5. Debugging & Escalation

- Investigate failures using logs, reproduction steps, and hypotheses instead of repeatedly asking AI to modify code without understanding the cause.
- Raise blockers and unclear requirements early.
- When requesting help, provide the expected behavior, actual behavior, and what has been attempted.
- Do not silently expand scope or make unapproved technical decisions.

## 6. Pull Request & Review

- Each PR should address one coherent issue or independently reviewable change.
- The author must explain the changes, provide verification evidence, and resolve review feedback.
- Reviewers evaluate correctness, scope, maintainability, and risks.
- PRs missing verification, containing unexplained changes, or introducing unapproved decisions may be returned without detailed review.
- **Review approval does not transfer responsibility for the implementation to the reviewer.**

## 7. Security

- Never expose secrets, credentials, or sensitive data to unauthorized AI tools.
- Do not execute destructive AI-generated commands without understanding and authorization.

---

**Core principle:** AI is a development assistant, not the engineer responsible for the outcome. Every contributor owns the quality, correctness, and maintainability of their work.
