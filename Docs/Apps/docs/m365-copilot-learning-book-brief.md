# Microsoft 365 Copilot Brief: Build a Front-End Learning Book From This File Alone

## Instructions to the Book Author

Create a complete, student-friendly front-end learning book using only the information in this Markdown document. The reader will not receive the original repository, its source files, or another reference document. Do not tell the reader to inspect an unavailable codebase. Do not assume they can open source files not reproduced in the book.

The book must teach the technical concepts used by the described ExamSimulator project, explain them with examples matching this specification, and guide a beginner through building the project from an empty folder. Include complete source listings in the book for every file a learner must create or modify. For long configuration files, provide the complete minimum working version and explain each important setting. Do not replace implementation listings with phrases such as "add the code from the repository" or "use the existing component."

Write as a patient instructor for a first- or second-year college student who understands basic computer use but may be new to TypeScript, React, mobile apps, asynchronous code, and automated tests. Define technical terms before relying on them. Explain the purpose of code, the inputs and outputs, how data moves, common mistakes, and how to verify each step. Use small code excerpts to teach concepts, then provide a coherent complete project in the hands-on chapters or appendix.

Treat code samples in this brief as the intended design and behavior of the project. You may improve formatting or fill in small implementation details needed to make a complete instructional example, but do not claim to reproduce a source line that is not present here. Clearly label any extension or simplification. Keep the final result technically consistent across its diagrams, types, examples, tests, file tree, commands, and completed listings.

## Intended Book

**Suggested title:** *From React Basics to a Cross-Platform Exam Practice App*

The book should be a hands-on front-end development text, not a marketing brochure or a general React encyclopedia. It should explain the architecture by repeatedly connecting a concept to the ExamSimulator example. At the end, a learner who follows the chapters should be able to create and run a browser version of the complete six-route project, understand how the shared React Native code is structured, and explain what would be required to run the native targets.

### Reader Outcomes

By completing the book, a student should be able to:

1. Explain the difference between JavaScript, TypeScript, JSX, React, React Native, React Native Web, Metro, Babel, and Vite.
2. Create a typed React Native Community CLI project and identify the app entry points.
3. Build function components, pass props, compose children, and apply styles.
4. Use `useState`, `useEffect`, `useRef`, Context, and `useReducer` appropriately.
5. Navigate between six screens and explain the native and web navigation implementations.
6. Keep UI code independent from concrete service implementations through interfaces and a composition root.
7. Handle asynchronous operations, loading states, cleanup, errors, and safe user messages.
8. Define and validate a versioned JSON content format stored in a ZIP package.
9. Write unit and component tests using Jest, mocks, and fixtures.
10. Explain the limits of the M1 mocks and avoid claiming that unfinished production features exist.
11. Recreate the project from the book alone, run the browser app, and verify its behavior.

### Pedagogical Rules

- Introduce one major idea at a time and keep each chapter's code runnable.
- Show expected folder paths before each code listing.
- Explain every non-trivial line or logical block. Avoid unexplained magic values and unexplained imports.
- Never imply that TypeScript types validate untrusted JSON at runtime.
- Use original example text. Do not reproduce proprietary exam content or copied textbook passages.
- Use a consistent naming scheme across all code listings.
- Include checkpoints, exercises, common errors, expected outcomes, and short review questions in each practical chapter.
- Provide complete answers or solution sketches in a clearly separated instructor/solutions section.
- Mark optional native platform work separately from the browser-first learning path.
- Do not claim Windows, Android, or iOS builds pass without actual platform build evidence.

## Project Definition

### Product

ExamSimulator is an offline-first, cross-platform exam-practice learning project. Its M1 implementation is a UI and architecture foundation, not a production exam service. It demonstrates a six-screen practice flow using React Native Community CLI, React Native Web, TypeScript, React state, a small service boundary, a gateway layer that maps API JSON into model classes, a bundled offline fallback set, tests, and an optional HTTP Worker that serves a 50-question sample.

### Technology Baseline

Use these versions and tools as the described baseline. If a generated book chooses a newer compatible version, it must say so and keep commands and APIs consistent throughout.

- Node.js: 22.11 or newer.
- npm for package installation and scripts.
- React Native Community CLI, React 19, React Native 0.88, TypeScript 6.
- React Navigation native stack 7 for native navigation.
- React Native Web and Vite 8 for the browser preview.
- Jest 29 and React Native Testing Library for tests.
- JSZip for the in-memory sample package and AdmZip for Node package validation.
- Optional Cloudflare Worker with Wrangler for a fixed-response HTTP boundary.
- Android Gradle/Kotlin files and iOS Xcode/Swift files are native host projects, not the place where the shared screen logic is implemented.

Use the actual package scripts below in the generated material:

```json
{
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "lint": "eslint .",
    "start": "react-native start",
    "web": "vite --host 0.0.0.0",
    "test": "jest",
    "typecheck": "tsc --noEmit",
    "validate:package": "node tools/content-preparation/validate-package.js"
  },
  "engines": {
    "node": ">= 22.11.0"
  }
}
```

### Run and Verification Commands

Explain what each command starts or checks; distinguish servers from one-shot checks.

```sh
npm install
npm run web
npm test -- --runInBand
npm run typecheck
npm run validate:package -- tests/fixtures/valid-exam.zip
npm run validate:package -- tests/fixtures/valid-prep.zip
```

The browser app opens at `http://localhost:5173`. For Android, configure the Android SDK and JDK, run `npm start` in one terminal, and `npm run android` in another. iOS requires macOS, Xcode, and CocoaPods. The project’s native platform statuses must be described as unverified unless the reader actually builds and tests those targets.

For the optional Worker, work in its own directory:

```sh
cd worker
npm install
npm run typecheck
npm run dev
```

The app's screens call the Worker's question endpoint through the gateway layer (see the Gateway concept below), but authorization remains a mock and never calls the Worker. When the Worker is not running, sync falls back to a bundled 10-question set, so the browser walkthrough works without it. Present starting the Worker as optional: it switches the app from 10 bundled questions to the 50-question API sample.

## Architectural Requirements

The project must retain these boundaries in every code sample and the final hands-on implementation:

- Exactly six routes: `Welcome`, `Authorization`, `Sync`, `ExamHome`, `Question`, `Results`.
- Native stack navigation and browser navigation may use different implementation mechanisms, but they render the same screen components and preserve those route names.
- Screens call typed service interfaces. Concrete services are created only in `src/core/services/ServiceContainer.tsx`.
- Keep application state in a small reducer and contexts; do not add Redux, Zustand, MobX, Expo, or a generic dependency-injection framework.
- Keep package validation independent from React Native.
- The standalone browser package preview must not import the app's `src/` renderer code.
- Exam and Prep packages are separate. Both include correct answers. Only Prep includes `explanations.json`.
- M1 ZIP files contain plain JSON. ZIP packaging is not encryption.
- Use stable application error codes and sanitized display messages. Avoid logging credentials; redact token, password, key, secret, passcode, and authorization fields.
- Do not add D1, Google Drive credentials, real authorization, device binding, lease signing, key release, or production encryption as part of the M1 teaching implementation.
- Explain that future work must be separately designed, authorized, tested, and documented; do not silently include production features in beginner code.

## Six-Screen User Story

The finished teaching project shows this journey:

1. **Welcome:** a short introduction and Start button.
2. **Authorization:** an email, exam code (initially `AZ-305`), and Exam/Prep mode selector. The form validates required values and email shape locally. It calls mock connectivity and authorization interfaces, then moves to Sync. The mock does not verify real identity.
3. **Sync:** displays a pending message while the sync service asks a question-bank gateway for a `QuestionBank`. The gateway fetches a versioned JSON payload from the API (`GET /api/v1/questions?examCode=...&mode=exam|prep`), and a mapper translates it into app-owned `Question` and `QuestionBank` model classes. If the API is unreachable, sync falls back to a bundled 10-question set. On success it stores session metadata and advances to Exam Home. Teach the gateway/mapper pattern as the way the app absorbs API changes without changing screens.
4. **Exam Home:** displays exam code, mode, and available question count, then begins the practice flow.
5. **Question:** loads a prompt and choices from the exam engine and requires a selection before moving forward. Selecting a choice records the answer and returns feedback (correct answer and optional explanation); the screen shows it in Prep mode only. The answer key is not returned until the learner submits a choice.
6. **Results:** displays the score, which counts the correct answers recorded in memory during the session, and allows a user to reset and return to Welcome. Scores are not persisted and the answer key lives in the app, so this is practice scoring, not secure assessment.

Route transition methods have distinct meanings: `navigate` pushes a new route, while `replace` swaps the current route so a completed one-way step is not reopened by Back.

## Concepts the Book Must Teach

### 1. Web, Mobile, and Cross-Platform UI

Explain that a web page uses browser DOM elements, while React Native uses platform components such as `View`, `Text`, `Pressable`, and `TextInput`. React Native Web provides browser implementations for those components. Shared React screen components can therefore target multiple platforms, but native app launch and build configuration remain platform-specific.

Show why project entry points differ:

- `index.js` registers the app with React Native's `AppRegistry`.
- `web.tsx` finds a DOM root and mounts the web app with `createRoot`.
- `App.tsx` composes the native providers and native navigator.
- `App.web.tsx` composes the browser providers and browser navigator.
- `vite.config.ts` maps `react-native` imports to `react-native-web` and prefers `.web.tsx` files.

Explain that a shared component does not mean every native platform build has been verified.

### 2. JavaScript, TypeScript, JSX, and Compilation

Define JavaScript as the runtime language, TypeScript as static analysis and type syntax, and JSX as syntax for describing UI trees inside JavaScript/TypeScript. Explain that Babel/Metro/Vite transform source for a runtime, while TypeScript checking catches many mistakes before runtime. A type such as `ContentSession` does not inspect arbitrary JSON received over a network or read from a ZIP.

Teach basic concepts actually used: imports/exports, functions, object and array literals, optional chaining (`?.`), nullish fallback (`??`), union types, interfaces, generic functions, promises, `async`/`await`, `try`/`catch`/`finally`, and immutable object spread (`...state`).

### 3. React Function Components and Composition

Explain a component as a function that returns UI. Explain JSX nesting, reusable child components, props, and `children`. Use `ScreenFrame` as the example: it accepts a title and children, and composes a scrollable screen layout around the content.

Include an illustrative, complete small example:

```tsx
import React from 'react';
import { ScrollView, Text, View } from 'react-native';

type ScreenFrameProps = React.PropsWithChildren<{ title: string }>;

export function ScreenFrame({ title, children }: ScreenFrameProps) {
  return (
    <ScrollView contentContainerStyle={{ flexGrow: 1, padding: 24 }}>
      <View>
        <Text>{title}</Text>
        {children}
      </View>
    </ScrollView>
  );
}
```

Explain that inline styles above are pedagogical shorthand; the completed project should use a shared `StyleSheet` for reusable styles.

### 4. Props, Local State, and Controlled Inputs

Use the Authorization screen to explain that React props are inputs provided by a parent and React state stores data that changes over time. A controlled `TextInput` displays a state value and calls its setter on change. Explain why email, exam code, selected mode, busy state, and error message belong to the screen while it is mounted, whereas candidate/session data must be shared across routes.

Illustrative pattern:

```tsx
const [email, setEmail] = useState('');

<TextInput
  value={email}
  onChangeText={setEmail}
  keyboardType="email-address"
/>
```

Explain that setting state schedules a render; do not mutate variables or objects as if they were React state.

### 5. `useEffect`, Promises, Cleanup, and Stale Results

Explain that effects synchronize a component with asynchronous work or external systems. A promise may complete after its screen has unmounted. In Sync and Question screens, a local `active` boolean is set to false in the effect cleanup; callbacks update state or navigate only while active.

Show the pattern:

```tsx
useEffect(() => {
  let active = true;
  loadQuestion().then(value => {
    if (active) setQuestion(value);
  });
  return () => {
    active = false;
  };
}, [loadQuestion]);
```

Explain effect dependencies and why using an incorrect dependency list can lead to stale values or repeated requests. Include error handling in the complete sample; do not teach a fire-and-forget promise as production quality.

### 6. `useRef` as a Non-Rendering Guard

Explain that a ref persists between renders but changing its `.current` does not cause a render. Authorization uses a ref to close the small gap between pressing Continue and React rendering the busy state, preventing a double submit. Contrast this with `useState`, which is used to update visible loading UI.

### 7. Context and `useReducer`

Explain Context as a way to provide state/services to nested components without passing props through every layer. Explain a reducer as a pure function `(state, action) -> newState`, useful when updates have named transitions. In this project, separate contexts carry state and dispatch.

Use the actual domain shape:

```ts
type ExamMode = 'exam' | 'prep';

interface ContentSession {
  examCode: string;
  mode: ExamMode;
  contentVersion: string;
  questionCount: number;
}

type AppAction =
  | { type: 'APP_INITIALIZED' }
  | { type: 'AUTHORIZED'; examCode: string; mode: ExamMode }
  | { type: 'SYNC_COMPLETED'; session: ContentSession }
  | { type: 'SESSION_RESET' };
```

The book should show a reducer with an exhaustive `switch`, return fresh objects, and explain the four transitions. The completed implementation may include the candidate field on `AUTHORIZED`; include it consistently in all later files and tests.

### 8. Navigation and Typed Route Contracts

Teach native navigation with React Navigation's typed native stack. The route parameter type has exactly six keys, each mapped to `undefined` because this M1 flow does not pass route parameters. Explain that TypeScript catches misspelled route names at compile time but does not decide navigation UX.

Explain the browser implementation's simple in-memory route stack and the screen-facing `navigate` and `replace` methods. Call out the tradeoff: this is adequate for a small learning preview but does not provide full browser history, deep linking, or all accessibility behaviors of a production routing solution.

### 9. Service Interfaces and the Composition Root

Explain interfaces as capability contracts and a service container as the one place that connects implementations. This project has nine service capabilities:

- authorization
- sync
- exam repository
- encryption transform
- exam engine
- screen renderer placeholder
- connectivity
- API client
- logger

Show the conceptual contract and dependency use:

```ts
interface AuthorizationService {
  authorize(email: string, examCode: string, mode: ExamMode): Promise<AuthorizedCandidate>;
}

interface ServiceContainer {
  authorization: AuthorizationService;
  sync: SyncService;
  examEngine: ExamEngine;
  connectivity: ConnectivityService;
  // Explain the remaining capabilities in the complete listing.
}
```

Clearly label abbreviated interfaces as excerpts. Explain that concrete mock implementations live in `ServiceContainer.tsx`; screens receive them through `useServices()`. Tests can replace the provider hook with deterministic fakes. The identity encryption transform is a deliberate placeholder and must never be described as encryption.

### 9A. Gateway, Mapper, and Model Classes (Absorbing API Changes)

Teach this as a dedicated chapter. A gateway (anti-corruption layer) sits between the app and the API. It fetches the API's JSON, a mapper validates and translates it into app-owned model classes, and only those models reach the rest of the app. When the API changes shape, a new mapper is added in the gateway layer and no screen, state, model, or exam-engine code changes.

Data flow:

```text
Worker API -> ApiClient -> QuestionBankGateway -> QuestionBankMapper -> QuestionBank (model) -> sync service + exam engine -> screens
```

**Model classes** (`src/core/models/Question.ts`) know nothing about the wire format:

```ts
export class Question {
  constructor(
    readonly id: string,
    readonly prompt: string,
    readonly choices: readonly string[],
    readonly correctAnswer: string,
    readonly explanation?: string,
  ) {}
  isCorrect(choice: string): boolean { return choice === this.correctAnswer; }
}

export class QuestionBank {
  constructor(
    readonly examCode: string,
    readonly title: string,
    readonly mode: ExamMode,
    readonly contentVersion: string,
    readonly questions: readonly Question[],
  ) {}
  get count(): number { return this.questions.length; }
  questionAt(index: number): Question | undefined { return this.questions[index]; }
}
```

**Sample API response (v1 wire format).** The Worker serves `GET /api/v1/questions?examCode=AZ-305&mode=exam|prep`. The sample contains 50 original questions across four topics (identity/governance/monitoring, data storage, business continuity, infrastructure). Show two or three questions in the book in this exact shape and explain that the full file is a list of the same objects:

```json
{
  "apiVersion": 1,
  "exam": { "code": "AZ-305", "title": "AZ-305 Practice Question Bank", "contentVersion": "1.1.0", "mode": "prep" },
  "questions": [
    {
      "id": "q001",
      "topic": "Identity, governance and monitoring",
      "difficulty": "easy",
      "text": "Several teams deploy into many subscriptions. You must consistently block deployment of unapproved resource types. What should you use?",
      "options": [
        { "id": "A", "text": "Azure Policy assigned at a management group" },
        { "id": "B", "text": "A resource lock on each resource group" },
        { "id": "C", "text": "An Azure Monitor alert rule" },
        { "id": "D", "text": "A network security group on each subnet" }
      ],
      "answer": "A",
      "explanation": "Policy evaluates resource properties at deployment time, and a management group assignment is inherited by every subscription beneath it."
    }
  ]
}
```

The Worker omits `explanation` when `mode=exam` and includes it for `prep`; it returns 400 for a missing exam code or invalid mode and 404 for an unknown exam code.

**Field mapping:** `exam.code`, `exam.title`, `exam.mode`, `exam.contentVersion` map to the `QuestionBank` fields of the same meaning. `text` becomes `Question.prompt`. `options[].text` becomes the `choices` strings. `answer` is an option id such as `"A"`; the mapper resolves it to that option's text, stored as `correctAnswer`, so the model never holds option ids. `topic`, `difficulty`, and unknown extra fields are ignored. `apiVersion` selects the mapper and is not stored.

**Mapper behavior** (`QuestionBankMapper.ts`): defines `interface QuestionBankMapper { readonly apiVersion: number; map(payload: Record<string, unknown>): QuestionBank }` and a v1 implementation. Every field is read through runtime-checked helpers (non-empty string, array, object, mode is `exam` or `prep`), because a TypeScript cast does not validate received data. It requires 2 to 6 options with unique ids and unique texts, an `answer` that matches an option id, unique question ids, and a non-empty question list. Failures throw a `PayloadMappingError` carrying a developer-facing reason that names the field path, for example `questions[3].answer does not match an option id`.

**Gateway behavior** (`QuestionBankGateway.ts`): `createQuestionBankGateway({ apiClient, logger, mappers = [v1], buildPath })` returns `fetchQuestionBank(examCode, mode)`. It requests `/api/v1/questions?examCode=<encoded>&mode=<mode>`, reads `apiVersion`, and picks the matching mapper; an unknown version is rejected. It rejects a payload whose `exam.mode` differs from the requested mode so Prep content cannot enter an Exam session. A `PayloadMappingError` is logged with its reason and converted to the stable `PROTOCOL_ERROR`, so the user sees only the generic message. Transport errors from `ApiClient` already carry stable codes and pass through. Both `buildPath` and `mappers` are injectable.

**Absorbing an API change:** show a hypothetical v2 payload (`choices: string[]`, `answerIndex`, renamed fields, a moved path) and a second mapper registered through `mappers: [v1, v2]` plus a new `buildPath`. State clearly which files do not change.

**Sync and engine** (`QuestionBankServices.ts`): an in-memory `ActiveExam` holds the loaded `QuestionBank` and the answers recorded so far. The sync service fetches through the gateway, stores the bank, and returns session metadata (`examCode`, `mode`, `contentVersion`, `questionCount`). Only when the API is unreachable (`NETWORK_UNAVAILABLE`) does it fall back to a bundled 10-question set; malformed payloads and authorization errors are not masked. The exam engine implements `getQuestion(index)` (id, prompt, choices; no answer key), `submitAnswer(index, choice)` (records the answer and returns `{ isCorrect, correctAnswer, explanation? }`), and `getResults()` (counts correct recorded answers against the bank size). Out-of-range indexes, unknown choices, and use before sync raise `CONTENT_INVALID`.

**Question screen behavior:** selecting a choice calls `submitAnswer`. In Prep mode the screen shows the correct answer and explanation and locks the choice; in Exam mode it records the answer without revealing it. Load and submit errors show the sanitized message.

**Limits to state honestly:** the answer key is held in the app because both package modes include correct answers, so scoring is practice scoring and not a secure assessment; answers and scores are in memory only; downloaded content is not cached for later offline use; the Worker's `Access-Control-Allow-Origin: *` is acceptable only for public sample data.

Tests to teach: the sample JSON maps to 50 models with every answer resolving to a choice; the gateway requests the versioned path; the real Worker handler's Exam and Prep responses are consumable; extra fields are tolerated; malformed payloads, an unsupported `apiVersion`, and a mode mismatch map to `PROTOCOL_ERROR`; the v2 mapper is absorbed without other changes; sync falls back only on `NETWORK_UNAVAILABLE`; the engine gives feedback and scores.

### 10. Configuration and URL Safety

Teach configuration as input that should be validated early. `loadAppConfig` parses a URL and allows HTTPS. Development HTTP is restricted to `localhost`, `127.0.0.1`, and Android emulator host `10.0.2.2`. Development defaults to `http://localhost:8787`; release needs an explicit HTTPS URL. A host can set `globalThis.__EXAM_SIMULATOR_ENV__?.API_BASE_URL` before app startup.

Explain why the emulator's `10.0.2.2` is not the same as the development computer's `localhost` from inside an Android emulator. Configuration errors map to a safe fatal startup screen. Do not use real credentials in example config.

### 11. Errors and Safe Messages

Teach stable error codes versus arbitrary error strings. The app uses codes such as `AUTHORIZATION_REQUIRED`, `NETWORK_UNAVAILABLE`, `PROTOCOL_ERROR`, `CONFIGURATION_INVALID`, `CONTENT_INVALID`, and `UNKNOWN`. `toUserMessage` maps a code to generic display text. Explain why raw network exceptions, URLs, file paths, tokens, and stack traces should not be shown to end users.

### 12. Structured Logging and Redaction

Explain why logging needs useful context but should not leak secrets. The logger redacts top-level keys matching token, password, key, secret, passcode, and authorization, case-insensitively. Note the important limit: nested objects are not recursively scrubbed, so callers must not put credentials into nested fields. Informational logs are development-only.

### 13. HTTP, Fetch, Status Codes, and Timeouts

Teach the `fetch` promise, `Response.ok`, status handling, JSON parsing, abort controllers, timers, and `finally`. The API wrapper maps:

- HTTP 401/403 to `AUTHORIZATION_REQUIRED`.
- HTTP 5xx and transport/timeout failures to `NETWORK_UNAVAILABLE`.
- Other non-success statuses and malformed JSON to `PROTOCOL_ERROR`.

Explain that `request<T>`'s generic return type is not a runtime validator. The client is used only by the question-bank gateway, whose mapper performs the runtime validation; authorization does not use it. Do not teach an unstated backend integration as if it were wired into the app.

### 14. Content Data, JSON Schema, ZIP, and Assets

Teach JSON objects/arrays and the difference between schema version and content version. The ZIP root contains `manifest.json` and `questions.json`; `explanations.json` is required only for Prep; optional media is stored under `assets/`.

The manifest fields are:

```json
{
  "title": "AZ-305 Practice Exam",
  "questionCount": 1,
  "schemaVersion": 1,
  "contentVersion": "1.0.0",
  "examCode": "AZ-305",
  "mode": "exam"
}
```

A question has this shape:

```json
{
  "id": "sample-q01",
  "prompt": "Which option best fits the requirement?",
  "choices": ["Option A", "Option B"],
  "components": [
    { "type": "paragraph", "text": "Read the scenario carefully." }
  ],
  "correctAnswer": "Option A"
}
```

Allowed display component shapes are a closed vocabulary:

```ts
type DisplayComponent =
  | { type: 'paragraph'; text: string }
  | { type: 'list'; items: string[] }
  | { type: 'image'; asset: string; alt: string }
  | { type: 'video'; asset: string; alt: string };
```

There are 2 to 6 unique choices and `correctAnswer` must exactly match one choice. Question IDs are unique. Exam and Prep both contain correct answers for planned offline scoring; only Prep includes explanations. Explain that ZIP is a container format and does not encrypt its contents.

### 15. Runtime Validation and Safe DOM Construction

Explain why content files are untrusted input even when authored by the project team. The Node validator uses AdmZip and validates JSON, required files, versions, question count, IDs, answer choices, component types/properties, asset paths, referenced files, and mode-specific explanations. It runs outside React Native.

The browser preview uses JSZip and DOM nodes. It inserts text through `textContent`, handles the four known component types, and does not import the application's renderer. Explain the security benefit of creating nodes and assigning text rather than injecting package text as HTML. Also explain that the preview is not a substitute for validation and its JSZip script currently comes from a CDN.

### 16. Testing with Jest, Fakes, and Fixtures

Teach the test pyramid at the scale used here: core unit tests, component interaction tests, package-validator tests, Worker handler tests, and a root render smoke test. Explain arrange/act/assert and deterministic tests.

Required test behaviors to show in the book:

- valid and invalid configuration URLs.
- reducer initialization, authorization, sync, and reset.
- stable sanitized error behavior and sensitive-field redaction.
- HTTP status, invalid JSON, and timeout mappings using fake fetch.
- Exam and Prep fixture loading.
- visible screen behavior with mocked service hooks and navigation.
- valid packages pass; malformed and future schema packages fail.
- Worker health/version/authorization responses and invalid request handling.
- Worker question endpoint: 50 questions, no explanations for Exam, explanations for Prep, 400 and 404 cases.
- Gateway and mapper: the 50-question sample maps to models, malformed payloads and unsupported versions fail with `PROTOCOL_ERROR`, and a new API version is absorbed by registering one mapper.
- Exam engine: sync, offline fallback only on `NETWORK_UNAVAILABLE`, answer feedback, and scoring.

Explain that a test double replaces a dependency so a test does not need a live network, database, cloud account, or native emulator. Explain that tests should verify visible behavior and contracts, not implementation details that can change harmlessly.

### 17. Worker and HTTP Boundary

The optional Worker is a small Cloudflare-style fetch handler. It exposes:

- `GET /api/health` returning a fixed M1 health object.
- `GET /api/v1/content/version` returning content version `1.0.0` and schema version `1`.
- `GET /api/v1/questions?examCode=AZ-305&mode=exam|prep` returning the 50-question sample (see the Gateway concept); Exam omits explanations, a missing exam code or invalid mode returns 400, and an unknown exam code returns 404.
- `POST /api/v1/authorize` accepting a minimal email/examCode/mode JSON shape and returning fixed practice-candidate data.
- Other paths return JSON 404.

It validates request shape only and has no database or real authorization. The question endpoint is the only route the app calls (through the gateway); the authorization route is not used. Responses send `Access-Control-Allow-Origin: *` so the browser app on another port can read them, which is acceptable only because the data is public sample content. Teach request parsing and HTTP handlers without presenting the stub as a secure backend.

### 18. Build Tools and Project Configuration

Explain the distinct jobs:

- npm installs dependencies and runs scripts.
- Babel transforms supported language syntax.
- Metro bundles the React Native app.
- Vite serves/bundles the browser app.
- TypeScript checks types without emitting files (`tsc --noEmit`).
- Jest runs tests.
- Gradle builds Android native code.
- CocoaPods/Xcode resolve and build iOS native dependencies.
- Wrangler runs/deploys the Worker.

Explain the job of these project files in the generated code inventory: `package.json`, lockfiles, `tsconfig.json`, `babel.config.js`, `metro.config.js`, `vite.config.ts`, `jest.config.js`, `.eslintrc.js`, `.prettierrc.js`, `app.json`, `index.js`, `web.tsx`, `App.tsx`, `App.web.tsx`, `index.html`, Android Gradle/manifest/Kotlin files, and iOS Podfile/Xcode/Swift/plist/storyboard files. Mark generated build directories, dependency folders, lockfiles, and binary assets as items to understand but not hand-edit.

### 19. Accessibility and Responsive UI

Teach semantic React Native roles and labels: the buttons use `accessibilityRole="button"`; answer choices use radio semantics and a selected state; inputs have labels; errors use alert semantics. Explain that accessibility metadata communicates intent to assistive technologies and that visual styling alone is insufficient. Include manual checks for keyboard navigation, focus order, screen-reader names, readable text, and narrow browser widths.

## Required Project File Map

The final book must include a directory tree and explain the purpose of each listed authored file. It must not claim generated output is authored source.

```text
project-root/
  App.tsx
  App.web.tsx
  index.js
  web.tsx
  index.html
  app.json
  package.json
  package-lock.json
  tsconfig.json
  babel.config.js
  metro.config.js
  vite.config.ts
  jest.config.js
  .eslintrc.js
  .prettierrc.js
  .gitignore
  .watchmanconfig
  Gemfile
  src/
    app/
      navigation/
        NavigationTypes.ts
        RootNavigator.tsx
        RootNavigator.web.tsx
      screens/
        ScreenFrame.tsx
        WelcomeScreen.tsx
        AuthorizationScreen.tsx
        SyncScreen.tsx
        ExamHomeScreen.tsx
        QuestionScreen.tsx
        ResultsScreen.tsx
    core/
      config/AppConfig.ts
      content/FixturePackageLoader.ts
      content/PackageTypes.ts
      content/SampleQuestionBank.ts
      errors/AppError.ts
      gateway/QuestionBankGateway.ts
      gateway/QuestionBankMapper.ts
      logging/RedactingLogger.ts
      models/Question.ts
      services/HttpApiClient.ts
      services/QuestionBankServices.ts
      services/ServiceContainer.tsx
      services/ServiceContracts.ts
      state/AppState.tsx
  tests/
    unit/
      api-client.test.ts
      core.test.ts
      fixture-loader.test.ts
      package-validator.test.js
      question-bank-services.test.ts
      question-gateway.test.ts
      screens.test.tsx
      worker.test.ts
    fixtures/
      valid-exam/
      valid-exam.zip
      valid-prep/
      valid-prep.zip
      malformed/
      malformed.zip
      future-schema/
      future-schema.zip
  __tests__/App.test.tsx
  tools/content-preparation/
    validate-package.js
    preview/index.html
  worker/
    package.json
    package-lock.json
    tsconfig.json
    wrangler.toml
    src/index.ts
    src/data/question-bank.sample.json
  android/
    settings.gradle
    build.gradle
    gradle.properties
    gradlew
    gradlew.bat
    gradle/wrapper/gradle-wrapper.properties
    app/build.gradle
    app/proguard-rules.pro
    app/src/main/AndroidManifest.xml
    app/src/main/java/com/examsimulatorm1/MainActivity.kt
    app/src/main/java/com/examsimulatorm1/MainApplication.kt
    app/src/main/res/values/strings.xml
    app/src/main/res/values/styles.xml
    app/src/main/res/mipmap-*/
  ios/
    Podfile
    ExamSimulatorM1.xcodeproj/
    ExamSimulatorM1/AppDelegate.swift
    ExamSimulatorM1/SceneDelegate.swift
    ExamSimulatorM1/Info.plist
    ExamSimulatorM1/PrivacyInfo.xcprivacy
    ExamSimulatorM1/LaunchScreen.storyboard
    ExamSimulatorM1/Images.xcassets/
  docs/
    architecture.md
    m1-requirements.md
    package-format.md
    package-authoring.md
    known-limitations.md
    platform-status.md
    windows-build.md
    android-build.md
```

In the generated book, include a per-file explanation grouped by folder. Distinguish a file's job from the code inside it. Identify which files a beginner creates in the browser-first path, which files come from the React Native CLI/native scaffold, and which files are optional tools.

## Representative Code Patterns

Use these excerpts to anchor explanations. The full project appendix should expand them into consistent complete files.

### Typed Routes

```ts
export type RootStackParamList = {
  Welcome: undefined;
  Authorization: undefined;
  Sync: undefined;
  ExamHome: undefined;
  Question: undefined;
  Results: undefined;
};
```

Explain that this is a compile-time contract for route names and parameters.

### Reducer Pattern

```ts
function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case 'APP_INITIALIZED':
      return { ...state, initialized: true };
    case 'SYNC_COMPLETED':
      return { ...state, session: action.session, synchronized: true };
    case 'SESSION_RESET':
      return { ...initialAppState, initialized: state.initialized };
    default:
      return state;
  }
}
```

If the book uses an exhaustive union, show the omitted `AUTHORIZED` case too and explain how reducer inputs are narrowed by `action.type`. Never mutate `state`.

### Safe Error Conversion

```ts
export function toAppError(value: unknown): AppError {
  if (value instanceof AppError) return value;
  return new AppError('UNKNOWN');
}
```

Explain the `unknown` input: callers must narrow it before using it. Unknown errors do not become raw user-facing text.

### Content Validation versus Static Types

Show both the compile-time `PackageQuestion` interface and at least one runtime check for parsed JSON. Explain that a cast like `JSON.parse(text) as PackageQuestion[]` does not validate the parsed value. The book may present a small handwritten validator or a clearly named validation library, but it must not claim that a TypeScript cast provides safety.

### Testing a User Action

Show a screen test that renders a screen with fake navigation/services, presses a control, and asserts the observable navigation call or message. Explain why a fake makes the test deterministic.

## Book Structure Required

Generate a book with the following parts. Chapters may be combined only if no learning objective is lost.

### Part I: Foundations

1. Meet the project and define the capstone scope.
2. Install Node/npm, understand terminal location and project root, install dependencies, run the browser preview.
3. Learn JavaScript/TypeScript/JSX and the build pipeline.
4. Understand React Native, React Native Web, web versus native UI primitives, and application entry points.
5. Learn components, JSX, props, children, styles, and accessibility.

### Part II: Build the User Interface

6. Create the shared screen frame and Welcome screen.
7. Add typed navigation and the six routes; explain native stack and browser stack.
8. Build controlled form inputs and local state for Authorization.
9. Add mode selection, validation, error feedback, and duplicate-submit prevention.
10. Build Exam Home and Question interactions; handle selections and boundaries.
11. Build Results and reset navigation; show the score computed from the answers recorded by the exam engine and explain why it is practice scoring only.

### Part III: State and Application Architecture

12. Model types for candidate, mode, content session, actions, and service contracts.
13. Implement a pure reducer and Context provider.
14. Add a composition root and inject services (mocks plus the gateway-backed sync service and exam engine).
15. Explain async effects, promises, cleanup, stale results, and loading/error states.
16. Add configuration validation, stable errors, and logging redaction.
16A. Build the gateway layer: model classes, a versioned mapper with runtime validation, the gateway, the Worker's question endpoint and 50-question sample, and the offline fallback. Show how a new API version is absorbed by one mapper.

### Part IV: Content and Quality

17. Define the package schema and produce small Exam and Prep JSON examples.
18. Explain ZIP archives and implement the fixture loader.
19. Build a Node-only package validator and standalone package preview.
20. Add tests with Jest, Testing Library, fakes, fixtures, and error cases.
21. Explain the optional Worker and HTTP client, including how the gateway calls the Worker's question endpoint while authorization stays mocked.

### Part V: Rebuild and Present

22. Recreate the complete project from an empty folder in a dependable order.
23. Run all checks and debug common setup/type/test errors.
24. Review security, accessibility, responsiveness, and platform status.
25. Present the capstone: architecture diagram, feature demo, test evidence, known limits, and future plan.

### Appendices

- Full directory tree and per-file guide.
- Complete minimum working code listings, grouped by implementation milestones.
- Command reference with working directory for each command.
- Glossary.
- Troubleshooting table.
- Exercise solutions or instructor notes.
- Verification checklist and capstone rubric.

## Hands-On Build Order

The book must use a build order that keeps the project understandable and runnable after each milestone:

1. Create the React Native Community CLI TypeScript project or explain how the given scaffold provides it. Do not use Expo.
2. Verify Node/npm, install dependencies, explain root folder and package scripts.
3. Create the browser entry (`index.html`, `web.tsx`, `App.web.tsx`) and Vite alias so the browser-first path can render.
4. Create shared screen layout and simple screens; run the browser app.
5. Define route names and implement native and browser navigators.
6. Define domain types, service contracts, error codes, and app state.
7. Add the reducer and providers; wire them at app startup.
8. Add the services in the composition root: mocks for authorization and connectivity, and the HTTP client.
9. Add the model classes, the v1 mapper, the gateway, the sync service, and the exam engine; implement authorization, sync, question loading, answer feedback, and result display with pending/error/success states.
10. Add the 50-question sample JSON and the bundled 10-question fallback set; generate a valid fixture ZIP in memory.
11. Add the independent Node package validator, sample package files, ZIP fixtures, and browser preview.
12. Add and run tests after the behaviors they cover are introduced.
13. Add the Worker (health, version, authorize stub, and the question endpoint serving the sample JSON) and typecheck it separately.
14. Explain the generated native folders and platform-specific prerequisites; do not require a native build to complete the browser learning path.
15. Run typecheck, tests, package validation, and browser walkthrough; document actual outcomes.

At each milestone include: files created, concepts taught, code listing, exact command, expected result, likely error and resolution, and a short exercise.

## Test and Acceptance Checklist for the Student Project

The book's completed project should satisfy all of the following:

- [ ] Browser app starts with `npm run web` and displays Welcome.
- [ ] All six named routes can be visited in the expected order.
- [ ] Authorization form has email, exam code, and Exam/Prep selection.
- [ ] Invalid email/empty fields display a safe message without progressing.
- [ ] A valid mock authorization progresses to Sync.
- [ ] Sync loads questions through the gateway (50 when the Worker is running, the bundled 10 when it is unreachable) and updates app state.
- [ ] Exam Home displays session metadata.
- [ ] A question can be selected; advancing without a selection is prevented.
- [ ] The question engine does not return the correct answer with the question; it is returned only after an answer is submitted, and Prep mode alone displays it.
- [ ] Adding a new API version requires only a new mapper (and path) in the gateway layer; the test for this passes without other changes.
- [ ] The end-of-flow score is computed from recorded answers and is described as in-memory practice scoring, not secure assessment.
- [ ] Finish clears session state and returns to Welcome.
- [ ] Exam package excludes explanations; Prep package includes explanations.
- [ ] Package validator accepts valid Exam/Prep and rejects malformed/future-schema fixtures.
- [ ] Tests cover reducer, config, safe errors, redaction, service fakes, package validation, and visible screen behavior.
- [ ] `npm run typecheck` succeeds.
- [ ] Platform status is reported honestly and only from actual builds/tests.

## Common Beginner Mistakes to Explain

Include these errors and their fixes:

- Running commands from the wrong folder: show how to verify the repository root before running npm scripts.
- Forgetting `npm install`: explain that dependencies come from `package.json` and the lockfile.
- Opening a native app without Metro in development: explain the two-terminal workflow.
- Using `localhost` from an Android emulator: explain `10.0.2.2` for the host machine.
- Treating TypeScript interfaces as runtime validation: demonstrate malformed JSON and a runtime check.
- Mutating reducer state: show the incorrect mutation and the immutable return-value correction.
- Forgetting effect cleanup: explain stale async responses and unmounted screens.
- Recreating services during every render: explain provider lifetime and memoization.
- Navigating to a misspelled route: explain the typed route parameter map.
- Hiding all errors or showing raw exceptions: explain stable error codes and safe copy.
- Describing the gateway-backed sync as production content delivery, or a pass-through transform as "encryption": correct those descriptions. Sync reads a public sample API, keeps content in memory only, and falls back to bundled questions when offline.
- Reading API JSON inside a screen or the model: explain that only the mapper may know the wire format.
- Treating the practice score as secure: the answer key is held in the app, and scores are not persisted.
- Treating a ZIP as encrypted storage: clarify the difference between packaging and encryption.
- Claiming Android/iOS/Windows support from source files alone: require actual platform-specific build evidence.

## Glossary Required

Define, in beginner language and with one project example each:

- API, application state, asset, asynchronous operation, Babel, component, composition root, context, controlled input, CSS/StyleSheet, dependency, dependency injection (as a pattern, not a framework), effect, interface, JSX, Metro, mock, navigation stack, promise, prop, reducer, route, runtime validation, schema, service, static type, test double, transpilation/bundling, union type, ZIP archive.

## Final Quality Check for the Generated Book

Before presenting the book, verify that:

1. A reader can follow the document without repository access or another source file.
2. Every required file needed for the hands-on build has a complete listing or a precise scaffold/install instruction.
3. All code listings agree on names, exports, types, route names, paths, and package versions.
4. Any simplified sample is labeled and is not confused with the complete project.
5. The result keeps exactly six routes and the required M1 architectural boundaries.
6. No production credentials, real authorization, encryption, data services, or platform verification claims are invented.
7. The exercises have checkable outcomes and the solutions do not contradict the lessons.
8. The result is organized as a readable learning book with a table of contents, headings, diagrams, examples, and end-of-chapter practice, not merely a generated project README.
