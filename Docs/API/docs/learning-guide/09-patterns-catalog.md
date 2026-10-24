# Chapter 9 - Patterns Catalog

One place to look up every pattern used in the project. Each entry: **What**, **Why**, **Where**, **Reference**.

## 9.1 Architecture patterns

### Client-server with request/response
- **What:** the app asks, the API answers.
- **Why:** keeps sensitive decisions off the device.
- **Where:** the whole project.
- **Reference:** MDN Client-Server overview.

### Serverless / Function as a Service
- **What:** the platform runs your function on demand.
- **Why:** no servers to manage; scales automatically.
- **Where:** Cloudflare Workers; `export default app`.
- **Reference:** https://developers.cloudflare.com/workers/

### Layered architecture
- **What:** separate concerns into layers with one-way dependencies.
- **Why:** each layer can change without breaking the others.
- **Where:** contract (bottom) <- catalog and HTTP code <- tests.
- **Reference:** Chapter 1.5.

### Control plane vs data plane
- **What:** the API decides (control); Drive carries bulk bytes (data).
- **Why:** keeps the Worker light and cheap.
- **Where:** question-bank route.

### Walking skeleton / milestone delivery
- **What:** build the thinnest end-to-end slice first, thicken later.
- **Why:** reveals integration problems early.
- **Where:** M1 stubs.

## 9.2 API design patterns

### Resource-oriented, versioned URLs
- **What:** nouns in paths, `/v1` prefix.
- **Why:** predictable; allows breaking changes later.
- **Where:** `/api/v1/content/question-bank`.

### Error envelope with stable codes
- **What:** every failure is `{ error: { code, message, requestId } }`.
- **Why:** one client code path; messages can change, codes do not.
- **Where:** `errorResponse`, `apiErrorSchema`.

### Claim check (reference instead of payload)
- **What:** send a pointer; client fetches the data.
- **Why:** small responses.
- **Where:** question-bank route.
- **Reference:** https://learn.microsoft.com/azure/architecture/patterns/claim-check

### Health check endpoint
- **What:** cheap public "I'm alive" route.
- **Why:** monitoring.
- **Where:** `/api/health`.

### Explicit placeholder (stub)
- **What:** fixed fake responses that say they are fake (`"status":"stub"`).
- **Why:** prevents mistaking fake for real.
- **Where:** authorize and content/version.

### Correlation ID
- **What:** a unique ID on each request, in logs and in the response.
- **Why:** trace one request across systems.
- **Where:** `X-Request-Id`.

## 9.3 Code design patterns

### Middleware / chain of responsibility
- **What:** a series of functions each handling one concern, calling `next()`.
- **Why:** shared checks written once.
- **Where:** `app.use("*")`, `app.use("/api/v1/authorize")`.
- **Reference:** https://hono.dev/docs/guides/middleware

### Guard clauses (early return)
- **What:** check for the bad case and return immediately.
- **Why:** avoids deep nesting; the success path reads straight down.
- **Where:** all routes.

### Factory / helper function
- **What:** one function that builds a consistent object.
- **Why:** consistency, DRY.
- **Where:** `errorResponse`.

### Repository / lookup module
- **What:** a module that hides where data comes from.
- **Why:** you can change storage later.
- **Where:** `findQuestionBank`.

### Dependency / settings injection
- **What:** code receives what it needs (settings) instead of reaching for globals.
- **Why:** testability.
- **Where:** `context.env`, and `app.request(url, init, env)` in tests.

### Single source of truth / DRY
- **What:** one definition used everywhere.
- **Why:** no drift.
- **Where:** the contract; `contentVersionQuerySchema` reused by two routes.

### Typed unions and literals
- **What:** types listing the exact allowed values.
- **Why:** the compiler rejects invalid values.
- **Where:** `status: 400 | 401 | 404 | 500 | 503`; `z.enum`.

### Null-safe lookups
- **What:** a function that might find nothing returns `T | undefined`.
- **Why:** forces callers to handle "not found".
- **Where:** `findQuestionBank`, `header()?.trim()`.

### Fail safe / global exception handler
- **What:** unexpected errors produce a generic safe response.
- **Why:** no information leaks.
- **Where:** `app.onError`.

## 9.4 Validation and data patterns

### Parse, don't validate
- **What:** convert unknown input into a trusted typed value in one step.
- **Why:** after parsing, code can trust the type.
- **Where:** every `parse`/`safeParse`.

### Validate at the boundary
- **What:** check data where it enters the system.
- **Why:** everything inside can assume validity.
- **Where:** `zValidator`, query parsing.

### Allow-list validation
- **What:** permit only known-good values.
- **Why:** safer than blocking known-bad ones.
- **Where:** platform enum; Drive host list.

### Strict objects
- **What:** reject unknown fields.
- **Why:** prevents hidden dependencies and over-posting.
- **Where:** `.strict()` everywhere.

### Validate your own output
- **What:** parse responses with the schema before sending.
- **Why:** catches contract-breaking code changes immediately.
- **Where:** `healthResponseSchema.parse(...)`, `questionBankResponseSchema.safeParse(...)`.

### Integrity checksum
- **What:** compare a hash of received data with a trusted hash.
- **Why:** detect corruption/tampering.
- **Where:** `file.sha256`.

## 9.5 Operations patterns

### Configuration as code / config in the environment
- **What:** settings live in files or the environment, not in logic.
- **Why:** same code in every environment.
- **Where:** `wrangler.jsonc`, `AppBindings`.

### Pin your environment
- **What:** lock versions and runtime dates.
- **Why:** reproducible behavior.
- **Where:** `compatibility_date`, version ranges, lock file.

### Structured logging
- **What:** logs as JSON with named fields.
- **Why:** searchable.
- **Where:** global middleware.

### Principle of least information
- **What:** reveal only what is needed.
- **Why:** reduces attack surface (for example, do not distinguish unknown student from revoked student).
- **Where:** generic error messages.

## 9.6 Testing patterns

### Arrange-Act-Assert
Prepare input, call, check output. All tests.

### Negative testing
Prove bad input fails. Tests 2, 4, 5, 7, 8, 10, 11, 12.

### Test environment parity
Run tests in the same runtime as production. `vitest.config.ts` with the Cloudflare plugin.

### Executable documentation
Tests and `m1.http` show how the API is used.

## 9.7 Pattern quick-reference by problem

| Problem | Pattern to reach for |
| --- | --- |
| "Many routes need the same check" | Middleware |
| "Errors look different in each place" | Error envelope + factory helper |
| "Data from outside might be wrong" | Validate at the boundary (Zod) |
| "The response is too big" | Claim check |
| "App and server disagree on formats" | Contract-first, single source of truth |
| "A setting differs per environment" | Config in the environment |
| "I need to find one failing request" | Correlation ID + structured logs |
| "A value might be missing" | Null-safe lookup, guard clause |
| "I can't tell if the download is correct" | Integrity checksum |

Next: [Chapter 10 - Exercises, troubleshooting, glossary, references](10-exercises-glossary-references.md)
