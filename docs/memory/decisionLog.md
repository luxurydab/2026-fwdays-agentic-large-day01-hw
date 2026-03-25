# Decision Log

## ADR-001: Keep A Monorepo With App Plus Publishable Packages

### Context
- The hosted app and the embeddable package share most core editor behavior.
- Maintaining separate repositories would increase drift in types, exports, and build tooling.

### Decision
- Keep the web app, shared packages, examples, and release scripts in a single Yarn workspace monorepo.

### Alternatives Considered
- Separate repo for the npm package.
- Separate repo for the hosted app.

### Consequences
- Positive:
  - shared code stays close to product usage
  - examples can track package changes quickly
- Negative:
  - build and release logic are more complex
  - doc and test coverage must span multiple package boundaries

## ADR-002: Use Local-First Browser Persistence As A Core Behavior

### Context
- The editor must remain useful with unreliable connectivity.
- Scene and file payloads have different storage characteristics.

### Decision
- Persist scene/app state to `localStorage` and larger payloads to IndexedDB, with debounced writes.

### Alternatives Considered
- Remote-only persistence.
- IndexedDB-only persistence.

### Consequences
- Positive:
  - resilient offline editing
  - fast local recovery after refresh
- Negative:
  - quota and browser-storage edge cases
  - cross-tab state arbitration becomes necessary

## ADR-003: Use Socket.IO For Live Sync And Firebase For Persisted Recovery

### Context
- Realtime collaboration needs low-latency event delivery.
- Late joiners and reconnects still need recoverable room state.

### Decision
- Use WebSocket-style event sync for active collaboration and Firestore/Firebase Storage for durable room snapshots and files.

### Alternatives Considered
- Firestore-only collaborative synchronization.
- Custom backend with authoritative scene state only.

### Consequences
- Positive:
  - responsive live updates
  - recoverable room state after disconnects
- Negative:
  - more moving parts
  - correctness depends on reconciliation logic across channels

## ADR-004: Keep Encryption Client-Side For Rooms And Share Links

### Context
- Shared content should not require the backend to understand scene payloads.
- URLs already act as transport for room/share access.

### Decision
- Encrypt room and share-link payloads client-side and store only opaque ciphertext remotely.

### Alternatives Considered
- Server-side encryption with server-visible plaintext.
- Unencrypted remote scene storage.

### Consequences
- Positive:
  - stronger privacy posture for stored payloads
  - backend remains simpler
- Negative:
  - URL fragment handling becomes security sensitive
  - recovery is impossible without the client-held key

## ADR-005: Keep Product-Specific Integrations Out Of The Core Package

### Context
- The published editor package should stay reusable and not force app-specific concerns on consumers.

### Decision
- Keep collaboration orchestration, Sentry, share dialogs, AI endpoints, and Plus hooks in `excalidraw-app` rather than the generic package surface.

### Alternatives Considered
- Expose all app features from `@excalidraw/excalidraw`.
- Maintain a forked app-specific package.

### Consequences
- Positive:
  - cleaner package-consumer experience
  - fewer hosted-product assumptions in the npm package
- Negative:
  - feature wiring spans package and app layers
  - some behaviors are harder to discover without reading both sides

## ADR-006: Use Vite Plus PWA Caching Instead Of A Heavier Legacy Build Stack

### Context
- The app needs fast local iteration and static deployment output.

### Decision
- Use Vite for dev/build and Workbox-based PWA support for runtime caching.

### Alternatives Considered
- Keep or reintroduce a heavier bundler workflow.
- Avoid PWA support entirely.

### Consequences
- Positive:
  - fast development workflow
  - static output suitable for Vercel/Docker
  - offline-friendly asset caching
- Negative:
  - manual chunk and cache rules are brittle
  - service-worker-related failures need explicit filtering and debugging discipline

## Undocumented Behavior Findings

### UBF-001: Deleted Elements Have A Tombstone Retention Window

### What The Code Does
- Collaboration keeps deleted elements syncable for roughly `1 day` instead of dropping them immediately.
- This preserves deletion tombstones so collaborators and restored sessions do not accidentally resurrect removed content.

### File Path
- `excalidraw-app/data/index.ts`
- `excalidraw-app/app_constants.ts`

### File Description
- `isSyncableElement()` keeps deleted elements eligible for sync while their `updated` timestamp remains within `DELETED_ELEMENT_TIMEOUT`.
- The retention window is defined centrally as `24 * 60 * 60 * 1000` milliseconds.

### What Was Documented
- The existing product and architecture documentation described local persistence, collaboration, and recovery behavior at a high level.
- It did not describe the explicit deleted-element retention window or the tombstone-based sync behavior.

### Documentation Gap
- Readers could incorrectly assume that deletion means immediate removal from all persistence and sync paths.

### UBF-002: Local Persistence And Remote Sync Use Different Deletion Semantics

### What The Code Does
- Local scene saves write only non-deleted elements.
- Collaboration logic still propagates recently deleted elements remotely so peers can converge on deletions.

### File Path
- `excalidraw-app/data/LocalData.ts`
- `excalidraw-app/data/index.ts`
- `excalidraw-app/collab/Collab.tsx`

### File Description
- Local autosave serializes `getNonDeletedElements(elements)` into browser storage, so deleted elements are omitted from the local scene snapshot.
- Collaboration uses `getSyncableElements()` for room persistence and live sync, which still includes recently deleted elements during the tombstone window.
- Room initialization also strips deleted elements before first room creation so already-deleted local content is not written into a new collaboration room.

### What Was Documented
- The docs said the app autosaves locally and syncs collaborative state remotely.
- They did not document that the two paths intentionally serialize different element sets.

### Documentation Gap
- Without this distinction, maintainers may treat local storage and remote sync as equivalent data models when they are not.

### UBF-003: Multi-Tab Freshness Uses Timestamp Heuristics, Not Transactional Coordination

### What The Code Does
- Browser-state freshness is tracked with `version-dataState` and `version-files` timestamps in `localStorage`.
- This is a lightweight heuristic for tab freshness rather than a strict concurrency protocol.

### File Path
- `excalidraw-app/app_constants.ts`
- `excalidraw-app/data/tabSync.ts`
- `excalidraw-app/App.tsx`

### File Description
- The storage keys define separate version markers for scene/app state and files.
- `tabSync.ts` keeps per-tab in-memory timestamps and compares them with timestamps stored in `localStorage`.
- The app reloads scene state or files from browser storage only when the persisted timestamp is newer than the current tab's local marker.

### What Was Documented
- The docs described local recovery and cross-session resilience.
- They did not explain that multi-tab arbitration is timestamp-based and intentionally approximate.

### Documentation Gap
- The implementation can be misread as stronger multi-tab consistency than the code actually guarantees.

### UBF-004: Local File Cleanup Is Opportunistic Rather Than Deterministic

### What The Code Does
- Unused local image files are cleaned up only when they are not on the current canvas and old enough by `lastRetrieved` age.
- Cleanup is opportunistic, so stale binaries may linger and some files are retained longer than a reader might expect.

### File Path
- `excalidraw-app/data/LocalData.ts`

### File Description
- `LocalFileManager.clearObsoleteFiles()` scans IndexedDB-backed file entries and deletes only files that are both absent from the current canvas and older than one day by `lastRetrieved`.
- Cleanup is a best-effort sweep, not a strict reference-counted or immediate deletion mechanism.

### What Was Documented
- The docs said files are persisted separately and restored when possible.
- They did not document garbage-collection timing, age thresholds, or reliance on retrieval timestamps.

### Documentation Gap
- Storage behavior looks simpler in the docs than it is in practice, especially when debugging quota issues or file disappearance.

### UBF-005: PWA Install Prompt Capture Depends On Early Listener Registration

### What The Code Does
- The app registers `beforeinstallprompt` outside the normal React lifecycle to catch the browser event early enough.
- Refactoring this later in startup can silently break install-prompt availability.

### File Path
- `excalidraw-app/App.tsx`

### File Description
- The module-level `beforeinstallprompt` listener caches the event in `pwaEvent` before React component initialization.
- The command palette later checks `pwaEvent` and triggers `prompt()` only if that early-captured event is still available.

### What Was Documented
- The decision log covered Vite and PWA caching support.
- It did not record the timing-sensitive startup requirement for the install prompt event.

### Documentation Gap
- A maintainer could innocently move this wiring into a component and regress install behavior without realizing the hidden constraint.

### UBF-006: Self-Embedding Is Not Treated The Same As Generic Iframe Embedding

### What The Code Does
- The app detects same-origin self-embedding and marks `isSelfEmbedding`.
- This creates special-case behavior for internal iframe/export scenarios.

### File Path
- `excalidraw-app/App.tsx`

### File Description
- Startup code compares `document.referrer` origin with the current origin to detect same-origin self-embedding.
- When detected, the app short-circuits normal rendering and shows a dedicated guard screen instead of behaving like a generic third-party embed.

### What Was Documented
- Product docs said the editor supports embeddable use cases.
- They did not distinguish same-origin self-embedding from ordinary third-party embedding.

### Documentation Gap
- The embedding model appears flatter in the docs than in the runtime behavior, which hides integration-specific branches.

### UBF-007: Clipboard Sharing Failures Are Explicit User-Facing Errors

### What The Code Does
- Copying collaboration links to the system clipboard is guarded and surfaces an app error when browser permissions or sandboxing block it.

### File Path
- `excalidraw-app/share/ShareDialog.tsx`
- `packages/excalidraw/clipboard.ts`
- `excalidraw-app/collab/Collab.tsx`

### File Description
- The share dialog calls `copyTextToSystemClipboard(activeRoomLink)` and catches failures to raise a localized collaboration error.
- The shared clipboard helper tries several browser-specific strategies and throws when no copy path succeeds.
- The collaboration layer stores the error message and renders it through an error dialog, making clipboard failure explicitly user-visible.

### What Was Documented
- The documented collaboration flow says the user copies or system-shares the room link.
- It does not mention the explicit failure path or the browser-environment sensitivity of clipboard operations.

### Documentation Gap
- The documented flow reads as guaranteed success, while the code treats clipboard access as a fallible integration point.

### UBF-008: Telemetry Intentionally Omits Several Known Error Classes

### What The Code Does
- Sentry filtering suppresses categories such as Safari pad loop issues, IndexedDB closing issues, stale dynamic import failures, and quota-related errors.
- Missing telemetry for those issues is therefore not evidence that they are absent in production.

### File Path
- `excalidraw-app/sentry.ts`

### File Description
- Sentry is initialized with an `ignoreErrors` list that deliberately drops several noisy or already-understood client-side failures.
- The ignored classes include Safari `window.__pad.performLoop` errors, IndexedDB closing errors, stale dynamic-import fetch failures, quota exceeded errors, and IndexedDB backing-store failures.

### What Was Documented
- The docs mentioned safe failure behavior and Sentry integration at the architecture level.
- They did not document the intentional observability blind spots introduced by error filtering.

### Documentation Gap
- Incident triage can be misleading if maintainers assume Sentry is a complete source of truth for client-side failures.

## Related Docs
- Product rationale and user-facing requirements behind these decisions: [../product/PRD.md](../product/PRD.md)
- Terminology used across ADRs and behavior notes: [../product/domain-glossary.md](../product/domain-glossary.md)
- Structural implementation details for the decisions above: [../technical/architecture.md](../technical/architecture.md)
- Runtime caveats that explain the undocumented behavior findings: [../technical/behaviors.md](../technical/behaviors.md)
- Operational setup impacted by build, service, and environment decisions: [../technical/dev-setup.md](../technical/dev-setup.md)