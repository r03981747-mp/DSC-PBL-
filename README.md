# Campus Library Intelligence System (CLIS)

CLIS is a working, full-stack campus library management prototype built for a Discrete Structures PBL. It combines a real catalogue and circulation workflow with an interactive Math Lab that demonstrates all six course units against the same seeded library data.

The application is intentionally transparent: the recommendation engine is a rule-based ranking model, and the forecast/optimisation views are deterministic teaching models. There are no trained machine-learning claims. The optional Study Assistant is a teaching aid with a built-in guide fallback; it cannot read private records or perform actions.

## What is included

- **Three enforced roles** — student, librarian and administrator — with API-level guards, not just hidden navigation.
- **Real circulation** — physical copies, availability, loans, due dates, overdue detection, returns, fines and reservation queues.
- **Catalogue management** — create, edit, search, filter, inspect and safely remove titles; copies with history are retired rather than deleted.
- **Explainable recommendations** — five normalised features with fixed weights and an already-read penalty, with a score breakdown in the UI.
- **Decision-support analytics** — eight-week demand history, an explicitly illustrative growth recurrence, inventory projection, per-title risk and a bounded-knapsack purchase plan.
- **Study Assistant** — page-aware starter questions and a read-only teaching drawer on every authenticated page, using the optional OpenAI Responses API or a deterministic built-in study guide.
- **Personal reading list** — students can save titles from the catalogue or book details and manage them in My library; ownership is checked on the server.
- **Role-aware notifications** — students see their own due, overdue and ready-hold alerts; staff see ready holds, overdue follow-up and low-stock alerts derived from live data.
- **Staff CSV exports** — permission-checked catalogue, circulation and analytics downloads with spreadsheet-formula protection and no passwords, tokens or email addresses.
- **Interactive Math Lab** — Logic, Counting, Relations, Recurrences, Graphs and Optimisation, all recalculating from live inputs.
- **Audit history and policy administration** — settings for loan limits, loan periods, fines, holds, low stock and purchase budget.
- **Accessible responsive UI** — desktop and mobile layouts, keyboard-friendly controls, loading, empty and error states.
- **Automated tests** — 41 tests covering shared mathematics, domain rules, HTTP authentication/role/circulation flows, reading lists, notifications, CSV escaping and guided assistant behaviour.

## Tech stack

| Layer | Choice |
| --- | --- |
| Client | React 18, Vite 6, React Router, Lucide icons, custom CSS |
| API | Node.js 24, Express 4 |
| Database | SQLite through the built-in `node:sqlite` module |
| Authentication | scrypt password hashes, random sessions stored in SQLite, Bearer tokens |
| Shared logic | Plain ES modules in `shared/`, imported by both server and client |
| Tests | Node's built-in test runner |

No external database server, ORM or UI framework is required.

```
React + Vite client
        │  /api JSON
        ▼
Express API ── role guards, validation, domain services
        │
        ├── SQLite database (data/clis.db)
        └── shared/ mathematics imported by API and UI
```

## Quick start

### Requirements

- **Node.js 24.x recommended** (tested with Node 24.21). The project uses the built-in `node:sqlite` module, so it needs a Node version that provides that module.
- npm 10 or newer.

### Development

```bash
npm install
npm run seed
npm run dev
```

Open **http://localhost:5173**. Vite proxies `/api` to the Express server on port 4000.

`npm run seed` is optional on the first run: the API seeds a missing database automatically. Run it explicitly before a demo to restore the pristine dataset.

The Study Assistant works without configuration by using its built-in guide. To enable live AI, copy `.env.example` to `.env`, set `OPENAI_API_KEY`, and restart the API. The default model is `gpt-4.1-mini`. See [`docs/AI_SETUP.md`](docs/AI_SETUP.md) for the exact data sent to the provider and the fallback behaviour.

### Production build

```bash
npm run build
npm start
```

Open **http://localhost:4000**. The Express server serves the built site from `dist/`.

### Tests

```bash
npm test
```

This runs the four suites in `tests/` with Node's built-in test runner. The HTTP suite starts a disposable API server on port 4277 with a temporary SQLite database, so it does not touch `data/clis.db`. Tests run with `NODE_ENV=test`, skip the developer's `.env` and force the guided assistant mode, so they make no paid provider calls.

## Demo accounts

All seeded accounts use the password **`demo123`**.

| Email | Role | Seeded situation | Best used to show |
| --- | --- | --- | --- |
| `aarav@campus.edu` | Student | 1 active loan, no overdue loan | Successful borrowing, reservation and recommendations |
| `kabir@campus.edu` | Student | 2 loans, 1 overdue | Rejected loan with an explained overdue restriction |
| `rohan@campus.edu` | Student | 3 of 3 loan slots used | Rejected loan at the borrowing limit |
| `meera@campus.edu` | Student | Ready hold on *Structure and Interpretation of Computer Programs* | Borrowing a held copy |
| `priya@campus.edu` | Librarian | Front-desk account | Returns, hold promotion, issuing, students, analytics |
| `admin@campus.edu` | Administrator | Dr. Ananya Rao | Policy, users, audit log and demo reset |

The seed contains **25 titles**, **49 physical copies**, **13 accounts** (11 students, 1 librarian, 1 administrator), **15 active loans** and **5 open reservations** (4 waiting, 1 ready).

## A short demo route

1. **Aarav → Catalogue → *Artificial Intelligence: A Modern Approach***. Inspect the four-part eligibility check and request the loan. The loan is issued and availability falls.
2. **Aarav → Catalogue → any title → Save**. Open **My library → Reading list** to show the saved title and remove it.
3. **Aarav → Catalogue → *The Silk Roads***. It has no available copy, so join the reservation queue and show the live position.
4. **Kabir → Catalogue → *Concrete Mathematics***. Request the loan: the API rejects it with the overdue reason while the copy remains available.
5. **Meera → My library → Reservations**. Borrow the ready *Structure and Interpretation of Computer Programs* hold.
6. **Aarav → top-bar bell**. Show the role-aware due/overdue and ready-hold alerts; staff accounts see operational alerts instead.
7. **Priya → Circulation**. Find Ishaan Bhatt's *Introduction to Algorithms* loan and return it. Zoya Khan's waiting reservation is promoted to ready.
8. **Zoya → My library → Reservations**. Borrow the promoted copy.
9. **Priya → Circulation**. As a staff issue, select Tara Sethi and *Concrete Mathematics*, inspect the shared eligibility preview, and issue the loan.
10. **Priya → Circulation → Export CSV**. Download the circulation export; the catalogue and analytics exports are available on their own pages.
11. **Aarav → For you**. Expand the score breakdown for a recommendation, then open **Study assistant** and ask how the score is calculated.
12. **Admin → Settings**. Change the borrowing limit, save, then inspect **Audit**. Finish with **Reset demo data** to restore the seed and sign everyone out.

The full minute-by-minute version, including all six Math Lab units and evidence capture, is in [`docs/DEMO_WALKTHROUGH.md`](docs/DEMO_WALKTHROUGH.md).

## Role capabilities

| Capability | Student | Librarian | Administrator |
| --- | :---: | :---: | :---: |
| Search the catalogue | Yes | Yes | Yes |
| Borrow / reserve a copy | Own account | — | — |
| View own loans, holds and history | Yes | — | — |
| Save and manage a personal reading list | Yes | — | — |
| Receive role-aware in-app notifications | Own alerts | Operational alerts | Operational alerts |
| Export catalogue, circulation and analytics CSV | — | Yes | Yes |
| Use the Study Assistant | Yes | Yes | Yes |
| Issue and return copies | — | Yes | Yes |
| Manage catalogue and copies | — | Yes | Yes |
| Manage students and reservations | — | Yes | Yes |
| View collection analytics | — | Yes | Yes |
| Manage accounts, policy and audit | — | — | Yes |
| Use every Math Lab unit | Yes | Yes | Yes |

The API independently verifies the session and role for every protected route. A student cannot issue a loan by calling the API directly, and a librarian cannot edit accounts or policy.

## Math Lab

Open **Math Lab** in the sidebar, or use the direct tab links from the Project page. Every panel recalculates immediately from the controls.

| Unit | Live model | What to try |
| --- | --- | --- |
| **1. Logic** | `E = A ∧ C ∧ ¬O ∧ L` for active, copy-available, not-overdue and below-limit | Select Kabir and an available title; switch to the 16-row truth table and the equivalent forms |
| **2. Counting** | Permutations, combinations, subsets, induction and pigeonhole | Change the reading-list size and subject collection; change the induction rows and pigeonhole inputs |
| **3. Relations** | Reading-order partial order, same-department equivalence relation and function classification | Inspect the relation matrix and Hasse covers; switch to equivalence classes and functions |
| **4. Recurrences** | First-order, second-order and inventory recurrences | Set `a = 1.2` to show divergence; change the second-order coefficients; make stock run out |
| **5. Graphs** | Library walkway graph, BFS, Tarjan cut vertices/bridges/blocks, weighted book similarity and Huffman coding | Route *Main entrance* to *Archive room*; change the similarity threshold and focus title; rebuild a Huffman code |
| **6. Optimisation** | Convex quadratic loss, gradient descent, Hessian, stability bound and Lagrange budget allocation | Set `η = 0.21` to cross the stability bound; change the budget, costs and weights; inspect the first-order conditions |

The same calculation modules live in `shared/` and are used by the backend where the result must be consistent, for example eligibility and analytics.

## Recommendation model

The recommender is not machine learning. It is a deterministic scoring function:

```
score(book) = 0.32·category + 0.22·author + 0.18·tags
            + 0.18·popularity + 0.10·availability
            − 0.30·alreadyRead
```

Each feature is normalised to `[0, 1]`. The UI exposes the feature value, weight and contribution for every candidate, so a result can be explained rather than accepted as a black box. Books already on active loan are excluded, and returned titles receive the already-read penalty.

## Study Assistant

The **Study assistant** button in the top bar opens a page-aware drawer on every authenticated page. Its starter questions change with the current route and role. Replies are labelled **Live AI** when the server has an OpenAI key and **Built-in study guide** otherwise; a provider failure degrades to the guide with **Study guide · AI unavailable**.

The assistant is deliberately read-only. It has no tools, cannot mutate library records and is not given database access. The server sends only the current question, at most six recent messages and a static page identifier/summary; passwords, tokens, account details, loans, holds, reading-list entries and unrelated records are never included. The API key stays server-side, requests use the Responses API with `store: false`, and question/answer content is not logged.

Without a key, the deterministic guide in `shared/assistant.js` covers CLIS workflows and all six Math Lab units, so the feature remains useful offline. Setup, limits and data handling are documented in [`docs/AI_SETUP.md`](docs/AI_SETUP.md).

## Reading list and notifications

Students can save a catalogue title from its card or details drawer. The `reading_list` table stores one row per user and book, enforced by a unique constraint; add/remove routes are student-only and always use the signed-in user's id. **My library → Reading list** shows live availability and opens the normal book details drawer. Saving is a bookmark, not a reservation.

The top-bar bell is computed from current data rather than sent by email or SMS:

- **Students:** their own loans due within three days, overdue loans and ready holds.
- **Staff:** ready holds awaiting pickup, overdue-loan follow-up and titles at or below the low-stock threshold.

Every alert links to the page where the action can be completed. Student responses never contain another borrower's records.

## Staff CSV exports

Librarians and administrators can download CSV files from the catalogue, circulation and analytics pages. The API re-checks the role for each export rather than relying on button visibility.

| Export | Main columns |
| --- | --- |
| Catalogue | Id, title, author, category, ISBN, year, shelf, copy counts, active loans, waiting holds, price, tags |
| Circulation | Loan id, title, author, copy code, borrower identifier, department, issue/due/return dates, status, fine, overdue days |
| Analytics | KPI summary, observed weekly demand, forecast inputs/projection, popular titles, low-stock and queue tables, method assumptions |

Values containing commas, quotes or line breaks are quoted; text beginning with a spreadsheet formula character is prefixed with an apostrophe. Files carry a UTF-8 BOM and CRLF line endings for Excel compatibility. Password hashes, session tokens, email addresses and unrelated account data are omitted.


## Analytics and forecasting boundary

The analytics page uses real seeded borrow history for its eight-week demand buckets and per-title activity. The future projection is an **illustrative recurrence**, not a trained forecast or a live institutional feed:

```
D(t) = D(t−1) × (1 + g)
I(t+1) = max(0, I(t) + restock − D(t))
```

Growth, restock and budget are user-supplied scenario inputs. The purchase plan is a bounded 0/1 knapsack over recent demand, waiting reservations, catalogue prices and a default maximum of two copies per title. The method and assumptions are displayed on the page.

## Project structure

```
.
├── server/
│   ├── auth.js          scrypt hashing, sessions and role middleware
│   ├── assistant.js     optional OpenAI Responses API call and safe fallback
│   ├── csv.js           formula-safe CSV writer and download headers
│   ├── db.js            schema, settings, transactions, audit helpers
│   ├── domain.js        catalogue, circulation, reservations, reading list, notifications, analytics
│   ├── index.js         Express routes, exports and static production serving
│   └── seed.js          deterministic demo data and reset
├── shared/
│   ├── assistant.js     page context, starter questions and guided answers
│   ├── logic.js         eligibility, truth tables, inference
│   ├── counting.js      combinatorics, induction, pigeonhole
│   ├── relations.js     matrices, closures, equivalence, functions
│   ├── recurrence.js    first/second-order and inventory sequences
│   ├── graph.js         BFS, shortest path, blocks, similarity, Huffman
│   ├── optimization.js  gradient descent, Hessian, Lagrange
│   ├── forecast.js      demand buckets, projection, purchase plan
│   └── recommend.js     transparent ranking model
├── src/
│   ├── pages/           role workspaces and Math Lab tabs
│   ├── components/      layout, assistant, notifications, catalogue and UI primitives
│   ├── context/         session and toast state
│   ├── styles/          custom responsive design system
│   └── api.js           JSON request helper, downloads and endpoint map
├── tests/
│   ├── shared.test.js   mathematics and formatting
│   ├── domain.test.js   circulation and recommendation rules
│   ├── features.test.js assistant guide, reading list, notifications and CSV escaping
│   └── api.test.js      HTTP auth, roles, validation, new features and flows
├── docs/
│   ├── AI_SETUP.md
│   ├── PROJECT_REPORT.md
│   ├── DEMO_WALKTHROUGH.md
│   └── TEAM_CONTRIBUTIONS.md
├── data/clis.db         generated SQLite demo database
└── dist/                generated production frontend
```

## Configuration and reset

| Variable | Default | Purpose |
| --- | --- | --- |
| `PORT` | `4000` | Express API / production site port |
| `DB_PATH` | `data/clis.db` | SQLite database location; `:memory:` is supported for tests |
| `OPENAI_API_KEY` | empty | Optional server-side key for the live Study Assistant; empty uses the built-in guide |
| `OPENAI_MODEL` | `gpt-4.1-mini` | Responses API model used when a key is configured |

Copy `.env.example` to `.env` to override defaults. `.env` is git-ignored; the API loads it with Node's built-in `process.loadEnvFile()`. Tests skip `.env` and force guided mode.

To reset manually:

```bash
# Stop the API first if using the CLI.
npm run seed
```

In a running app, sign in as `admin@campus.edu` and use **Settings → Reset demo data**. This reseeds inside the current server process, invalidates all sessions and returns the UI to the login screen.

The database is safe to delete while the server is stopped; the API will create and seed a fresh one on the next start.

## Verified checks

- `npm test`: **41 / 41 passing** — shared math, domain rules, feature helpers and HTTP integration.
- `npm run build`: production build succeeds. Vite reports only a large-chunk advisory for the charting/Math Lab bundle.
- API smoke checks: guided assistant, role-aware notifications, reading-list ownership/idempotency and authenticated CSV exports return the expected status codes; student export requests return `403`.
- Browser walkthrough at **1440×1000** and **390×844**: no page-level horizontal overflow, no console exceptions.
- End-to-end UI checks: successful loan, overdue rejection, waitlist join, ready-hold borrowing, staff issue, return with queue promotion, promoted-hold borrowing, reading-list save/remove, notification drawer, CSV download, policy change, audit entry and demo reset.

## Known limitations

This is an academic prototype, not a production library system.

- Demo passwords are public and there is no production identity provider, HTTPS termination, rate limiting or email delivery.
- Borrower-history privacy, backup/restore and multi-library tenancy are out of scope.
- No payment gateway, barcode scanner hardware or external catalogue integration.
- Demand growth, restock and purchase planning are illustrative teaching models, not live forecasts.
- The recommender uses fixed, inspectable weights; it does not train on user behaviour.
- Notifications are computed in-app only; there is no email or SMS delivery.
- The Study Assistant cannot read a learner's account or perform actions. Live AI depends on an external provider and a server-side key; the built-in guide is static teaching material.
- Dates in the seeded history are generated relative to the day the database is seeded, so due-date labels change between resets.

For the report outline, formula details, testing evidence and limitations discussion, see [`docs/PROJECT_REPORT.md`](docs/PROJECT_REPORT.md). For the presentation route, see [`docs/DEMO_WALKTHROUGH.md`](docs/DEMO_WALKTHROUGH.md). For assistant setup and data handling, see [`docs/AI_SETUP.md`](docs/AI_SETUP.md). For the editable contribution record, see [`docs/TEAM_CONTRIBUTIONS.md`](docs/TEAM_CONTRIBUTIONS.md).
#   D S C - P B L -  
 