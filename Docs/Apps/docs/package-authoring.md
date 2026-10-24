# Package Authoring

1. Create a ZIP with `manifest.json` and `questions.json` at its root.
2. Keep schema and content versions separate and make every question ID unique.
3. Use only `paragraph`, `list`, `image`, and `video` component types.
4. Put referenced media under `assets/` and remove unreferenced assets.
5. Include `explanations.json` only in Prep packages.
6. Run `npm run validate:package -- tests/fixtures/valid-exam.zip` and the Prep equivalent.
7. Open `tools/content-preparation/preview/index.html` in a browser and select the ZIP.

The preview renderer is standalone and does not import application renderer modules.