# Chapter 7 - Testing

Files:

- [test/api.test.ts](../../test/api.test.ts)
- [backend/api-tests/m1.http](../../backend/api-tests/m1.http)
- [vitest.config.ts](../../vitest.config.ts)

## 7.1 What testing is and why it matters here

- **What:** a test is a small program that calls your code with known input and checks the output.
- **Why:** an API is a promise to other programs. Tests keep the promise true when you change code. They also act as a precise description of behavior: reading the test names tells you what the API does.
- **Pattern:** *automated regression testing*, *arrange-act-assert* (also called given-when-then).
- **Reference:** https://vitest.dev/guide/

## 7.2 Kinds of tests (and which this project uses)

| Kind | What it checks | In this project |
| --- | --- | --- |
| Unit | One function in isolation. | Not separately; the catalog is covered through routes. |
| **Integration / API** | A whole route, with middleware and validation, in memory. | `test/api.test.ts` (all 12 tests). |
| End-to-end | Real app talking to a real deployed server. | Planned in later milestones. |
| Manual | A human sends requests. | `m1.http` and the app smoke test. |

## 7.3 Testing without a server: `app.request`

Hono lets you call the app as a function:

```ts
const response = await app.request("/api/health");
```

No port is opened, no network used, so tests are fast and reliable. The third argument supplies *settings* (the Worker's `env`):

```ts
await app.request(url, { headers }, { QUESTION_BANK_URL: driveUrl });
```

This is *dependency injection* for tests: every test decides its own configuration, so no test depends on the real `wrangler.jsonc` value.

## 7.4 Anatomy of a test

```ts
describe("M1 API stubs", () => {            // group
  it("serves health without app headers or a database", async () => {   // one behavior
    const response = await app.request("/api/health");                  // act
    const body: unknown = await response.json();                        // read result
    expect(response.status).toBe(200);                                  // assert
    expect(body).toMatchObject({ status: "ok", apiVersion: "1" });      // assert
  });
});
```

| Function | Meaning |
| --- | --- |
| `describe(name, fn)` | Group related tests. |
| `it(name, fn)` | One test; the name should read like a sentence of expected behavior. |
| `expect(value)` | Start an assertion. |
| `.toBe(x)` | Strict equality (for numbers, strings). |
| `.toEqual(obj)` | Deep equality (every field must match exactly). |
| `.toMatchObject(obj)` | Contains at least these fields (extras allowed). |
| `.resolves` | Wait for a promise, then assert on its result. |

`const body: unknown` is good TypeScript hygiene: JSON from `response.json()` is untrusted until checked.

## 7.5 The 12 tests, explained

### Group: "M1 API stubs" (8 tests)

| # | Test | Proves |
| --- | --- | --- |
| 1 | serves health without app headers or a database | Health is public and returns `ok`, version `1`, an ISO time, and an `X-Request-Id`. |
| 2 | rejects missing required app headers | The global middleware blocks calls missing `X-App-Version`/`X-Platform` with 400. |
| 3 | validates authorization input and returns an explicit M1 stub | Happy path returns exactly `{ status: "stub", entitlements: [] }`. |
| 4 | rejects unknown authorization fields | `.strict()` works: an extra `examCode` field gives 400. |
| 5 | requires JSON content type for authorization | A `text/plain` body gives 400. |
| 6 | requires a bearer header and returns only an M1 version stub | 401 without bearer; 200 stub echo with it. |
| 7 | rejects invalid content version queries | `mode=unknown` gives 400. |
| 8 | rejects unknown content version query parameters | An unexpected query key gives 400 with `INVALID_REQUEST`. |

### Group: "question-bank download pointer" (4 tests)

| # | Test | Proves |
| --- | --- | --- |
| 9 | returns metadata and the Drive URL instead of the questions | Full exact JSON shape with catalog values and the injected URL. |
| 10 | requires a bearer header | 401 when the `Authorization` header is absent. |
| 11 | returns 404 for an exam without a question bank | `PACKAGE_NOT_FOUND` for an unknown exam. |
| 12 | returns 503 when the URL is missing or not a Google Drive https URL | Both an absent setting and a non-Google host are refused. |

Notice test 12 covers a *security* behavior (the allow-list) and a *configuration* behavior (missing URL) together.

## 7.6 Good test habits shown in this file

- **One reason to fail per test** (mostly): the name tells you exactly what broke.
- **Test the unhappy paths:** missing headers, wrong types, unknown routes. Most real bugs live there.
- **Do not copy production secrets** into tests; the bearer value `m1-placeholder` is a placeholder, not a credential.
- **Shared constants** (`appHeaders`, `driveUrl`) avoid repeating long literals.

## 7.7 Running tests

```powershell
npm test               # run once
npm run test:watch     # re-run on every save
npm run typecheck      # types only, no tests
```

Sample passing output:

```text
 Test Files  1 passed (1)
      Tests  12 passed (12)
```

If a test fails, Vitest shows the *expected* value, the *received* value, and the line number. Read it as: "I expected A but got B in this assertion."

## 7.8 The REST Client file (`m1.http`)

- **What:** requests written as plain text, separated by `###`, with variables like `@baseUrl`.
- **Why:** quick manual checks against `npm run dev`, and a readable list of how to call each route.
- **How:** install the VS Code extension "REST Client" (by Huachao Mao), start `npm run dev`, then click "Send Request" above a request.

```http
@baseUrl = http://127.0.0.1:8787
@appVersion = 0.1.0
@platform = android

### Question-bank download pointer
GET {{baseUrl}}/api/v1/content/question-bank?examCode=AZ-305&mode=prep
Authorization: Bearer m1-placeholder
X-App-Version: {{appVersion}}
X-Platform: {{platform}}
```

Note: with the placeholder `REPLACE_WITH_FILE_ID` URL in `wrangler.jsonc`, this request currently returns a URL containing that text (it is a syntactically valid Google URL, so validation passes). Replace it with a real ID to test the whole flow.

PowerShell equivalent:

```powershell
$h = @{ 'X-App-Version'='0.1.0'; 'X-Platform'='android'; Authorization='Bearer m1-placeholder' }
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/v1/content/question-bank?examCode=AZ-305&mode=prep' -Headers $h
```

## 7.9 Writing your own test: a recipe

Goal: confirm the `mode` is required on the question-bank route.

1. Open `test/api.test.ts`; find the `describe("question-bank download pointer", ...)` block.
2. Add:

   ```ts
   it("rejects a missing mode", async () => {
     const response = await app.request(
       "/api/v1/content/question-bank?examCode=AZ-305",
       { headers },
       { QUESTION_BANK_URL: driveUrl },
     );
     expect(response.status).toBe(400);
   });
   ```
3. Run `npm test`. It should pass because `mode` is required by the schema.
4. *Break it on purpose* (temporarily make `mode` optional in the contract) and confirm the test fails. A test you have never seen fail is not yet trustworthy.

## 7.10 Testing patterns glossary

| Pattern | Meaning | Example here |
| --- | --- | --- |
| Arrange-Act-Assert | Prepare, call, check. | Every test. |
| Test double / fake input | Provide a fake environment. | The `env` argument with a test URL. |
| Golden/exact match | Compare to an exact expected object. | Test 9 `toEqual`. |
| Negative testing | Check bad input is rejected. | Tests 2, 4, 5, 7, 8, 10, 11, 12. |
| Boundary testing | Test limits. | Extend by adding tests for max lengths. |

## 7.11 Self-check questions

1. Why does `app.request` make tests faster than starting a server?
2. Why pass `env` as the third argument in tests?
3. Which test would fail first if someone made `.strict()` into a normal object? Why?
4. What does "negative testing" mean?
5. Why watch a test fail at least once?

## 7.12 Exercises

1. Write a test that a `POST` to `/api/v1/content/question-bank` is not allowed (what does Hono return? Is it consistent with the error envelope? If not, propose a fix).
2. Write a test that an `examCode` of 65 characters is rejected.
3. Write a test proving that the response never contains a `questions` field.

Next: [Chapter 8 - Security and safe logging](08-security-and-logging.md)
