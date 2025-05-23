# REMCOM MVP Specification (spec.md)

---

## 1  Purpose & Objectives

The goal of this Minimum Viable Product (MVP) is to deliver a **laptop‑friendly, web‑only enforcement tool** that lets roadside traffic officials manually process and reconcile **Warrants of Arrest (WoA)** and **outstanding infringement notices** without Automatic Number Plate Recognition (ANPR), handheld devices, or payment gateways.
The system maintains **complete digital records, immutable audit logs, and daily exports compatible with TRAFMAN** while remaining self‑contained (file‑based datastore) for easy pilot deployment.

---

## 2  In‑Scope Features

| # | Feature                    | Key Points                                                                                                                                                           |
| - | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | **Manual Data Entry**      | Officers type a Driver ID *or* Vehicle Reg into a single search box.                                                                                                 |
| 2 | **Centralised Lookup**     | Query all outstanding Notices *and* WoAs linked to that key; visually separate driver‑linked vs vehicle‑linked items.                                                |
| 3 | **Actionable Enforcement** | Buttons: **Mark Paid** (Notices) and **Mark Executed** (WoAs). Actions require a logged‑in session; every change writes an audit‑log row.                            |
| 4 | **PDF Generation**         | One‑click printable **Payment Receipt** or **Executed Warrant Slip** rendered from a shared template using dynamic data.                                             |
| 5 | **TRAFMAN Export**         | Auto‑assemble a **daily XML batch** of all payments & executions for manual upload to TRAFMAN. Filename pattern: `trafman_export_YYYYMMDD.xml`.                      |
| 6 | **Admin & Reconciliation** | Admins view audit logs, create/edit users, re‑open records, and upload TRAFMAN *response* XML files to auto‑reconcile; mismatches are flagged for manual resolution. |

**Out of Scope**: ANPR, handheld apps, real‑time mobile sync, payment gateway, automatic vehicle diversion flow.

---

## 3  Data Model

All tables use **natural primary keys** (highlighted **bold**).

### 3.1 Driver

| Field           | Type        | Format / Example       | Mandatory |
| --------------- | ----------- | ---------------------- | --------- |
| **id\_number**  | string(13)  | `8804125082083`        | ✓         |
| full\_name      | string(120) | `Sibusiso Ndlovu`      | ✓         |
| date\_of\_birth | date        | `1988‑04‑12`           | ✓         |
| licence\_number | string(20)  | `NDL12345678`          | ✓         |
| licence\_expiry | date        | `2030‑04‑11`           | ✓         |
| address         | string(255) | `12 Main St, Pretoria` | ✓         |
| mobile\_phone   | string(20)  | `+27 82 123 4567`      | ✓         |
| email           | string(100) | `s.ndlovu@example.com` | ✓         |

### 3.2 Vehicle

| Field                    | Type                   | Example             | Mandatory |
| ------------------------ | ---------------------- | ------------------- | --------- |
| **registration\_number** | string(12)             | `ABC 123 GP`        | ✓         |
| make                     | string(40)             | `Toyota`            | ✓         |
| model                    | string(40)             | `Hilux`             | ✓         |
| colour                   | string(20)             | `White`             | ✓         |
| year\_manufactured       | int(4)                 | `2019`              | ✓         |
| vin                      | string(17)             | `AHTFR22G9K1234567` | ✓         |
| owner\_id\_number        | FK → Driver.id\_number | `8804125082083`     | ✓         |

### 3.3 Notice / Fine

| Field                 | Type                       | Example             | Mandatory                   |
| --------------------- | -------------------------- | ------------------- | --------------------------- |
| **notice\_number**    | string(20)                 | `N‑2025‑000123`     | ✓                           |
| issue\_date           | date                       | `2025‑05‑20`        | ✓                           |
| violation\_type       | string(60)                 | `Speeding`          | ✓                           |
| amount\_due           | decimal(10,2)              | `750.00`            | ✓                           |
| due\_date             | date                       | `2025‑06‑20`        | ✓                           |
| linked\_driver\_id    | FK Driver                  | `8804125082083`     | ✓                           |
| linked\_vehicle\_reg  | FK Vehicle                 | `ABC 123 GP`        | ✓                           |
| status                | enum(`OUTSTANDING`,`PAID`) | `OUTSTANDING`       | ✓                           |
| payment\_date         | date?                      | `NULL / 2025‑05‑23` | ✓ (NULL allowed until paid) |
| officer\_user         | FK User.username           | `ofc‑kgotso`        | ✓                           |
| execution\_flag       | bool                       | `Y/N`               | ✓                           |
| execution\_date       | date?                      | `NULL / 2025‑05‑23` | ✓                           |
| court\_reference      | string(30)                 | `CPT‑M‑7154‑24`     | ✓                           |
| issuing\_authority    | string(60)                 | `City of Tshwane`   | ✓                           |
| location\_description | string(120)                | `N1 South @ 250 km` | ✓                           |

### 3.4 Warrant of Arrest

| Field                  | Type                           | Example                       | Mandatory |
| ---------------------- | ------------------------------ | ----------------------------- | --------- |
| **warrant\_number**    | string(20)                     | `WOA‑2025‑00456`              | ✓         |
| issue\_date            | date                           | `2025‑05‑12`                  | ✓         |
| issuing\_court         | string(60)                     | `Pretoria Magistrates’ Court` | ✓         |
| linked\_notice\_number | FK Notice                      | `N‑2025‑000123`               | ✓         |
| linked\_driver\_id     | FK Driver                      | `8804125082083`               | ✓         |
| linked\_vehicle\_reg   | FK Vehicle                     | `ABC 123 GP`                  | ✓         |
| amount\_outstanding    | decimal(10,2)                  | `750.00`                      | ✓         |
| status                 | enum(`OUTSTANDING`,`EXECUTED`) | `OUTSTANDING`                 | ✓         |
| execution\_date        | date?                          | `NULL / 2025‑05‑23`           | ✓         |
| officer\_user          | FK User.username               | `ofc‑kgotso`                  | ✓         |
| receipt\_reference     | string(30)                     | `RCPT‑2025‑7890`              | ✓         |
| notes                  | text                           | `Driver arrested on site`     | ✓         |

### 3.5 User (local credential store)

File‑based `users.json`:

```json
[
  {
    "username": "admin",
    "passwordHash": "$2b$12$…",
    "role": "ADMIN"
  },
  {
    "username": "ofc-kgotso",
    "passwordHash": "$2b$12$…",
    "role": "OFFICER"
  }
]
```

---

## 4  Authentication & Authorisation

| Role        | Permissions                                                                                                                                                                     |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **OFFICER** | • Search / view records• Mark **Paid** on Notice• Mark **Executed** on WoA• Generate PDFs• *Cannot* edit users, view raw audit log, or upload reconciliation files              |
| **ADMIN**   | All OFFICER rights **plus**:• Create / edit users (updates `users.json`)• View & filter audit log• Upload reconciliation XML and resolve mismatches• Re‑open or correct records |

*Login flow*: Basic HTML form → bcrypt hash check → signed HTTP‑only session cookie (`Flask‑Login` session).  8‑hour sliding expiry.  No email reset; password changes via ADMIN interface only.

---

## 5  Application Architecture

| Layer                    | Choice                                                                              | Rationale                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Web framework            | **Flask 2.x**                                                                       | Lightweight, file‑based apps are simple; Works with Jinja, WeasyPrint for PDFs.       |
| Templating               | **Jinja2**                                                                          | Native to Flask.                                                                      |
| Persistence (mock)       | **SQLite + SQLAlchemy** *or* CSV/JSON text files                                    | Switchable via config; start with CSV/JSON for street‑side demo, later migrate to DB. |
| PDF engine               | **WeasyPrint**                                                                      | HTML → PDF, no external headless browser needed.                                      |
| XML builder              | **lxml or xml.etree**                                                               | Generate TRAFMAN export.                                                              |
| Front‑end                | **HTMX + Tailwind CSS**                                                             | Minimal JS, fast forms, responsive.                                                   |
| Offline cache (optional) | Service‑worker caches static assets & last lookup; IndexedDB stores recent records. |                                                                                       |

**Folder Layout**

```
remcom/
├── app.py              # Flask entry‑point
├── models.py           # SQLAlchemy models / dataclasses
├── templates/
│   ├── base.html
│   ├── login.html
│   ├── search.html
│   ├── record.html      # Notice & WoA detail
│   └── admin/
├── static/
│   └── tailwind.css
├── pdf_templates/
│   ├── receipt.html
│   └── warrant.html
├── data/
│   ├── drivers.csv
│   ├── vehicles.csv
│   ├── notices.csv
│   ├── warrants.csv
│   ├── users.json
│   └── audit.log
└── exports/
    └── trafman_export_YYYYMMDD.xml
```

---

## 6  Endpoint Specification

| Method | URL                         | Description                 | Auth    |
| ------ | --------------------------- | --------------------------- | ------- |
| GET    | `/login`                    | Login form                  | –       |
| POST   | `/login`                    | Verify creds, set session   | –       |
| GET    | `/logout`                   | Clear session               | Any     |
| GET    | `/search`                   | Search form + results       | OFFICER |
| POST   | `/action/pay`               | Mark Notice paid            | OFFICER |
| POST   | `/action/execute`           | Mark WoA executed           | OFFICER |
| GET    | `/pdf/receipt/<notice_no>`  | Render payment receipt PDF  | OFFICER |
| GET    | `/pdf/warrant/<warrant_no>` | Render executed‑warrant PDF | OFFICER |
| GET    | `/admin/users`              | User management screen      | ADMIN   |
| POST   | `/admin/users`              | Create / edit user          | ADMIN   |
| GET    | `/admin/audit`              | Audit log viewer            | ADMIN   |
| POST   | `/admin/reconcile`          | Upload TRAFMAN response XML | ADMIN   |
| GET    | `/admin/export`             | Trigger daily XML export    | ADMIN   |

---

## 7  PDF Templates

Both PDFs share a branded header and footer:

* **Header**: Municipal logo (left), system title "REMCOM MVP" (center), date/time (right).
* **Body**: Key‑value table of dynamic fields (notice/warrant number, driver, vehicle, officer, amount, action timestamp).
* **Signature block**: Officer signature line + badge number.
* **Footer**: "Generated by REMCOM — Electronic Enforcement Record" + optional QR code encoding the record URL.

### Required Static Assets

* `logo.png` at 300 dpi (embed in repo)
* `seal.png` for court warrants

---

## 8  TRAFMAN XML Export

Daily cron (or manual ADMIN click) builds:

```xml
<TRAFMANExport date="2025-05-23">
  <Payment>
    <NoticeNumber>N-2025-000123</NoticeNumber>
    <DriverID>8804125082083</DriverID>
    <Amount>750.00</Amount>
    <PaidOn>2025-05-23T08:12:55+02:00</PaidOn>
    <Officer>ofc-kgotso</Officer>
  </Payment>
  <Execution>
    <WarrantNumber>WOA-2025-00456</WarrantNumber>
    <ExecutedOn>2025-05-23T08:15:42+02:00</ExecutedOn>
    <Officer>ofc-kgotso</Officer>
  </Execution>
</TRAFMANExport>
```

Upload response XML back into `/admin/reconcile` to match `<Ack status="OK" …>` nodes against local records.

---

## 9  Audit Logging

Append‑only line in `audit.log` (CSV):

```
2025-05-23T08:12:55+02:00,ofc-kgotso,NOTICE,N-2025-000123,PAID,{"status":"OUTSTANDING"→"PAID","payment_date":null→"2025-05-23"}
```

Fields: timestamp, user, entity\_type, entity\_key, action, diff(JSON).

---

## 10  Security Checklist (OWASP Top 10)

* **A01 – Broken Auth**: bcrypt + HTTP‑only cookies; brute‑force lockout after 10 failures.
* **A02 – Cryptographic Failures**: Force HTTPS; store only password hashes.
* **A03 – Injection**: ORM or parameterised queries only.
* **A05 – Security Misconfig**: Disable debug; set `SECURE_HSTS_SECONDS`.
* **A07 – Identity & Access**: Role checks on every endpoint.
* … (full checklist in `/docs/security.md`).

---

## 11  Implementation Roadmap (Step‑by‑Step)

| Step      | Deliverable                                               | Estimated Effort |
| --------- | --------------------------------------------------------- | ---------------- |
| **0**     | Git repo init, set up virtualenv, Tailwind build pipeline | 0.5 d            |
| **1**     | Data model dataclasses + CSV loader/saver                 | 1 d              |
| **2**     | Auth flow (login/logout, bcrypt, roles)                   | 0.5 d            |
| **3**     | Search page + results list                                | 1 d              |
| **4**     | Record detail & action buttons (pay/execute)              | 1 d              |
| **5**     | Audit logging wrapper                                     | 0.5 d            |
| **6**     | PDF templates & generation                                | 1 d              |
| **7**     | Admin – user CRUD & audit viewer                          | 1 d              |
| **8**     | TRAFMAN XML export & upload reconciliation                | 1 d              |
| **9**     | Service‑worker offline cache (optional)                   | 1 d              |
| **10**    | Packaging script (`run_local.sh`, Dockerfile)             | 0.5 d            |
| **11**    | User acceptance test & bug‑fix                            | 1 d              |
| **Total** | \~10 developer‑days                                       |                  |

---

## 12  Sample Test Data

* `data/drivers.csv` — 10 mock drivers with valid SA IDs
* `data/vehicles.csv` — 10 vehicles mapped to drivers
* `data/notices.csv` — 15 notices in mixed states
* `data/warrants.csv` — 5 warrants (3 outstanding, 2 executed)
* `data/users.json` — `admin` / `ofc-demo` hashes created with `python -m bcrypt`.

---

## 13  Future Enhancements (Post‑MVP)

1. Swap CSV persistence for PostgreSQL + REST API layer.
2. Integrate ANPR feed for automatic vehicle hits.
3. Mobile progressive‑web‑app interface usable on handheld scanners.
4. Payment‑gateway integration for card / EFT on the roadside.
5. Real‑time sync & push notifications to central command.

---

**End of spec.md**
