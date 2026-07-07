# ARCHITECTURE.md — Core Technical Decisions

This document outlines the critical architectural guardrails and implementation standards established for **Caliber** to ensure local-first resilience, strict Domain-Driven Design (DDD) boundaries, and predictable data synchronization.

---

## 1. Local-First Media Strategy (Company Logos)

### Context
In traditional API-first architectures, user-uploaded assets (like business logos for invoices) are uploaded directly to a cloud storage bucket (e.g., Supabase Storage), and the resulting public URL string is stored in the database. In an offline-first application, this pattern introduces a critical failure path.

### Decision
Caliber will process, store, and render all media assets strictly locally within the client browser sandbox before any cloud interaction occurs.

### Implementation
1. **Ingestion:** The Feature layer uses a standard HTML file input. The file is intercepted via the browser `FileReader` API and retained as a raw binary `Blob` or converted to a compressed `Uint8Array`.
2. **Persistence:** Dexie.js natively supports storing binary `Blob` data inside IndexedDB. The logo is written directly to the `business_profiles` table as a binary payload.
3. **Rendering:** When generating the HTML invoice view or compiling the client-side PDF, a temporary object URL is created using the Object URL allocation utility. (See CODE_PLACEHOLDER_1)
4. **Cloud Sync Strategy (Open-Core):** To maintain a minimal and zero-cost cloud footprint, binary media streams are excluded from the open-core synchronization tier. The cloud Postgres instance stores metadata only; media state is retained purely on-device.

```ts
const localLogoUrl = URL.createObjectURL(profile.logo_blob);
```
---

## 2. Structural Isolation via Nx Module Boundaries

### Context
In a large enterprise monorepo, maintaining architectural purity over time is challenging. Without automated checks, feature components can easily bypass encapsulation layers and import data-access or infrastructure singletons directly, creating tightly coupled spaghetti code.

### Decision
We enforce strict unidirectional dependency flow across our 4-tiered architecture using **Nx Tag Enforcements** via ESLint.

### Implementation
Configure the `@nx/enforce-module-boundaries` rule inside the root `eslint.config.js` or project configurations using explicit `tags`. (See CODE_PLACEHOLDER_2 for the tag configuration matrix):

* **`type:domain`**: Core models and calculation engines. Has **zero** dependencies.
* **`type:application`**: State orchestration (NgRx Signal Stores). Can only depend on `type:domain`.
* **`type:infrastructure`**: Data repositories (Dexie, Supabase). Can only depend on `type:domain`.
* **`type:feature`**: UI Smart/Presentational components. Can depend on `type:application` and `type:domain`.

```json
[
  {
    "sourceTag": "type:feature",
    "onlyDependOnLibsWithTags": ["type:application", "type:domain"]
  },
  {
    "sourceTag": "type:application",
    "onlyDependOnLibsWithTags": ["type:domain"]
  },
  {
    "sourceTag": "type:infrastructure",
    "onlyDependOnLibsWithTags": ["type:domain"]
  }
]
```
---

## 3. RxJS Idempotency & Race-Condition Guards

### Context
The background synchronization engine fires on two distinct triggers: a deterministic 5-minute interval timer, and immediate browser `online` state window events. If a user reconnects to a network precisely when the interval clock ticks, multiple sync execution loops could fire concurrently, leading to data locks or duplication anomalies.

### Decision
Synchronization cycles must operate as a serialized, sequential queue. Concurrent sync loops are structurally barred from executing.

### Implementation
We leverage the RxJS **`concatMap`** operator within the core synchronization service stream. `concatMap` forces subsequent emissions to wait until the inner asynchronous sequence (the complete Push/Pull database transaction lifecycle) resolves. (See CODE_PLACEHOLDER_3 for the reactive engine stream logic).

```ts
import { inject, Injectable } from '@angular/core';
import { merge, interval, fromEvent, from } from 'rxjs';
import { filter, concatMap } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class SupabaseSyncEngine {
  initSyncLoop() {
    merge(
      interval(5 * 60 * 1000),      // Polling loop
      fromEvent(window, 'online')   // Hardware network trigger
    ).pipe(
      filter(() => navigator.onLine),
      // concatMap guarantees serialization. Next sync waits if current sync is running.
      concatMap(() => from(this.executeDeltaSync()))
    ).subscribe();
  }

  private async executeDeltaSync(): Promise<void> {
    await this.pushLocalChanges();
    await this.pullRemoteChanges();
  }
}
```
