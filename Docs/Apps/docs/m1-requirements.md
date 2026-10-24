# Exam Simulator M1 Requirements

## Purpose

M1 establishes the offline-first application shell, package contract, service boundaries, deterministic mocks, tests, and a minimal Worker boundary. Application releases and content releases remain independent.

## Scope

- React Native Community CLI with TypeScript; Expo is not used.
- Exactly six routes: Welcome, Authorization, Sync, Exam Home, Question, and Results.
- Screens depend on typed service interfaces delivered by the composition root.
- Exam and Prep packages are separate ZIPs; both include correct answers, while only Prep includes `explanations.json`.
- Package validation runs independently of the native application.
- Worker endpoints return fixed stub data only; no D1, Drive, identity lookup, leases, or production secrets.
- Windows and Android targets are planned. Platform verification must be reported honestly; iOS is NOT VERIFIED on Windows.

## Acceptance Baseline

`npm test -- --runInBand`, `npm run typecheck`, package fixture validation, and Worker type checking are the local checks. Native builds require the corresponding platform SDK and are not claimed unless run successfully on that environment.