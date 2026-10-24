# Known Limitations

- The provided source text contains OCR errors and incomplete sentences. Where the detailed baseline was clear, it was followed; unresolved details were not inferred.
- API base URL is supplied by the host through `globalThis.__EXAM_SIMULATOR_ENV__.API_BASE_URL`; missing or invalid configuration renders a fatal startup state.
- Sync fetches questions from the Worker sample API through the gateway and falls back to the bundled 10-question set when the API is unreachable. Downloaded content is held in memory only and is not cached for later offline use.
- Recorded answers and scores live in memory for one session and are lost on restart.
- The Worker's CORS header allows any origin because every response is public stub data; restrict it before serving anything non-public.
- Native Windows integration, portable Windows packaging, and Android release signing are not verified in this workspace.
- iOS is NOT VERIFIED because this workspace runs on Windows.
- The package preview currently uses a CDN-hosted JSZip browser script; offline-only preview operation is not yet provided.