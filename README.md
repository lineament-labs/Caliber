# Caliber

An architectural showcase by **Liniment Labs**. Caliber is a local-first, offline-ready time tracking and invoicing engine built with a highly structured, 4-tiered Domain-Driven Design (DDD) layout.

Designed specifically for independent service operators, Caliber enables precision timekeeping, fully configurable multi-tier payment structures (time-slice billing), and browser-native PDF invoice generation—operating with 100% resilience regardless of network connectivity.

## The Architecture (The Liniment Labs Way)

Caliber is engineered inside an Nx Monorepo using an **Open-Core Strategy**, showcasing elite enterprise structural discipline while remaining decoupled from heavy commercial backends. It transitions away from standard "API straight to memory" state models, treating the local device hardware as the absolute source of truth.

```
[ Feature Layer (Thin UI Components) ]
        |
[ Application Layer (NgRx Signal Store) ]
        |
[ Domain Layer (Pure Types & Rate-Slice Engine) ]
        |
[ Infrastructure Layer (Dexie.js Repository)] -- (Instant Offline Read/Write)
        |
(Background Sync Engine)
        |
[ If Connected via Wi-Fi / Tailscale ]
[ Remote Cloud Database (Supabase / Postgres)]
```

### The 4-Tier Breakdown
1. **App / Feature:** Comprises highly optimized, presentational Angular components driven entirely by reactive state.
2. **Application:** Managed via an **NgRx Signal Store** connected natively to local reactive hardware streams.
3. **Domain:** Pure business logic containing no external framework dependencies. Houses the deterministic calculation engine for splitting and computing multi-tier hourly rates.
4. **Infrastructure (Data Access):** Abstracts data interaction behind clear interfaces provided via Injection Tokens. Features a local **Dexie.js (IndexedDB)** repository for instant operations and a decoupled, background delta-sync worker powered by **Supabase**.

## Key Technical Highlights

* **Offline-First Resilience (PWA):** Fully functional without an internet connection. Service Workers cache core assets while data writes instantly to device hardware via IndexedDB.
* **Complex Time-Slice Engine:** A pure domain engine capable of calculating shifting payment brackets (e.g., base rates, late-night premiums, and weekend multipliers) across single time blocks.
* **Client-Side Document Compilation:** Generates print-ready A4 PDF invoices completely on the client CPU using native browser storage blobs for assets.
* **Idempotent Background Sync:** Uses deterministic UUID v4 generation on the client and Unix timestamps to execute smooth, background delta-sync cycles to a self-hosted NAS or cloud provider when connectivity is active.
* **Open-Core Security Blueprint:** Implements a direct `User ➔ Permission` framework, proving strict route-guard capabilities without exposing enterprise-tier RBAC group matrices.

## Workspace Layout

```text
```text
├── apps/
│   └── caliber-pwa/              # Thin Angular PWA shell & service workers
├── libs/
│   └── caliber/
│       ├── data-access/          # Infrastructure: Dexie schemas, Supabase replication loop
│       ├── application/          # Application: NgRx Signal Stores & state orchestration
│       ├── domain/               # Domain: Pure models, time calculators, validation logic
│       └── feature/              # UI: Presentational forms, matrices, and punch-cards
```

