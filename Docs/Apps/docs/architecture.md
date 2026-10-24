# Architecture

## Dependency Direction

Screens call capabilities from `src/core/services/ServiceContracts.ts`; implementations and platform details remain behind that boundary. `src/core/services/ServiceContainer.tsx` is the only application file that creates concrete services. The app bootstrap validates configuration before mounting providers.

## Runtime

React Native Community CLI was selected to retain native Windows and Android build paths without introducing Expo. React Navigation provides the typed native stack. React `useReducer` and separate state/dispatch contexts own application state. React Native Windows New Architecture/Fabric is a future build requirement and has not been verified in this Windows workspace.

## Gateway Layer

The app never reads the API's JSON directly. `src/core/gateway/QuestionBankGateway.ts` requests the payload through `ApiClient`, selects a mapper by the payload's `apiVersion`, and returns a `QuestionBank` built from the model classes in `src/core/models/Question.ts`. `QuestionBankMapper.ts` is the only code that knows the wire format; it validates the untrusted payload at runtime, ignores unknown extra fields, and reports a reason that is logged but never shown to users (the UI sees only `PROTOCOL_ERROR`).

To absorb an API change, add a mapper for the new `apiVersion` (and, if the endpoint moved, pass a different `buildPath`) in the gateway layer. Screens, state, models, and the exam engine do not change. `tests/unit/question-gateway.test.ts` demonstrates this with a hypothetical v2 payload.

Sync loads the bank through the gateway. If the API is unreachable (`NETWORK_UNAVAILABLE`) it falls back to the bundled sample set, keeping the app offline-first; malformed payloads and authorization failures are not masked.

## M1 Limits

Authorization and connectivity are deterministic mocks. Exam/Prep mode selection is client-selected for this demonstration only. M1 does not implement authorization policy, content encryption, offline caching of downloaded content, or production package handling. Scoring counts answers recorded in memory during a session and is not persisted.