# SPEC.md — Caliber Product Specification

## 1. Authentication (Auth-Lite Strategy)
Provides resilient user management that caches identity locally to allow seamless offline routing and permission enforcement.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| AUTH-01 | Secure remote login via Supabase Auth (Email/Passcode). | Infrastructure | [ ] |
| AUTH-02 | Local singleton caching of JWT, User details, and Permissions array in Dexie `auth_session` table. | Infrastructure | [ ] |
| AUTH-03 | Local-first Angular Route Guards that read permissions from active Dexie session when offline. | Application / App | [ ] |
| AUTH-04 | Session hydration check on application boot to automatically renew or validate tokens if network is active. | Application | [ ] |
| AUTH-05 | Non-commercial restriction notice banner injected when structural premium matrices (RBAC) are bypassed. | Feature | [ ] |

---

## 2. Time Tracking & Rate-Slice Engine
Manages highly granular clock-in events and passes them through a deterministic domain engine to calculate tiered pricing schedules.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| TIME-01 | Reactive stopwatch interface for tracking live "Clock-In" sessions on mobile and desktop views. | Feature | [ ] |
| TIME-02 | Rule configuration grid allowing creation of arbitrary hourly rate adjustments (e.g., Night Premium, Weekend Overtime). | Feature | [ ] |
| TIME-03 | Pure Domain calculation function to segment a single `TimeLog` across overlapping time-bracket rules. | Domain | [ ] |
| TIME-04 | Immediate write of `TimeLog` data to Dexie with a `pending` state and immediate client-side mutation timestamps. | Infrastructure | [ ] |
| TIME-05 | Automatic background calculation of current shift earnings updated via an active RxJS interval stream. | Application | [ ] |

---

## 3. Invoice Management
Groups unbilled client hours into structured, professional business accounts with independent tracking numbers.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| INV-01 | Multi-select interface displaying all unbilled, completed `TimeLog` records filtered by date bounds. | Feature | [ ] |
| INV-02 | Deterministic generation of sequential `invoice_number` schemes matching commercial formats (e.g., `INV-YYYY-XXX`). | Domain | [ ] |
| INV-03 | Client profile registry mapping billing addresses, email contacts, and structural reference flags. | Infrastructure | [ ] |
| INV-04 | State modifier hooks to pivot invoice lifecycles cleanly between `Draft` ➔ `Issued` ➔ `Paid`. | Application | [ ] |

---

## 4. PDF Printing (Browser Native)
Compiles complete business summaries into beautifully formatted A4 papers without invoking external cloud rendering infrastructure.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| PDF-01 | Binary upload interface in settings to cache a custom company graphic directly into Dexie as a local `Blob`. | Feature | [ ] |
| PDF-02 | Client-side print pipeline utilizing `jspdf` to draw typographic tables, totals, and metadata within browser memory. | Feature / App | [ ] |
| PDF-03 | Layout styling layer using Tailwind CSS `@media print` rules to optimize standard web layouts for physical A4 routing. | Feature | [ ] |
| PDF-04 | Offline file attachment capability allowing immediate sharing of locally generated invoice files to native mobile targets. | Feature | [ ] |

---

## 5. Historical Record Keeping
Retains past business units, modifications, and analytics logs safely inside persistent IndexedDB registers.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| HIST-01 | Reactive archive timeline rendering past invoices with robust text searching and multi-field data sorting filters. | Feature | [ ] |
| HIST-02 | Business analytics dashboard computing running tax allocations, gross earnings, and total hours logged per unit. | Application | [ ] |
| HIST-03 | Request browser verification step on application bootstrap to upgrade IndexedDB storage to explicit `persistent` status. | Infrastructure | [ ] |
| HIST-04 | Soft-delete protocol enforcing record mutations flag as `deleted` rather than hard stripping them out of the database array. | Domain / Infra | [ ] |

---

## 6. Background Sync Engine
Coordinates communication pathways between local storage tables and the remote server using an idempotent protocol.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| SYNC-01 | Background replication controller listening on a 5-minute interval clock loop and browser `online` window events. | Infrastructure | [ ] |
| SYNC-02 | Upstream Push phase reading `pending` items from Dexie and executing high-speed Postgres `ON CONFLICT` upserts. | Infrastructure | [ ] |
| SYNC-03 | Downstream Pull phase comparing local `last_successful_sync_time` logs against remote table mutation entries. | Infrastructure | [ ] |
| SYNC-04 | Client-side conflict engine managing basic state reconciliation using a "Server Wins Unless Local Is Dirty" standard. | Domain | [ ] |
| SYNC-05 | Real-time state signal markers (`⏳` / `✅`) projecting the synchronization status of individual records inside UI cards. | Feature | [ ] |

---

## 7. Backup & Recovery (Fail-Safe Procedures)
Insulates business operations against catastrophic physical data drops, browser data wipes, or network partition errors.

| ID | Feature Description | Architecture Tier | Status |
| :--- | :--- | :--- | :---: |
| BACK-01 | Local runtime exception handler catching unexpected script faults and caching diagnostic information inside a `client_logs` table. | Infrastructure | [ ] |
| BACK-02 | Universal export button providing a raw encrypted or plaintext JSON string download of the entire Dexie store schema array. | Feature | [ ] |
| BACK-03 | Manual recovery intake portal allowing complete local restoration of the system from a previous JSON data backup file. | Feature | [ ] |
| BACK-04 | Device connectivity validation layer preventing sync loop initialization if Tailscale mesh networks are inaccessible. | Infrastructure | [ ] |
