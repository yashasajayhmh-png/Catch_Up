# Catch-Up — Midnight Glass Triage

> A local-first, privacy-focused "What Did I Miss?" chat triage app built for speed, calm clarity, and zero-compromise UX.

Built with **React + TypeScript (strict) + Vite**, **Tailwind CSS**, **Framer Motion**, **lucide-react**, and **Zustand**.

---

## 🎨 Design Direction: Midnight Glass Triage

- **Dark-First Aurora Backdrop**: Deep near-black background (`#0B0D12`) paired with an animated, drifting 3-blob aurora gradient (violet `#7C5CFF`, cyan `#22D3EE`, magenta `#F472B6`) at 12% opacity. Respects `prefers-reduced-motion`.
- **Glassmorphism**: Translucent cards (`bg-white/5` with `backdrop-blur-md` and `1px border white/10`), rounded 20px corners, subtle cursor-following radial glow on mouse movement, and 2px hover lift.
- **Semantic Color Coding**:
  - **Now**: Coral / Amber (`#F97316`)
  - **Debt**: Violet (`#8B5CF6`)
  - **Decisions**: Electric Blue (`#3B82F6`)
  - **FYI**: Slate (`#64748B`)
  *(Every item displays an icon + text badge in addition to color.)*
- **Typography**: System UI font stack with tight tracking on headings and large bold count-up numbers.
- **Light Theme Support**: Toggleable via header with automatic `prefers-color-scheme` support on warm off-white (`#F7F7F5`).
- **Accessibility & Touch Targets**: All interactive targets are $\ge 44\text{px}$, contrast $\ge 4.5:1$, visible focus rings, and full `aria-label` tags.

---

## 📱 Screens & User Journey

1. **Import (`/import`)**:
   - Hero title *"What did I miss?"* with a vibrant gradient effect.
   - Pulsing dashed drop-zone for `.txt` and `.zip` WhatsApp exports.
   - *"Choose file"* and *"Try sample chat"* actions.
   - Collapsible guide: *"How to export a WhatsApp chat"*.
   - Privacy status pill: lock icon + *"Processed on your device"* + live *"Network requests: 0"*.
2. **Who are you? (`/identity`)**:
   - Selectable list of participants with initials avatars, names, message counts, and visual activity mini-bars.
   - Active identity glows with violet highlight.
   - Comma-separated aliases input for nicknames and phone number formats.
3. **Processing (`/processing`)**:
   - Animated 4-step stepper: *Reading messages* $\rightarrow$ *Filtering noise* $\rightarrow$ *Finding tasks & decisions* $\rightarrow$ *Ranking*.
   - Determinate progress bar with real-time updates.
   - *"Skip AI, use fast mode"* option.
4. **Triage Feed (`/triage`)**:
   - **Sticky Glass Header**: Chat name, date context, active identity avatar (tap to change), settings shortcut ⚙, and light/dark theme toggle.
   - **Hero Stat Strip**: Animated count-up counter (e.g. *412 messages $\rightarrow$ 9 need you*).
   - **Segmented Tabs**: Sliding spring pill indicator with live counter badges for **Now**, **Debt**, **Decisions**, and **FYI**.
   - **Card Anatomy**: 4px left accent stripe, type chip, right-aligned due date badge, 2-line summary, up to 3 reason chips (*"Tagged you"*, *"Possibly unanswered"*), sender, timestamp, and action buttons (*Done*, *Dismiss*, *"Wrong?"*).
   - **Low Confidence (<0.6)**: Rendered with dashed border and *"Possibly"* prefix.
   - **Debt Tab**: Dismissible banner (*"Things you were asked or promised that look unfinished. We may be wrong: you may have replied elsewhere."*). Grouped into *"Asked of you"* and *"You promised"*, with *"Open for N days"* badges.
   - **Decisions Tab**: Vertical timeline layout.
   - **FYI Tab**: Compact one-line rows + collapsible *"Hidden as noise: N messages"*.
   - **Interactions**:
     - Mobile swipe right to mark *Done*, swipe left to *Dismiss*.
     - Desktop keyboard shortcuts: `j`/`k` to navigate, `d` for Done, `x` for Dismiss, `Enter` to open source viewer.
     - 5-second Undo toast for restored cards.
     - Collapsed *"Cleared"* section for completed/dismissed items.
     - Confetti-free *"all clear"* celebration when Now is zero.
5. **Source Viewer (`/source`)**:
   - Mobile: 75% height draggable bottom sheet.
   - Desktop ($\ge 1024\text{px}$): Sticky right-hand side pane in a 2-pane triage layout.
   - Highlights the target message in chat-bubble style with 3 preceding and 3 succeeding messages for context. Underlines detected due text.
   - *"Why this ranks here"* breakdown with 4 animated progress bars (*Urgency*, *Directness*, *Unresolved*, *Sender*) and origin indicator (*Rules*, *AI*, *Both*).
6. **Settings (`/settings`)**:
   - Switch identity and aliases.
   - Toggle processing mode (*Rules only* vs *Rules + AI*).
   - Privacy inspection panel (requests count, offline ready badge).
   - Export feedback JSON or clear local data.

---

## 🔌 Backend Contract Mapping (`src/api/client.ts`)

All external network operations are centralized in `src/api/client.ts`. When a backend server is running at `BASE_URL`, requests are sent over HTTP. If the server is offline or unreachable, `client.ts` automatically falls back to the high-performance local on-device Web Worker / NLP pipeline.

### Field-to-UI Mapping

| Backend Contract Field | Type | UI Component / Placement | Notes / Fallback |
| :--- | :--- | :--- | :--- |
| `id` | `string` | Unique React key, card actions | Required |
| `msgId` | `number` | Source viewer target message look-up | Anchors the 3-before / 3-after context |
| `type` | `"task" \| "question" \| "decision" \| "info"` | Top-left card badge (Icon + Text) | Fallback: `"info"` |
| `summary` | `string` | Card title / summary text (max 2 lines) | Fallback: message excerpt |
| `addressedTo` | `string[] \| "all" \| "unknown"` | Directness score & reason chip (`"Tagged you"`) | Used to calculate directness bar |
| `dueText` | `string \| null` | Right-aligned due badge on card, underlined in source | Hidden if `null` |
| `dueTs` | `number \| null` | Relative due calculation, red badge if `< Date.now()` | Hidden if `null` |
| `confidence` | `number` (0..1) | Dashed border styling & `"Possibly "` prefix if `< 0.6` | Fallback: `0.5` |
| `origin` | `"rule" \| "llm" \| "both"` | Source viewer ranking badge (*"Rules"*, *"AI"*, *"Both"*) | Fallback: `"rule"` |
| `score` | `number` (0..100) | Ranking order in tab lists | Fallback: `0` |
| `reasons` | `string[]` | Up to 3 chips on card (e.g. *"Tagged you"*, *"Needs reply"*) | Fallback: `[]` |
| `tab` | `"now" \| "debt" \| "decisions" \| "fyi"` | Tab categorization and filter pill counts | Fallback: `"fyi"` |
| `debtKind` | `"unanswered" \| "open_promise" \| null` | Sub-grouping on Debt tab (*"Asked of you"* vs *"You promised"*) | Fallback: `null` |
| `status` | `"open" \| "done" \| "dismissed" \| "wrong"` | Active card list vs collapsed *"Cleared"* drawer | Fallback: `"open"` |
| `sender` | `string` | Card metadata line & source bubble header | Fallback: `"Unknown"` `// TODO(contract)` |
| `ts` | `number` | Card metadata relative timestamp & *"Open for N days"* | Fallback: `Date.now()` `// TODO(contract)` |
| `scoreBreakdown` | `{ urgency, directness, unresolved, sender }` | Source viewer 4 animated ranking bars (0..100) | Computed from item heuristics if omitted |

### Contract Gaps & `// TODO(contract)` Reference

The following fields in `src/api/client.ts` contain `// TODO(contract)` comments to ensure easy adaptation to your backend schema:

1. **`BASE_URL` (`line 93`)**:
   - Defaults to `http://localhost:3000`. Set this to your backend server port.
2. **`sender` & `ts` on `BackendItem` (`lines 116-117`)**:
   - If your backend items do not embed the sender name or timestamp directly, `client.ts` gracefully falls back to `"Unknown"` and `Date.now()`, or resolves them from the source message context.
3. **`scoreBreakdown` on `BackendItem` (`lines 118-125`)**:
   - If your backend does not provide individual component scores (`urgency`, `directness`, `unresolved`, `sender`), the UI uses rule-based defaults calculated from `reasons` and `confidence`.
4. **`POST /api/item/status` (`line 320`)**:
   - Receives `{ id, status: "done" | "dismissed" | "wrong" }`.

---

## ⚡ Strictly Compliant Microcopy

All system microcopy strictly adheres to requirements:
- `"Possibly unanswered"` and `"Possibly still open"` (debt is never stated as absolute fact).
- Footer: `"Urgency measured as of {date}. Dates are resolved from when each message was sent."`
- Fast mode notice: `"Fast mode: AI is unavailable on this device. Results use rules only."`
- Bad file error: `"This doesn't look like a WhatsApp export."`

---

## 🚀 Running & Testing

### Development Server
```bash
npm run dev
```
Accessible at `http://localhost:5173`.

### TypeScript Strict & Production Build
```bash
npm run build
```
Builds client assets into `dist/` with zero TypeScript errors.

### Automated Test Suite
```bash
npm test
```
Runs 26 unit tests verifying WhatsApp parsing formats, Hinglish date detection, Silent Debt tracking, and ranking scoring.
