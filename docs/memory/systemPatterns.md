# System Patterns

## Architectural Style
- Primary style: client-heavy SPA backed by a monorepo of shared packages.
- Packaging style: one product app plus reusable libraries published independently.
- Persistence style: local-first with selective backend synchronization.
- Collaboration style: event-driven over WebSocket, with periodic full-state correction.

## Core Patterns

### Local-First Persistence
- Scene elements and app state are written to `localStorage` after a `300ms` debounce.
- Files, library data, and TTD chats use IndexedDB instead of `localStorage` for size and structure reasons.
- The app remains usable offline even when realtime sync or remote persistence is unavailable.

### Encrypted Remote Sharing
- Collaboration rooms use a room ID plus client-side encryption key embedded in the URL hash.
- Shareable links store encrypted/compressed scene payloads remotely and keep the decryption key client-side in the URL hash.
- Consequence: the server stores opaque payloads, but URL handling becomes part of the security model.

### Incremental Sync With Full Resync Safety Net
- `Portal.broadcastScene()` only emits changed syncable elements most of the time.
- The app performs a full scene sync every `20s` to reduce drift from dropped or delayed messages.
- Deleted elements remain syncable for `1 day` to prevent stale peers from reviving removed content.

### Multi-Layer Storage
- `localStorage` stores lightweight scene and app state.
- IndexedDB stores heavier binary/file payloads and persisted chat history.
- Firebase Firestore stores encrypted room scene documents.
- Firebase Storage stores room/share-link image binaries.

### Browser-Tab Version Arbitration
- The app writes `version-dataState` and `version-files` timestamps into `localStorage`.
- Before restoring, the app can detect whether another tab wrote a newer browser state.
- Trade-off: simple implementation, but ordering is timestamp-based and not transactional.

### Feature Surface Separation
- Main editor behavior lives in `packages/excalidraw`.
- Product-specific behaviors such as collaboration, AI dialogs, share dialogs, Sentry, and Plus integration stay in `excalidraw-app`.
- Result: the npm package remains embeddable while the hosted app can add product-specific UI.

## Data Flow Overview

### Local Editing Flow
1. User edits elements in the Excalidraw component.
2. App updates in-memory scene and app state.
3. Debounced save writes scene/app state to `localStorage` and files to IndexedDB.
4. Cross-tab version markers are updated.

### Collaboration Flow
1. User opens a `#room=` URL or starts a session.
2. Client joins a Socket.IO room.
3. Initial scene is loaded from Firebase, then reconciled with local in-memory state.
4. Subsequent updates stream over WebSocket as encrypted event payloads.
5. Scene snapshots are also persisted to Firestore for recovery and late joiners.
6. Files are uploaded to Firebase Storage and peers fetch them on demand.

### Share-Link Flow
1. User exports current scene to backend.
2. Scene JSON is serialized, compressed, encrypted, and POSTed.
3. Image files are uploaded separately to Firebase Storage under a share-link prefix.
4. Returned share ID plus encryption key becomes a `#json=` URL.

### AI Flow
1. User triggers Diagram-to-Code or TTD UI.
2. Client prepares image/text payloads from the current scene or frame.
3. Request is sent to the external AI backend.
4. Responses stream or return generated HTML/code.
5. TTD chat history is stored in IndexedDB.

## Scaling Approach
- Frontend bundle scaling:
  - Vite code-splits locales, Mermaid conversion, and CodeMirror-related chunks.
- Network scaling:
  - incremental sync reduces broadcast size
  - volatile events are used for cursor/viewport/user-state updates
- Storage scaling:
  - binary data is pushed out of `localStorage` into IndexedDB/Firebase Storage
- Operational scaling:
  - static app can be hosted cheaply on Vercel/Nginx while collab/share/AI remain separate services

## Patterns Not Present Or Not Dominant
- Not a microservices repo; external services exist, but this repository is primarily a frontend monorepo.
- No clear CQRS/event-sourcing model; the app uses direct scene state mutation plus reconciliation.
- No multi-tenancy model in the product code examined.

## Related Docs
- Product workflows that these patterns are meant to support: [../product/PRD.md](../product/PRD.md)
- Terminology for scenes, reconciliation, rooms, and syncable elements: [../product/domain-glossary.md](../product/domain-glossary.md)
- Component/module breakdown and infrastructure topology: [../technical/architecture.md](../technical/architecture.md)
- Edge cases and hidden runtime rules behind these patterns: [../technical/behaviors.md](../technical/behaviors.md)