# Package Format

Packages are ZIP files with package-relative paths and these root files:

- `manifest.json`: `title`, `questionCount`, `schemaVersion`, `contentVersion`, `examCode`, and `mode`.
- `questions.json`: unique question identifiers, prompt, answer choices, display components, and correct answer.
- `explanations.json`: required for Prep only and forbidden for Exam.
- `assets/`: optional referenced image/video assets.

The M1 schema version is `1`. Each multiple-choice question has a `choices` array of 2–6 unique strings, and `correctAnswer` must exactly match one choice. The closed display vocabulary is `paragraph` (`text`), `list` (`items`), `image` (`asset`, `alt`), and `video` (`asset`, `alt`). Components are read-only; there are no expressions, loops, input handlers, or conditions. Correct answers are present in both modes for offline scoring in a later module. M1 fixture data is plain JSON inside ZIP files; production encryption belongs to M6.