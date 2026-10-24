# Chapter 8 - Security and Safe Logging

Security is not a feature added at the end. In this project it shows up in small decisions on almost every line. This chapter names them.

## 8.1 What / Why / Pattern / Reference

- **What:** security means making sure only the right people can do the right things, and that mistakes do not expose private data.
- **Why:** this API will eventually decide who may take paid exams and will help deliver exam content and keys. A leak of questions or answers harms the business.
- **Pattern:** *defense in depth* - several independent layers so one failure is not fatal.
- **Reference:** OWASP Top 10: https://owasp.org/www-project-top-ten/ and OWASP API Security Top 10: https://owasp.org/API-Security/

## 8.2 The security controls in this codebase

| Control | Where | Protects against |
| --- | --- | --- |
| Strict schemas (`.strict()`) | contract | Unexpected or smuggled fields (mass assignment). |
| Length limits (`max`) | contract | Oversized inputs, resource abuse. |
| Enum values | contract | Injection of unexpected values. |
| Required headers | global middleware | Anonymous, unidentified callers. |
| Content-Type check | authorize middleware | Parser confusion. |
| Safe error envelope | `errorResponse` | Leaking internals in errors. |
| Ignored thrown error | `onError` | Stack-trace or SQL leakage. |
| Log sanitization | global middleware | Log injection and secret leakage. |
| HTTPS-only + host allow-list | question-bank schema | Redirecting users to attacker sites; unencrypted downloads. |
| Config in settings | `wrangler.jsonc` | Secrets or URLs hard-coded in source. |
| SHA-256 verification | app side | Corrupted or tampered downloads. |
| No secrets in repo | project rule | Credential theft from source control. |

## 8.3 OWASP concepts mapped to this project

### Broken access control (OWASP A01)

- **What:** users doing things they should not be allowed to do.
- **Here:** M1 only checks that a bearer header *exists*. Anyone can get the Drive URL. This is an **intentional, documented M1 limit**, not a finished control.
- **Future:** verify a signed lease (ES256), check device binding, check entitlement for the exam and mode, then respond.

### Injection (OWASP A03)

- **What:** untrusted text interpreted as commands or markup.
- **Here:** all input is validated against strict schemas. Query parameters are never concatenated into SQL (there is no database yet). When D1 is added, always use parameterized queries (`prepare(...).bind(...)`), never string building.

### Security misconfiguration (OWASP A05)

- **What:** unsafe defaults, verbose errors, wrong settings.
- **Here:** a bad `QUESTION_BANK_URL` is caught by validation and reported as a generic 503. Detailed errors are not shown to clients. `compatibility_date` is pinned.

### Cryptographic failures (OWASP A02)

- **What:** missing or weak protection of data in transit or at rest.
- **Here:** the download URL must be `https`. Hash integrity uses SHA-256. Real key handling is future work and must follow the requirements document.

### Logging and monitoring failures (OWASP A09)

- **What:** not logging enough, or logging the wrong things.
- **Here:** structured logs with a request ID, plus a rule about what must never be logged (next section).

### Server-side request forgery / open redirect style risks

- **What:** tricking a server or client into contacting an attacker-chosen address.
- **Here:** the host allow-list means a wrong setting cannot point the app at an arbitrary website.

## 8.4 Logging rules

**Never log:** passwords, full tokens or bearer values, tickets, key material, device public keys, email addresses, full request bodies, or anything a user typed that was not validated.

**Do log:** a request ID, a timestamp (added by the platform), route, status, app version (bounded), validated platform.

```ts
console.info(JSON.stringify({
  event: "api_request",
  requestId,
  appVersion,                              // trimmed to 64 chars
  platform: platform.success ? platform.data : "unknown",   // only a validated value
}));
```

**Why structured (JSON) logs?** Tools can filter by field: "show all `api_request` events for `platform: ios`".

**Why a request ID?** A user reports "it failed". The error body and header contain the ID. You search logs for it and see the exact request, without the user sharing anything sensitive.

## 8.5 Handling secrets properly

| Kind of value | Where it belongs | Why |
| --- | --- | --- |
| Public configuration (a public file URL) | `vars` in `wrangler.jsonc` | Not sensitive; versioned with code. |
| Secret (API key, private signing key, service-account JSON) | `npx wrangler secret put NAME` or the Cloudflare dashboard | Encrypted at rest, not in source control. |
| Local development secrets | `.dev.vars` file, which must be excluded from source control (add it to `.gitignore`) | Never committed. |

If you ever commit a secret by accident, **rotate it** (create a new one, revoke the old one). Deleting it from a later commit does not remove it from history.

## 8.6 The public-link problem, honestly

The new feature returns a fixed Google Drive link. Think through the threat:

1. Any app user can read the answers because the link is public and identical for all.
2. A user can share the link; it works for anyone with it.
3. The URL appears in logs on the client and in network inspection tools.

What this means: for a **sample** or **non-confidential practice** bank it is acceptable. For paid or proctored exam content it is not. The requirements describe a more robust target: lease-based authorization, device-bound short-lived tickets, streamed encrypted packages, and one-time key release. The pointer design here is a stepping stone.

Ways to harden, from small to large:

1. Keep the file private; use a service account on the Worker to stream it after real lease verification (Worker has the Drive credentials, the app never sees a Drive URL).
2. Use short-lived signed URLs (for example Cloudflare R2 presigned URLs).
3. Encrypt the file and release the key only to authorized devices.
4. Remove `answer` fields from the downloadable file and check answers server-side (changes the offline requirement).

## 8.7 Rate limiting and abuse

An endpoint that anyone can call repeatedly invites abuse. Not implemented in M1. The contract already reserves `RATE_LIMITED`. Cloudflare offers rate-limiting rules and Workers bindings for it.

## 8.8 Security review checklist for any new route

- [ ] Does it require the app headers (via the global middleware)?
- [ ] Does it require the right authorization, and does it actually *verify* it?
- [ ] Are all inputs validated with a strict schema and length limits?
- [ ] Does every error use the envelope, with a safe message?
- [ ] Are unexpected exceptions hidden by `onError`?
- [ ] Is anything sensitive logged?
- [ ] Are new settings typed in `AppBindings` and is the secret/non-secret choice right?
- [ ] Is the response validated against a contract schema?
- [ ] Are there tests for the unhappy paths?

## 8.9 Self-check questions

1. Why is the platform logged only after `safeParse`?
2. Why is the bearer check in M1 not real security?
3. Where should a service-account key live?
4. What does "defense in depth" mean? Give two examples from the code.
5. What is the risk of a static public download link?

## 8.10 Exercises

1. Search the repository for `console.` and list everything that is logged. Confirm none of it is sensitive.
2. Write a threat list for the question-bank route using the headings: who is the attacker, what do they want, how could they get it, what stops them today.
3. Propose how a signed, expiring download URL would change the response contract (which fields would you add?).

Next: [Chapter 9 - Patterns catalog](09-patterns-catalog.md)
