# JobTracker-AI — Skills & Project Guide

## Project Purpose
A premium, local-first Kanban-style job application tracker for QA/testing professionals. Users manage applications across a 6-column pipeline (Wishlist → Applied → Follow-up → Interview → Offer → Rejected) with drag-and-drop, analytics, AI readiness scoring, and full IndexedDB persistence. No backend or authentication — everything runs in the browser.

## Tech Stack
| Layer | Choice |
|---|---|
| Framework | React 19 + TypeScript 6 |
| Build | Vite 8 |
| Styling | Tailwind CSS 4 (utility-first, no CSS modules) |
| Drag & Drop | @dnd-kit/core + @dnd-kit/sortable |
| Database | IndexedDB via `idb` library |
| Charts | Pure CSS (no chart library — status bars, monthly bars, funnel) |
| Testing | Vitest + React Testing Library (unit), Playwright (E2E) |
| Icons | Emoji unicode only (no icon library dependency) |
| Deployment | Static — `npm run build` → deploy `dist/` to any static host |

## Folder Structure
```
JobTracker-AI/
├── src/
│   ├── __tests__/          # Vitest unit tests
│   │   ├── types.test.ts   # 57 tests — formatDate, daysAgo, readiness score, etc.
│   │   ├── db.test.ts      # 9 tests — IndexedDB CRUD + migration
│   │   ├── components.test.tsx  # 39 tests — Dashboard, Charts, Goal, etc.
│   │   ├── sampleData.test.ts   # 11 tests — 25 demo jobs
│   │   └── importExport.test.ts # 9 tests — import/export round-trip
│   ├── components/         # All UI components
│   │   ├── Dashboard.tsx   # 13 gradient stat cards
│   │   ├── KanbanColumn.tsx # Droppable column with gradient header
│   │   ├── JobCard.tsx     # Draggable card with left color strip
│   │   ├── JobModal.tsx    # Add/Edit form with interview tracking
│   │   ├── JobDetailModal.tsx # Full detail side panel
│   │   ├── ConfirmModal.tsx   # Delete confirmation
│   │   ├── Charts.tsx      # Status distribution, monthly, funnel
│   │   ├── ActivityFeed.tsx  # Recent activity log
│   │   ├── ApplicationGoalTracker.tsx # Editable monthly goal
│   │   ├── ApplicationHealth.tsx  # Fresh/Follow-up/Attention badge
│   │   ├── ReadinessScore.tsx   # 0-100 score bar
│   │   ├── CompanyAvatar.tsx   # Colored circle with first letter
│   │   ├── ResumeAnalytics.tsx # Per-resume performance
│   │   └── ColumnContainer.tsx # Wrapper (currently secondary)
│   ├── App.tsx             # Main app — state, DnD, routing (none), modals
│   ├── db.ts               # IndexedDB layer (openDB, CRUD, migration)
│   ├── types.ts            # All TS types + pure helper functions
│   ├── sampleData.ts       # 25 realistic demo jobs generator
│   ├── index.css           # Tailwind imports, glassmorphism, gradients, animations
│   ├── main.tsx            # Entry point
│   └── test-setup.ts       # Vitest setup (fake-indexeddb)
├── e2e/                    # Playwright E2E tests
│   └── app.spec.ts
├── public/                 # Static assets
└── dist/                   # Build output (gitignored)
```

## Coding Conventions
- **No semicolons** — Use ASI (automatic semicolon insertion)
- **No comments** — Code should be self-documenting; comments only where logic isn't self-evident
- **JSDoc** — Never add JSDoc/docstrings
- **Functional components** — Always use `function Component()` not `const Component = () =>`
- **Props interface** — Define `interface Props {}` in the same file, export if reused
- **Hooks** — Use `useState`, `useEffect`, `useCallback`, `useMemo` from React; no external state library
- **Async** — Use `async/await` over `.then()`
- **1-letter vars** — Acceptable in loops/maps (`j` for job, `s` for status, `i` for index)
- **No barrel exports** — Import directly from file path, not `index.ts`
- **No default exports** — Use named exports everywhere (except `export default function App()` which is required by React)

## Component Architecture
- **App.tsx** is the single state manager — holds all jobs, modal states, filters
- Components receive data via props, never call `db.ts` directly
- **JobCard** uses `useSortable` from @dnd-kit — card identity is `job.id`
- **KanbanColumn** uses `useDroppable` — column identity is `status` string
- Drag handlers in App.tsx: `handleDragStart` saves active job, `handleDragEnd` updates status + timeline + activities
- Modals are conditionally rendered in App.tsx based on state flags

## State Management
- **Single source of truth**: `jobs: Job[]` in App.tsx
- **No Context API, no Redux** — all state passes through App.tsx
- Derived state via `useMemo`: `filteredJobs`, `columnJobs`, `resumeOptions`, `allActivities`
- Changes flow: user action → state update → `updateJob()` → IndexedDB persist
- `loadJobs` runs on mount; writes to IndexedDB after every mutation

## IndexedDB Usage
- **Database**: `job-tracker` (version 3)
- **Store**: `jobs` (keyPath: `id`)
- **Indexes**: `status`, `priority`, `dateApplied`, `company`
- **Migration**: `upgrade()` handler handles v1→v2→v3 transitions
- **`migrateJob()`** — Ensures old records get defaults for new fields
- **`addJob()`** uses `db.add()` (fails on duplicate)
- **`importJob()`** uses `db.put()` (upsert — safe for re-import)
- **`updateJob()`** uses `db.put()`
- **Mocking**: `fake-indexeddb` auto-patches global in tests

## Testing Strategy
- **Vitest** for unit/component tests — fast, no browser needed
- **125 tests** across 5 files — always run `npm test` before commit
- Component tests use `@testing-library/react` — query by visible text, not test IDs
- DB tests use `fake-indexeddb/auto` — no mocking needed
- Playwright tests are separate — run with `npx playwright test`
- Test file naming: `*.test.ts` or `*.test.tsx` in `src/__tests__/`
- E2E tests live in `e2e/` directory

## UI Guidelines
- **Glassmorphism** — `.glass` class with `backdrop-filter: blur(16px)` for modals, columns, widgets
- **Gradients** — Dashboard cards use `grad-*` classes (e.g. `grad-blue`, `grad-purple`)
- **Animations** — `animate-fade-in` for staggered card entrance (use `style={{ animationDelay }}`)
- **Cards** — `card-hover` class for lift effect on hover
- **Status colors** — blue (wishlist), yellow (applied), purple (follow-up), orange (interview), green (offer), red (rejected)
- **Priority colors** — red (urgent), orange (high), blue (medium), gray (low)
- **Dark mode** — `.dark` class on `<html>`, persisted in localStorage. Use `dark:` prefix
- **Spacing** — `p-4` cards, `gap-3` between columns, `px-6` for page padding
- **Typography** — Inter font, `text-xs` for labels, `text-sm` for body, `text-lg` for headings
- **Responsive** — Grids use `grid-cols-1 md:grid-cols-3 lg:grid-cols-8` patterns

## Naming Conventions
- **Files**: PascalCase for components (`JobCard.tsx`), camelCase for utilities (`sampleData.ts`, `db.ts`)
- **Interfaces**: PascalCase (`Job`, `NewJob`, `TimelineEntry`)
- **Types**: PascalCase with type suffix if needed (`JobStatus`, `Priority`, `ApplicationSource`)
- **Constants**: UPPER_SNAKE_CASE (`COLUMNS`, `DB_NAME`, `STORE_NAME`, `STORAGE_KEY`)
- **Functions**: camelCase (`formatDate`, `daysAgo`, `getApplicationHealth`)
- **Event handlers**: `handle*` prefix (`handleDragEnd`, `handleAddJob`, `handleExport`)
- **State setters**: `set*` prefix (standard React convention)
- **Props interface**: `Props` in each component file

## Git Workflow
```bash
git add -A && git commit -m "message" -m "Co-authored-by: CommandCodeBot <noreply@commandcode.ai>"
git push origin main
```
- `main` branch only — no feature branches, no PRs needed
- Always run `npm run build` and `npm test` before pushing
- Commit messages: present tense, descriptive ("Fix import to use upsert" not "Fixed bug")

## Vercel Deployment
1. `npm run build` generates `dist/`
2. Deploy `dist/` as static site — no server config needed
3. Vercel auto-detects Vite — just set build command to `npm run build` and output dir to `dist`
4. SPA routing: configure `vercel.json` with `{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }` if needed

## Common Commands
```bash
npm run dev        # Start dev server on localhost:5173
npm run build      # TypeScript check + Vite production build
npm run test       # Run Vitest (125 tests)
npm test           # Same as npm run test
npx vitest run     # Run Vitest (non-watch mode)
npx vitest         # Run Vitest (watch mode)
npx playwright test # Run Playwright E2E tests
npm run preview    # Preview production build locally
npm run lint       # ESLint check
```

## Future Roadmap
- [ ] Multi-user sync via backend (optional)
- [ ] Email notifications for follow-ups
- [ ] Calendar integration for interviews
- [ ] Resume upload & parsing
- [ ] Advanced charts (recharts)
- [ ] Mobile responsive optimization
- [ ] PWA support for offline-first
- [ ] AI-powered job description parsing
- [ ] Auto-apply tracking

## Things to Never Change
- **IndexedDB as the only data store** — Never add a backend for authentication
- **6-column Kanban structure** — The pipeline column set is fixed
- **`idb` library** — The async IndexedDB wrapper stays (not raw IndexedDB)
- **@dnd-kit** — The drag-and-drop library stays (not react-beautiful-dnd, not native HTML5)
- **Tailwind CSS** — No CSS-in-JS, no Sass, no CSS modules
- **No routing** — Single-page app, no React Router (modals handle all navigation)
- **Sample data auto-load** — First visit always gets 25 demo jobs if DB is empty
- **Dark mode** — Must persist via localStorage with `.dark` class on `<html>`
- **25 demo jobs** — The sample data must keep exactly 25 jobs with 5/6/4/4/3/3 column distribution
- **Readiness score** — The 0–100 scoring algorithm with suggestions must remain
- **Timeline + Activity log** — Every status change must record both timeline entry and activity entry
