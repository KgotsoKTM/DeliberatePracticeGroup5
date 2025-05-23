Angular 17 workspace generated with the Angular CLI

Front-end–only MVP: data is loaded from local JSON files and persisted to browser LocalStorage (or, in unit tests, an in-memory mock).

All server-side-looking behavior (PDF generation, XML export) happens client-side with libraries such as jsPDF, pdf-make, and xmlbuilder2.

Tests use Jest for unit tests and Cypress for e2e (lighter & faster than Karma + Protractor).

Production build is a static bundle served by any web server (NGINX, GitHub Pages, etc.).

1 Blueprint – High-Level Phases (Angular)
Phase	Goal	Key Deliverables
0 Bootstrap	Create Angular workspace & tooling	ng new remcom-mvp, ESLint + Prettier, Husky pre-commit, Jest/Cypress setup
1 Domain Model	TS interfaces + mock data	src/app/models/, JSON fixtures in assets/mock-data/
2 Data Service Layer	CRUD over LocalStorage	DataService, RxJS observables, MockBackendService for tests
3 Auth Core	Login, role guard	AuthService, AuthGuard, /login component, session in LocalStorage
4 Search & Results	Search box + driver/vehicle tables	SearchComponent, SearchResultsComponent, filtering pipes
5 Record Actions	Pay / Execute + audit	NoticeActionService, optimistic UI update, audit stored in LocalStorage
6 PDF Output	Client-side receipts & warrants	PdfService (jsPDF / pdf-make templates)
7 Admin Module	User CRUD & audit viewer	Lazy-loaded AdminModule, route guards, material table
8 TRAFMAN Export	XML download + reconcile upload	TrafmanService, file download/upload, diff highlight
9 PWA / Offline	Angular Service Worker	ng add @angular/pwa, configure asset caching & data hydration
10 Packaging	Docker static site	Dockerfile (nginx-alpine), GitHub Actions deploy artifact

2 Iteration Plan – Medium Chunks
CLI Scaffold & Tooling

Models & Mock JSON

Reactive Data Service (read-only)

Write Support & Audit

Auth (login + guard)

Search UI

Notice → Mark Paid

WoA → Mark Executed

Receipt PDF

Warrant PDF

Admin – Users

Admin – Audit Logs

TRAFMAN Export

TRAFMAN Reconcile

PWA Enablement

Docker Image

Each chunk is ~½ day, ships green tests & running UI.

3 Micro-Steps (Right-Sized Tasks)
3.1 CLI Scaffold & Tooling
ng new remcom-mvp --style=scss --routing

npm i -D eslint prettier jest jest-preset-angular husky

Configure ESLint + Prettier, hook in Husky.

Replace Karma with Jest; add test:ci script.

Add Cypress (+ npm run e2e:ci).

3.2 Models & Mock JSON
Create src/app/models/driver.ts (etc.).

Place fixture JSONs in assets/mock-data/.

Jest test: expect(drivers.length).toBe(10).

3.3 Reactive Data Service
DataService with BehaviorSubject per entity.

Load fixtures via HttpClient (assets/…) then push to subjects.

Expose getDrivers() etc. returning Observable<Driver[]>.

Unit-test that subscribers get data.

3.4 Write Support & Audit
Add saveState() writing compressed JSON to LocalStorage.

Implement AuditService appending rows.

Jest test: state round-trip & audit entry added.

… (continue for each chunk, two-to-three bullet micro-tasks) …
4 LLM Prompts (Angular)
Feed these to your code-gen model sequentially. Each is wrapped in text for copy-paste.

Prompt 0 – Angular Workspace Bootstrap
text
Copy
Edit
You are an expert Angular 17 developer.

GOAL: initialise a clean Angular workspace with modern tooling.

Tasks:
1. Run: ng new remcom-mvp --style=scss --routing --strict
2. Add ESLint & Prettier:
   - npm i -D eslint prettier eslint-config-prettier eslint-plugin-prettier
   - Create .eslintrc.json extending angular-eslint.
3. Replace Karma with Jest:
   - npm i -D jest jest-preset-angular @types/jest ts-node
   - Configure jest.config.ts using preset.
   - Update package.json scripts: "test": "jest", "test:ci": "jest --runInBand".
4. Add Husky pre-commit hook running "npm run lint && npm run test".
5. Commit all files; ensure `npm run test` passes (zero tests).

Generate all file contents and commands required. Output a unified diff.
Prompt 1 – Domain Models & Fixtures
text
Copy
Edit
CONTEXT: REMCOM spec – driver, vehicle, notice, warrant fields.

Tasks:
1. Create TypeScript interfaces in src/app/models/:
   - driver.ts, vehicle.ts, notice.ts, warrant.ts (mandatory fields only).
2. Place sample JSON fixtures in assets/mock-data/ matching models:
   - drivers.json (10 objects), vehicles.json (10), notices.json (15), warrants.json (5).
3. Provide service test:
   - tests/models.spec.ts loads JSON (require) and asserts counts.

Return patch; ensure `npm run test` green.
Prompt 2 – Reactive DataService (Read-Only)
text
Copy
Edit
GOAL: central DataService loading JSON fixtures and exposing RxJS streams.

1. Generate DataService in src/app/core/data.service.ts.
2. In constructor, use HttpClient to GET each fixture under assets/.
3. Store each list in private BehaviorSubject<T[]>.
4. Expose getters: getDrivers$(): Observable<Driver[]> (etc.).
5. Provide unit test data.service.spec.ts:
   - Use HttpClientTestingModule.
   - Flush mocks, expect correct emitted lengths.

Output incremental diff.
Prompt 3 – LocalStorage Persistence & Audit
text
Copy
Edit
ENHANCE DataService with write ability.

1. Add private method saveState() that serialises all subjects to LocalStorage key "remcom-state" (JSON.stringify + LZ-string).
2. On next app start, if LocalStorage key exists, load it INSTEAD of fixtures.
3. Implement AuditService:
   - append(entry: AuditEntry) pushes to BehaviorSubject<AuditEntry[]> and persists.
   - AuditEntry: timestamp, username, entity, key, action, diffJson.
4. Update DataService mutation methods:
   - markNoticePaid(noticeId, officer) updates record, calls AuditService.append.
5. Unit tests:
   - Spy on LocalStorage, assert saveState called.
   - After markNoticePaid, audit list length +1.

Return diff; keep tests passing.
Prompt 4 – Auth Core
text
Copy
Edit
Implement simple local auth.

1. Create AuthService:
   - login(username, pw): compare bcrypt hashes stored in local users.json OR LocalStorage.
   - store session in LocalStorage "remcom-session".
   - currentUser$ BehaviorSubject<User | null>.
2. Create LoginComponent with reactive form (username, password).
3. Add AuthGuard protecting /search route.
4. Route config: /login, /search (lazy loaded later).
5. Unit tests:
   - AuthService.login success/fail flows.
   - AuthGuard redirects unauthenticated to /login.

Return patch.
Prompt 5 – Search Feature
text
Copy
Edit
ADD SEARCH.

1. Generate SearchModule with SearchComponent + SearchResultsComponent.
2. Form with single input; on submit call DataService.search(term): returns driver + vehicle matches.
3. Display two Angular Material tables (Driver-linked, Vehicle-linked).
4. Add filtering pipe for status column colour tags.
5. Cypress e2e: login, search for fixture reg "ABC 123 GP", expect table rows.

Provide diff.
Prompt 6 – Notice Mark-Paid Action
text
Copy
Edit
PAY ACTION.

1. In NoticeActionsComponent (child of SearchResults), add "Mark Paid" button for OUTSTANDING.
2. Button calls DataService.markNoticePaid(id, currentUser).
3. Show snackbar "Payment recorded".
4. Unit test: click button, expect status cell to update → PAID.

Return diff.
Prompt 7 – WoA Mark-Executed Action
text
Copy
Edit
EXECUTE WOA.

Mirror previous step:
1. Add button in WarrantActionsComponent.
2. DataService.markWarrantExecuted.
3. Snackbar confirmation, audit entry.
4. Jest + Cypress tests.

Provide patch.
Prompt 8 – PDF Receipt
text
Copy
Edit
PDF GENERATION.

1. Install pdfmake.
2. PdfService.generateReceipt(notice): returns Blob.
3. Add "Download Receipt" link in NoticeActions.
4. Unit test: blob size > 0, filename correct.
(continue similarly for Warrant PDF, Admin module, TRAFMAN export/reconcile, PWA, Docker)

Usage Flow
Run Prompt 0 → commit.

Run Prompt 1 → commit, npm run test passes.

Continue sequentially.
Every prompt yields working, test-covered code; nothing is orphaned.

This Angular-specific blueprint and prompt suite should let you drive a code-generation LLM (or a human team) in safe, incremental steps—keeping quality high while delivering real functionality at each iteration.