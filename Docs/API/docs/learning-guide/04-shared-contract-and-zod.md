# Chapter 4 - The Shared Contract and Zod

File: [shared/api-contract/index.ts](../../shared/api-contract/index.ts)

## 4.1 What a contract is, and why this project has one

- **What:** A *contract* is the written, exact agreement of what data crosses the boundary between the app and the API: field names, types, allowed values, and error formats.
- **Why:** two independent programs can only cooperate if they agree. If the API renames `sizeBytes` to `size`, the app silently breaks. A shared file means both sides are changed together and a mismatch is visible.
- **Pattern:** *contract-first design*, *single source of truth*, and *DRY* (Don't Repeat Yourself).
- **Project rule:** "Keep request, response, and error contracts centralized in `shared/api-contract/`" (`.github/copilot-instructions.md`).

## 4.2 The problem TypeScript alone cannot solve

TypeScript checks types only while you compile. Data arriving over the internet is just text. Anyone can send anything:

```json
{ "email": 12345, "deviceId": "", "hacker": true }
```

TypeScript cannot stop that at runtime. You need a **runtime validator**. This project uses Zod.

## 4.3 Zod in 10 minutes

- **What:** Zod lets you describe a shape once, then (a) check real data against it at runtime and (b) derive the TypeScript type from it automatically.
- **Why:** one description, two benefits, zero chance of the type and the check disagreeing.
- **Pattern:** *schema-driven validation*; *parse, don't validate* (turn unknown data into trusted typed data in one step, and reject if it does not fit).
- **Reference:** https://zod.dev

### The building blocks used in this file

| Zod call | Meaning | Example in this project |
| --- | --- | --- |
| `z.string()` | Must be text. | `message: z.string()` |
| `z.number()` | Must be a number. | `sizeBytes` |
| `.int()` / `.positive()` | Whole number / greater than zero. | `sizeBytes` |
| `.min(1)` / `.max(64)` | Length (or size) limits. | `examCode: z.string().min(1).max(64)` |
| `.trim()` | Remove surrounding spaces, then validate. | `email` |
| `.email()` | Must look like an email. | `email` |
| `.url()` | Must look like a web address. | `file.url` |
| `.regex(/.../)` | Must match a pattern. | `sha256` |
| `.datetime({ offset: true })` | ISO 8601 date-time with timezone. | health `time` |
| `z.enum([...])` | Exactly one of the listed strings. | `platformSchema`, `contentModeSchema`, `apiErrorCodeSchema` |
| `z.literal(x)` | Exactly this value. | `status: z.literal("stub")`, `apiVersion: z.literal(1)` |
| `z.tuple([])` | An array of exact length and types; empty here. | `entitlements: z.tuple([])` (must be empty) |
| `z.object({...})` | An object with named fields. | Everywhere |
| `.strict()` | Reject unknown fields. | Every object |
| `.refine(fn, msg)` | Add a custom rule. | Drive URL host check |
| `.safeParse(data)` | Check; return `{ success, data \| error }` without throwing. | Query and response validation |
| `.parse(data)` | Check; **throw** on failure. | Fixed stub responses |
| `z.infer<typeof schema>` | Get the TypeScript type from a schema. | The `export type ...` lines |

### `parse` versus `safeParse`

- `parse` throws an exception if data is wrong. Use it when failure means a *programming bug* (for example, a stub response we wrote ourselves is malformed). The global error handler converts it to a safe 500.
- `safeParse` returns a result object. Use it when failure is *expected* and you want to answer politely (bad query from a client, or a missing setting).

## 4.4 Walking through the contract file

### Platform and mode

```ts
export const platformSchema = z.enum(["android", "ios", "windows"]);
export const contentModeSchema = z.enum(["exam", "prep"]);
```

Only these values are allowed. `exam` and `prep` are the two business modes named in the requirements.

**Why enums:** a fixed vocabulary stops typos such as `"Android"` or `"preparation"` from ever reaching business logic.

### Error codes

```ts
export const apiErrorCodeSchema = z.enum([
  "INVALID_REQUEST", "LEASE_INVALID", "LEASE_EXPIRED", "NOT_AUTHORIZED",
  "ACCESS_EXPIRED", "NOT_ENTITLED", "PACKAGE_NOT_FOUND", "DEVICE_LIMIT_REACHED",
  "DEVICE_KEY_MISMATCH", "VERSION_NOT_ASSIGNED", "TICKET_ALREADY_USED",
  "TICKET_INVALID", "RATE_LIMITED", "UPSTREAM_UNAVAILABLE", "INTERNAL_ERROR",
]);
```

**What:** the complete list of machine-readable error names. Only a few are used in M1; the rest are agreed up front.

**Why:** the app's code can say "if code is `LEASE_EXPIRED`, ask the user to sign in again" without parsing English sentences. The human message can change freely; the code cannot.

**Pattern:** *error code vocabulary* (stable identifiers separate from display text).

Codes introduced or used by the question-bank feature:

| Code | HTTP status | When |
| --- | --- | --- |
| `LEASE_INVALID` | 401 | No usable bearer header. |
| `INVALID_REQUEST` | 400 | Bad or missing query/headers. |
| `PACKAGE_NOT_FOUND` | 404 | No question bank for that exam and mode. |
| `UPSTREAM_UNAVAILABLE` | 503 | Drive URL is not configured correctly. |

### The error envelope

```ts
export const apiErrorSchema = z.object({
  error: z.object({
    code: apiErrorCodeSchema,
    message: z.string(),
    requestId: z.string().min(1),
  }).strict(),
}).strict();
```

Every failure looks like:

```json
{ "error": { "code": "PACKAGE_NOT_FOUND", "message": "No question bank is available.", "requestId": "..." } }
```

**Pattern:** *error envelope* - one predictable wrapper for all failures so the client has one place to handle them.

### Authorization request and stub response

```ts
export const authorizeRequestSchema = z.object({
  email: z.string().trim().email().max(320),
  deviceId: z.string().min(1).max(128),
  devicePublicKey: z.string().min(1).max(2048),
}).strict();
```

- Length limits (`max`) protect the server from oversized input. 320 is the maximum length of a valid email address.
- `.strict()` means a client cannot sneak in `examCode` or similar fields. The test "rejects unknown authorization fields" proves it.

### Content-version query and response

```ts
export const contentVersionQuerySchema = z.object({
  examCode: z.string().min(1).max(64),
  mode: contentModeSchema,
}).strict();
```

Used by both `content/version` and `content/question-bank` because both are asked "which exam, which mode?".

**Pattern:** *reuse a schema* instead of copying it. If the rules for `examCode` change, they change once.

### Health response

`status: "ok"`, `apiVersion: "1"` (string), and an ISO `time`. Note: health uses the string `"1"` while the question-bank file uses the number `1`. They are different contracts; do not mix them up.

### The question-bank pointer schema (new feature)

```ts
const questionBankHosts = ["drive.google.com", "drive.usercontent.google.com", "docs.google.com"];

export const questionBankResponseSchema = z.object({
  apiVersion: z.literal(1),
  exam: z.object({
    code: z.string().min(1),
    title: z.string().min(1),
    contentVersion: z.string().min(1),
    mode: contentModeSchema,
  }).strict(),
  file: z.object({
    url: z.string().url().refine((value) => {
      const parsed = new URL(value);
      return parsed.protocol === "https:" && questionBankHosts.includes(parsed.hostname);
    }, "Download URL must be an https Google Drive URL."),
    sizeBytes: z.number().int().positive(),
    sha256: z.string().regex(/^[0-9a-f]{64}$/),
  }).strict(),
}).strict();
```

Field by field:

| Field | What | Why |
| --- | --- | --- |
| `apiVersion` | Format version number. | Lets the app know how to interpret the data. |
| `exam.code` | e.g. `AZ-305`. | Identify the exam. |
| `exam.title` | Display title. | Show in the UI without opening the file. |
| `exam.contentVersion` | e.g. `1.1.0`. | The app compares it with its stored copy and downloads only if it changed. |
| `exam.mode` | `exam` or `prep`. | Which mode this content is for. |
| `file.url` | Where to download. | Keeps the large file off the Worker. |
| `file.sizeBytes` | Expected size. | Progress bar and sanity check. |
| `file.sha256` | Fingerprint of the file. | Integrity check (see Chapter 6). |

#### Why the URL has a custom rule

A configuration mistake (or a malicious change) could make the API hand the app a link to any website. The `refine` rule enforces two things:

1. **`https:`** - encrypted transport only.
2. **An allow-listed host** - only Google Drive/Docs addresses.

**Pattern:** *allow-list validation* (permit only known-good values, instead of trying to block known-bad ones). It also reduces risks like open redirects and server-side request forgery style abuse.

#### Why `sha256` is a regex

SHA-256 is always 64 hexadecimal characters. The pattern `^[0-9a-f]{64}$` means: from the start (`^`) to the end (`$`), exactly 64 characters, each a digit or lowercase letter a-f. **Lowercase only** - PowerShell's `Get-FileHash` prints uppercase, so you must lowercase it before putting it in the catalog.

## 4.5 Inferred types

```ts
export type ApiErrorCode = z.infer<typeof apiErrorCodeSchema>;
export type QuestionBankResponse = z.infer<typeof questionBankResponseSchema>;
```

`z.infer` produces the type from the schema. `ApiErrorCode` becomes the union `"INVALID_REQUEST" | "LEASE_INVALID" | ...`. Add a new error code to the enum and the type updates automatically.

**Why two exports with similar names?** `questionBankResponseSchema` is a *value* (a runtime checker). `QuestionBankResponse` is a *type* (compile-time only). Same data description, different purposes. The `Schema` suffix tells them apart.

## 4.6 Strictness philosophy

Nearly every object ends with `.strict()`. This means:

- **Requests:** unexpected fields are rejected, so clients cannot depend on accidental behavior.
- **Responses:** the server cannot accidentally leak an extra field such as an internal ID.
- **Change control:** adding a field forces a conscious contract update.

**Trade-off:** strictness makes evolution deliberate. Adding an optional field later requires changing the schema on both sides; that is intended.

## 4.7 Sharing the contract with the app

The app lives in another project. Options to share this file, from simplest to most robust:

1. Copy it (risk: drift).
2. Publish it as a small internal package that both projects install.
3. Generate it from a neutral format (such as OpenAPI/JSON Schema) for apps written in other languages.

If the app is not TypeScript (for example Kotlin or Swift), it needs equivalent models. The agreement is the *JSON*, not the TypeScript file. The README says to reconcile the contract with the app before integration.

## 4.8 Self-check questions

1. Why can't TypeScript types validate an incoming request?
2. When would you use `parse` instead of `safeParse`?
3. What does `.strict()` prevent?
4. Why is `sha256` required to be lowercase hexadecimal?
5. Which two properties does the URL rule enforce?

## 4.9 Exercise

Add an optional field `releaseNotes` (a string up to 500 characters) to `exam`. Update the schema, the catalog type, one test, and confirm `npm run typecheck` and `npm test` pass. Think about what must change in the app too.

Next: [Chapter 5 - The Worker, line by line](05-worker-walkthrough.md)
