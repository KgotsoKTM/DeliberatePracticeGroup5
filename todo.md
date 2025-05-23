# REMCOM MVP – Angular Implementation

Use this **todo.md** as a living checklist. Check each item (`[x]`) once it’s complete and the corresponding tests pass.

---

## 📦 Phase 0 — Bootstrap Workspace

* [ ] **0.1 Create Angular workspace** `ng new remcom-mvp --style=scss --routing --strict`
* [ ] **0.2 Add ESLint & Prettier**

  * [ ] Install deps: `npm i -D eslint prettier eslint-config-prettier eslint-plugin-prettier @angular-eslint/builder`
  * [ ] Generate `.eslintrc.json` extending `@angular-eslint/recommended`
  * [ ] Add `.prettierrc`
* [ ] **0.3 Replace Karma with Jest**

  * [ ] `npm i -D jest jest-preset-angular @types/jest ts-node`
  * [ ] Create `jest.config.ts`
  * [ ] Update `package.json` scripts (`test`, `test:ci`)
* [ ] **0.4 Add Husky pre‑commit** (lints + tests)
* [ ] **0.5 Add Cypress & basic e2e config**
* [ ] **0.6 Initial commit – CI green**

---

## 🗄️ Phase 1 — Domain Models & Mock Data

* [ ] **1.1 Create TS interfaces** in `src/app/models/`

  * [ ] `driver.ts`
  * [ ] `vehicle.ts`
  * [ ] `notice.ts`
  * [ ] `warrant.ts`
* [ ] **1.2 Generate fixture JSON files** in `assets/mock-data/`

  * [ ] `drivers.json` (10)
  * [ ] `vehicles.json` (10)
  * [ ] `notices.json` (15)
  * [ ] `warrants.json` (5)
* [ ] **1.3 Unit‑test fixture counts** (`models.spec.ts`)

---

## 🔄 Phase 2 — Reactive Data Service

* [ ] **2.1 Create `DataService`** in `src/app/core/`

  * [ ] Load fixtures via `HttpClient`
  * [ ] Use `BehaviorSubject` per entity
* [ ] **2.2 Expose observable getters** (`getDrivers$`, `getNotices$`, …)
* [ ] **2.3 Write unit tests** with `HttpClientTestingModule`

---

## 💾 Phase 3 — Persistence & Audit

* [ ] **3.1 LocalStorage state hydrate/save**

  * [ ] `saveState()` compress & persist JSON
  * [ ] Load persisted state on app init
* [ ] **3.2 Create `AuditService`**

  * [ ] Define `AuditEntry` model
  * [ ] Append on every data mutation
* [ ] **3.3 Mutation helpers** in `DataService`

  * [ ] `markNoticePaid()`
  * [ ] `markWarrantExecuted()`
* [ ] **3.4 Unit tests** for round‑trip + audit entry

---

## 🔐 Phase 4 — Authentication

* [ ] **4.1 Create `AuthService`**

  * [ ] Store bcrypt hashes in `assets/users.json`
  * [ ] Session → LocalStorage `remcom-session`
* [ ] **4.2 LoginComponent** (reactive form)
* [ ] **4.3 AuthGuard** protecting secured routes
* [ ] **4.4 Unit tests** (success, failure, redirect)

---

## 🔍 Phase 5 — Search & Results

* [ ] **5.1 Generate `SearchModule`** (lazy)
* [ ] **5.2 SearchComponent** with single input
* [ ] **5.3 SearchResultsComponent**

  * [ ] Two Angular Material tables (driver‑linked, vehicle‑linked)
  * [ ] Status colour pipe
* [ ] **5.4 Cypress e2e** (login → search)

---

## 💰 Phase 6 — Notice “Mark Paid”

* [ ] **6.1 NoticeActionsComponent** button
* [ ] **6.2 Hook to `markNoticePaid()`**
* [ ] **6.3 Snackbar confirmation**
* [ ] **6.4 Unit + Cypress tests**

---

## 🚓 Phase 7 — Warrant “Mark Executed”

* [ ] **7.1 WarrantActionsComponent** button
* [ ] **7.2 Hook to `markWarrantExecuted()`**
* [ ] **7.3 Snackbar confirmation**
* [ ] **7.4 Tests updated**

---

## 🧾 Phase 8 — PDF Generation

* [ ] **8.1 Install pdf‑make**
* [ ] **8.2 PdfService.generateReceipt()**
* [ ] **8.3 Button/link in NoticeActions**
* [ ] **8.4 PdfService.generateWarrant()**
* [ ] **8.5 Unit test blob outputs**

---

## 🛠️ Phase 9 — Admin Module

* [ ] **9.1 Lazy‑loaded `AdminModule`**
* [ ] **9.2 UserManagementComponent** (add/edit/delete)
* [ ] **9.3 AuditViewerComponent** (table + filters)
* [ ] **9.4 Route guards (ADMIN)**
* [ ] **9.5 Unit + Cypress tests**

---

## 📤 Phase 10 — TRAFMAN Export & Reconcile

* [ ] **10.1 TrafmanService.buildExport()** (xmlbuilder2)
* [ ] **10.2 Download button** (ADMIN screen)
* [ ] **10.3 Upload reconcile XML** & diff mismatches
* [ ] **10.4 Unit tests** for XML structure

---

## ⚡ Phase 11 — PWA / Offline Support

* [ ] **11.1 `ng add @angular/pwa`**
* [ ] **11.2 Asset & API caching config**
* [ ] **11.3 IndexedDB stub for cached data**
* [ ] **11.4 Cypress offline mode test**

---

## 📦 Phase 12 — Docker & Deployment

* [ ] **12.1 Dockerfile (nginx‑alpine)**
* [ ] **12.2 GitHub Actions build & push artifact**
* [ ] **12.3 README quick‑start**

---

## ✅ Final QA & Release

* [ ] **Tag v0.1.0** once all tests pass and e2e green
* [ ] **User acceptance walkthrough** with traffic-officer stakeholder
* [ ] **Capture feedback & open tickets** for post‑MVP backlog

---

> Keep this list in sync with commits. If a task changes, update the description and re‑check items accordingly.
