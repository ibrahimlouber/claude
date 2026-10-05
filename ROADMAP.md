# Roadmap — Gamified Argumentation Learning Site

Status: **planning only**. Nothing below has been built. Open decisions are marked **[DECISION]**.
Known problems, data defects and design risks are tracked separately in [`CONCERNS.md`](CONCERNS.md).

## 1. Goal

An interactive website that teaches logical fallacies (and, depending on scope, cognitive biases) through game mechanics: short quizzes, progression, and spaced review.

## 2. Source material

The content comes from a single self-contained HTML reference, *Fallacies & Biases — A Reference*. It embeds one JavaScript array, `ENTRIES`, holding all the data.

| Property | Value |
|---|---|
| Entries | 384 (146 tagged fallacy, 241 tagged bias; 3 tagged both: base-rate fallacy, conjunction fallacy, gambler's fallacy) |
| Tier | 107 `core`, 277 `standard` |
| Evidence status | 332 `established`, 40 `popularization`, 12 `fringe` |
| Categories | 18 (formal/informal fallacy families + bias groups such as "Recall & memory") |
| `alreadyTaught: true` | 15 entries (classic fallacies: ad hominem, straw man, slippery slope, …) |

Fields per entry: `slug`, `name`, `latin`, `types[]`, `categories[]`, `tier`, `alreadyTaught`, `status`, `statusNote`, `etymology`, `definition`, `mechanism`, `example`, `source`, `related[]`.

Note: `example` and `source` are usually strings but are arrays in 15 entries. See CONCERNS.md.

## 3. Open decisions

1. **[DECISION] Scope.** Which entries does the game cover?
   - (a) Fallacies only. This fits "argumentation" most closely.
   - (b) Everything, presented as "reasoning errors".
   - (c) Fallacies first, with biases unlocked later. **Recommended.**
2. **[DECISION] Technical stack.** See §5.
3. **[DECISION] Language.** The source data is in English. Translating 384 entries is a separate content project.
4. **[DECISION] Accounts and leaderboards.** These need a backend. Local-only progress needs none (§5.4).

## 4. Phased plan

### Phase 0 — Data pipeline
- Store the original HTML unchanged in `source/`.
- Write `scripts/extract-entries` to parse `ENTRIES` into `data/entries.json`, so the JSON can be regenerated whenever the source changes.
- Normalise the data: make `example` and `source` always arrays, and either flag or repair dangling `related` slugs.
- Add a validation script that checks the schema, unique slugs, resolvable `related` links and the allowed enum values. Run it in CI.

### Phase 1 — Skeleton
- Folder layout, a README, a static page shell, a data loader and a router (hash routing is enough for static hosting).
- A browse/reference view, so learners can always look up a concept.

### Phase 2 — First game mode: "Spot the fallacy"
- Show an `example` and offer four candidate names.
- Pick distractors from the same category so the choice is hard but fair.
- Feedback shows the `definition` and `mechanism`, plus a link to the entry.

### Phase 3 — Progression
- Points, streaks and a mastery level per category.
- The `core` tier comes first; `standard` entries unlock once a category's core entries are mastered.
- Spaced repetition schedules reviews (§5.5).
- Every `fringe` and `popularization` entry shows an evidence-status label, so the game never presents it as settled science.

### Phase 4 — Further game modes
- **Match definitions:** pair names with definitions in a timed grid.
- **Category sort:** drag each entry into its family.
- **Related chains:** walk `related[]` links ("which concept is closest to X?"). This depends on fixing the broken links first.
- **Formal-logic drills:** for the syllogism and propositional entries (e.g. affirming the consequent), ask whether an argument form is valid.

### Phase 5 — Polish and release
- Accessibility pass: keyboard play, screen readers, colour contrast.
- Mobile layout, plus an optional PWA for offline play.
- Analytics, opt-in only if any.
- Public hosting, which first needs the licensing questions in CONCERNS.md resolved.

## 5. Technical stack options

The pros and cons below are my own engineering assessments, not measured benchmarks.

### 5.1 Front-end framework

| Option | Pros | Cons | Fit |
|---|---|---|---|
| **Plain HTML + CSS + ES modules** (no build) | No tooling. Hosts anywhere as static files. Matches the source file's style. Easy for a beginner to read. | No type checking. Managing state and UI by hand gets messy as game modes multiply. No component reuse without discipline. | Good for a prototype or for 1–2 modes. |
| **Vite + TypeScript (vanilla)** | Types catch data-shape bugs, such as the string-vs-array fields. Fast dev server. Still framework-free. | Needs Node and a build step. Same UI-state problems as plain JS. | Good middle ground. |
| **Vite + Svelte** | Little boilerplate. Reactive state suits game UIs. Small bundles. | Smaller ecosystem than React. One more thing to learn. | **Recommended** if the project will grow past one mode. |
| **Vite + React** (or Preact) | Largest ecosystem: drag-and-drop, animation and testing libraries. Widely known. | More boilerplate. Larger bundle (Preact reduces this). Overkill for a small quiz. | Good if contributors already know React. |
| **Astro** | Static-first. Can pre-render all 384 reference pages for SEO, with interactive "islands" for the games. | Two mental models (static pages + islands). Adds complexity if SEO is not a goal. | Good if a public, searchable reference matters. |
| **Next.js** | Full-stack: API routes for accounts and leaderboards. | Heaviest option. Server hosting pushes you toward particular platforms. Unnecessary without a backend. | Only if a backend is certain. |
| **Phaser** (game engine) | Real game feel: sprites, physics, animation. | Canvas-based, so accessibility and text-heavy content are harder. The content is mostly reading, not action. | Poor fit for core modes. Possibly for one mini-game. |

### 5.2 Language
- **TypeScript.** The dataset has inconsistent field types and 227 dangling links. A typed schema (validated with e.g. Zod) turns these into build-time errors instead of runtime bugs. It costs some setup and learning.
- **JavaScript.** Lower barrier to start. Data bugs show up as runtime failures in the browser.

### 5.3 Styling
- **Plain CSS with custom properties.** The source file already defines a palette and a dark mode this way, so it can be reused directly. No dependency.
- **Tailwind.** Fast to iterate and consistent spacing. Verbose markup, and it needs a build step.
- **Recommendation:** plain CSS variables, reusing the source palette.

### 5.4 Persistence (progress, scores)

| Option | Pros | Cons |
|---|---|---|
| `localStorage` | Trivial. No backend. Private by default. | About 5 MB, per browser and per device. Lost when the user clears site data. No sync between devices. |
| IndexedDB (e.g. via `idb` or Dexie) | Larger, structured storage. Suits spaced-repetition history. | More API complexity. Still per-device. |
| Hosted backend (Supabase, Firebase, Cloudflare D1, …) | Accounts, sync across devices, leaderboards. | Cost, auth, privacy and GDPR obligations, ongoing maintenance, vendor lock-in. |

**Recommendation:** start local-only (localStorage, then IndexedDB if review history grows), behind a small storage interface so a backend can be added later without rewriting the game logic.

### 5.5 Spaced-repetition algorithm
- **Leitner boxes.** Simplest, easy to explain to learners, and good enough for 384 items.
- **SM-2** (the SuperMemo/Anki classic). Well known, small to implement.
- **FSRS.** A newer algorithm used by recent Anki versions. Open-source implementations exist (e.g. `ts-fsrs`). I have not verified this library's current state; check it before adopting.
- **Recommendation:** Leitner for the MVP, behind an interface so it can be swapped later.

### 5.6 Testing and quality
- **Vitest** for unit tests: distractor selection, scoring, scheduling and the data validator.
- **Playwright** for end-to-end tests. Chromium is already available in this environment.
- **ESLint + Prettier** for linting and formatting.
- **GitHub Actions** CI running validation, tests and the build on each push.

### 5.7 Hosting

| Option | Pros | Cons |
|---|---|---|
| GitHub Pages | Free, already next to the repo, simple. | Static only. Public repo on free plans. |
| Netlify / Cloudflare Pages / Vercel | Preview deploy per PR, easy custom domains, optional serverless functions. | Another account. Free-tier limits. |

**Recommendation:** GitHub Pages while the site is static.

## 6. Proposed folder layout (not yet created)

```
/
├── README.md
├── ROADMAP.md
├── CONCERNS.md
├── source/            # original reference HTML, unmodified
├── data/entries.json  # generated, never hand-edited
├── scripts/           # extract + validate
├── src/
│   ├── data/          # loader, schema, normalisation
│   ├── games/         # one module per game mode
│   ├── progress/      # scoring, mastery, scheduling, storage interface
│   └── ui/
└── tests/
```

## 7. Definition of done for the MVP
- Phase 0 validation passes with zero errors, or with explicit, documented exceptions.
- "Spot the fallacy" is playable on desktop and mobile, by keyboard only, in light and dark mode.
- Progress survives a page reload.
- Every entry shown in the game carries its evidence status.
