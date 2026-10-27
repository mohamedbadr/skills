# Change Review reference

## Impact checklist

Work from the diff outward. For each item, either name the code you checked or mark it not applicable.

**Callers and contracts**
- Changed method, function, or constructor signatures: every call site, including reflection, DI registration, and string-based lookups.
- Renamed or removed public members, routes, events, message types, or CLI flags: every consumer, including other projects in the solution and front-end code calling the API.
- Changed return values, thrown exceptions, or null-handling: callers that depend on the old behavior.
- Interface or base-class changes: every implementation and override.

**Data**
- Entity, DTO, or schema changes: migrations exist and run in order, existing rows stay valid, defaults for new non-null columns.
- Serialization: JSON or XML property names, enum values stored as numbers or strings, versioned payloads, caches holding old shapes.
- Queries: filters that now include or exclude different rows, N+1 queries, missing indexes on new filters.

**Configuration and wiring**
- New config keys present in every environment file and documented; removed keys no longer read.
- DI or service registration for new types; lifetimes that match their dependencies.
- Feature flags, permissions, and role checks applied on every new path.
- Infrastructure (pipelines, Terraform, Docker, scripts) updated if the build output or runtime needs changed.

**Behavior**
- Shared helpers or base components changed for one feature but used by others.
- Error paths: failures are handled, logged, and do not leave partial state; transactions still cover the same work.
- Concurrency and async: missing awaits, shared mutable state, race conditions introduced by new background work.
- Security: user input validated, no secrets in code or logs, authorization not loosened.
- Localization, time zones, and culture-sensitive formatting if user-facing text or dates changed.

**Tests and build**
- Run the build. Report new errors and new warnings compared to `HEAD`.
- Run the existing tests that cover the changed files. Name tests that were changed or deleted, and say whether that hides a regression.
- New behavior with no test: name the behavior.

## Standards baseline

Use only when the repo documents no conventions, or for things its docs do not cover. Each item is a judgement call with a suggested fix, never an automatic violation.

- Does not match the style, naming, or structure of neighbouring code.
- Duplicated logic that already exists as a helper in the repo.
- Long function doing several things; long parameter list.
- Feature envy: logic that belongs in the type whose data it uses.
- Primitive obsession: magic strings or numbers that should be a type, enum, or constant.
- Shotgun surgery: one concept spread across many unrelated edits.
- Dead code, commented-out code, or leftover debug output.
- Comments that restate the code, or missing comments on non-obvious decisions.
- Swallowed exceptions or overly broad catches.
- Speculative generality: abstractions with a single use and no plan for more.

## Report template

```
# Review: <change name>

## Meta
- **Reviewed**: <YYYY-MM-DD HH:mm, local time>
- **Scope**: <diff command used>, <N> files, +<added> / -<removed>
- **Spec**: <path, ticket ID, or "none found">

## Summary
| Axis | Status | Worst finding |
|------|--------|---------------|
| Plan | <Aligned / Gaps found / Skipped: no spec> | <one line or "none"> |
| Impact | <No breakage found / Breakage found> | ... |
| Standards | <Clean / Issues found> | ... |

## Plan
| Step | Status | Evidence |
|------|--------|----------|
| Step 1: <title> | Done / Partial / Missing / Deviated | `file:line` or note |

### Findings
- **[Major] <title>** — `path/file.cs:42`
  - Evidence: <quoted spec line and what the code does instead>
  - Fix: <concrete change>

## Impact
- **[Blocker] <title>** — `path/file.cs:42` breaks `path/caller.cs:88`
  - Evidence: <what the caller expects vs what it now gets>
  - Fix: <concrete change>

Build: <result>. Tests run: <names and result>.

## Standards
- **[Minor] <title>** — `path/file.cs:42`
  - Rule: <doc and rule, or baseline item>
  - Fix: <concrete change>

## Not checked
- <what could not be verified, and why>

## Still needs manual testing
- <verification checklist items from the plan that only a human can run>
```

Mark unproven findings with **(Plausible)** after the severity. Omit an axis's Findings list when it has none, but keep the axis heading with its status.
