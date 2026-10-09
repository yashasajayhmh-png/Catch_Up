# Assumptions

The following design decisions and assumptions were made while building the prototype per `catchup-requirements.md` (spec §0 rule 9):

1. **Date Order Fallback:**
   - Per spec F1: if ambiguous, date format defaults to `DMY` (standard for India / WhatsApp India exports).
2. **Web Worker Bundling:**
   - The pipeline runs entirely in an ES module Web Worker (`src/workers/pipeline.worker.ts`), ensuring zero UI lag even with 5,000+ messages.
3. **Network Request Tracking:**
   - Implemented via a transparent runtime wrapper around `window.fetch` and `XMLHttpRequest.prototype.open`, combined with a `PerformanceObserver` for any stray resource loads. The counter shows the true count of network calls since the chat was opened.
4. **Offline & PWA:**
   - Vite PWA (`vite-plugin-pwa`) generates a production service worker caching the application shell (`dist/sw.js`).
5. **Tailwind Styling:**
   - Uses Tailwind v4 with the CSS custom properties specified in §9.2 of the requirements spec, supporting automatic light and dark mode through `prefers-color-scheme`.
6. **Silent Debt Rule Precision:**
   - In 1:1 chats or group chats with mentions, unanswered asks look forward 10 messages or 2 hours. If the user replies with any text (even "ok"), the ask is not debt.
   - For open promises made by the user, if a subsequent message from the user contains closure keywords (`done, sent, shared, etc.`), the promise is cleared from debt.
