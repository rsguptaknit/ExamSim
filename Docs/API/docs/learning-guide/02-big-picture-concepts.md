# Chapter 2 - Big-Picture Concepts

This chapter explains the ideas behind the project. Each section: What, Why, Pattern, Reference.

## 2.1 Client and server

- **What:** A *client* asks for something; a *server* answers. Here the client is the Exam Simulator app (Android, iOS, or Windows). The server is this Worker.
- **Why:** The app needs things it cannot or should not keep on the device by itself: who is allowed to use it, which exams they may take, and where to get exam content.
- **Pattern:** *client-server architecture*, using *request/response*: every interaction is one request followed by one response. The server never speaks first.
- **Reference:** MDN, "Client-Server overview": https://developer.mozilla.org/docs/Learn/Server-side/First_steps/Client-Server_overview

## 2.2 What an API is

- **What:** An *API* (Application Programming Interface) is a menu of operations one program offers to another, with rules for how to ask. A *web API* offers them over HTTP.
- **Why:** Separating the app from the server lets each be built and changed independently, as long as they follow the same menu (the *contract*, Chapter 4).
- **Pattern:** *interface/contract separation* - callers depend on the documented interface, not on how it is implemented.
- **Where in this project:** the four routes in `src/index.ts`.

## 2.3 HTTP in detail

HTTP is the language of the web. Every message has a few parts.

### A request has

```http
GET /api/v1/content/question-bank?examCode=AZ-305&mode=prep HTTP/1.1
Host: exam-simulator-api-dev.<your-subdomain>.workers.dev
Authorization: Bearer m1-placeholder
X-App-Version: 0.1.0
X-Platform: android
```

| Part | Example | Meaning |
| --- | --- | --- |
| **Method** | `GET` | What kind of action. |
| **Path** | `/api/v1/content/question-bank` | Which resource or operation. |
| **Query string** | `?examCode=AZ-305&mode=prep` | Extra key=value options after `?`, separated by `&`. |
| **Headers** | `Authorization: ...` | Metadata about the request. |
| **Body** | (JSON for `POST`) | The data being sent. `GET` normally has none. |

### A response has

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Request-Id: 8b0c...

{ "apiVersion": 1, "exam": { ... }, "file": { ... } }
```

| Part | Meaning |
| --- | --- |
| **Status code** | A number summarizing the result. |
| **Headers** | Metadata (here, the request ID for support). |
| **Body** | The data, usually JSON. |

### Methods used here

| Method | Use | In this project |
| --- | --- | --- |
| `GET` | Read something; must not change data. | health, content/version, content/question-bank |
| `POST` | Send data to be processed. | authorize |

### Status codes used here

| Code | Name | Where it appears |
| --- | --- | --- |
| 200 | OK | Successful routes. |
| 400 | Bad Request | Missing app headers, invalid body or query, unknown route. |
| 401 | Unauthorized | Missing bearer header on protected routes. |
| 404 | Not Found | No question bank for that exam and mode. |
| 500 | Internal Server Error | Unexpected exception (hidden details). |
| 503 | Service Unavailable | The Drive URL setting is missing or invalid. |

Rule of thumb: **4xx means the caller should change something; 5xx means the server has the problem.**

- **Pattern:** *uniform interface* (REST) - standard methods and status codes so every client understands the result.
- **Reference:** https://developer.mozilla.org/docs/Web/HTTP and https://developer.mozilla.org/docs/Web/HTTP/Status

## 2.4 Headers and why this API requires some

- `X-App-Version` and `X-Platform` identify which app build and platform is calling. The `X-` prefix is an old convention for custom headers.
- `Authorization: Bearer <token>` carries proof of permission. "Bearer" means whoever bears (holds) the token is trusted, so tokens must be protected.
- `Content-Type: application/json` tells the server how to read a body.
- `X-Request-Id` is added by the server to every response so a failed call can be traced in logs.

**Why:** the API requirements need to know the app version and platform for diagnostics and future compatibility rules, and need a standard place to carry the lease (a short-lived permission token) later.

**Pattern:** *cross-cutting concern* - checks that apply to many routes (headers) are done once, in middleware, not repeated in each route.

## 2.5 JSON

- **What:** JSON (JavaScript Object Notation) is a text format for structured data: objects `{}`, arrays `[]`, strings, numbers, booleans, `null`.
- **Why:** every language can read and write it, and humans can read it too.
- **Pattern:** *data interchange format*.
- **Where in this project:** all request bodies and responses; also the question-bank file itself.

Example from the sample question bank (shortened):

```json
{
  "apiVersion": 1,
  "exam": { "code": "AZ-305", "title": "AZ-305 Practice Question Bank", "contentVersion": "1.1.0", "mode": "prep" },
  "questions": [
    {
      "id": "q001",
      "topic": "Identity, governance and monitoring",
      "difficulty": "easy",
      "text": "...",
      "options": [{ "id": "A", "text": "..." }],
      "answer": "A",
      "explanation": "..."
    }
  ]
}
```

The real file has 50 questions and is about 33 KB. Notice it contains `answer` fields. Remember that for the security discussion in Chapter 8.

## 2.6 REST and route design

- **What:** REST is a style for designing web APIs around *resources* (things) identified by URLs, acted on by HTTP methods.
- **Why:** predictable URLs make an API easy to learn.
- **Pattern:** *resource-oriented URLs* and *versioned paths*.

Reading `/api/v1/content/question-bank`:

| Segment | Meaning |
| --- | --- |
| `/api` | This is the API, not a web page. |
| `/v1` | Version 1 of the API. If a breaking change is needed later, `/v2` can exist beside it so old apps keep working. |
| `/content` | The group of routes about learning content. |
| `/question-bank` | The specific resource. |

`/api/health` has no version because health checks must stay stable for monitoring tools.

## 2.7 Serverless and Cloudflare Workers

- **What:** *Serverless* means you provide code and the platform runs it on demand; you do not rent or manage a server. A *Cloudflare Worker* is a small program that runs in Cloudflare's data centers around the world, close to the caller (the "edge").
- **Why:** no machines to maintain, automatic scaling, low latency, and a free tier suitable for development.
- **How it works:** Cloudflare runs your code in lightweight *isolates* (sandboxed JavaScript contexts) rather than full operating-system servers. Your module exports a `fetch` handler; Cloudflare calls it for every incoming request. Hono's `app` object provides that handler, which is why `src/index.ts` ends with `export default app`.
- **Trade-offs:**
  - Workers are not Node.js. Some Node APIs are missing or need a compatibility flag; browser-style web standards (`fetch`, `Request`, `Response`, `URL`, `crypto`) are available.
  - Each request should be fast and stateless; do not rely on variables surviving between requests.
  - Size and CPU-time limits apply to a Worker, which is one reason big files are not sent through it (Chapter 6).
- **Pattern:** *function as a service (FaaS)* and *stateless request handling*.
- **Reference:** https://developers.cloudflare.com/workers/

### Web-standard objects you will meet

| Object | What it is | Where used |
| --- | --- | --- |
| `Request` | An incoming HTTP request. | Wrapped by Hono as `context.req`. |
| `Response` | An outgoing HTTP response. | `Response.json(...)` in `errorResponse`. |
| `URL` | A parsed web address. | Reading query parameters, checking the Drive host. |
| `crypto.randomUUID()` | Generates a random unique ID. | The request ID. |

## 2.8 Configuration, environment variables, and bindings

- **What:** A *setting* is a value that can differ between environments (your laptop, a test server, production). Cloudflare gives a Worker its settings as *bindings*; plain-text ones are called *variables* (`vars`).
- **Why:** the same code must run everywhere. The Drive URL for development is not the Drive URL for production, so it must not be baked into code.
- **Pattern:** *twelve-factor app, config in the environment*.
- **Where in this project:** `QUESTION_BANK_URL` in `wrangler.jsonc`, typed by `AppBindings` in `src/index.ts`, read as `context.env.QUESTION_BANK_URL`.
- **Important:** `vars` are plain text visible to anyone with dashboard access. Real secrets (keys, passwords) must use `wrangler secret put`, never `vars` and never the repository.
- **Reference:** https://developers.cloudflare.com/workers/configuration/environment-variables/

## 2.9 Stubs, milestones, and why M1 is fake on purpose

- **What:** A *stub* is a simplified stand-in that returns fixed, predictable answers. `authorize` returns `{"status":"stub","entitlements":[]}` regardless of who asks.
- **Why:** building the whole secure system at once is risky. A stub lets the app team connect and test wiring (URLs, headers, JSON shapes, error handling) while the real logic is built later. The word `stub` in responses makes it impossible to mistake the fake for the real.
- **Pattern:** *walking skeleton* / *milestone-based delivery* and *explicit placeholder*.
- **Reference:** the "M1 boundary" section of `README.md`.

## 2.10 The "return a pointer, not the data" idea (preview)

When a response would be large, the server can return a *link* to the data and let the client fetch it directly from where it is stored. This is common (for example, upload/download links for cloud storage). The question-bank route applies it to Google Drive. Details are in Chapter 6.

**Pattern:** *claim check* or *reference instead of payload*.

## 2.11 Self-check questions

1. Which HTTP status would you return if the caller sent an unknown query parameter, and why?
2. Why is `/api/health` outside `/v1`?
3. What does "stateless" mean for a Worker?
4. Why must a real password never be placed in `vars`?
5. What does a stub protect you from?

Next: [Chapter 3 - Toolchain and configuration](03-toolchain-and-configuration.md)
