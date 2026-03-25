# Architecture

## System Breakdown

### Root Workspace
- Purpose:
  - workspace orchestration
  - shared scripts
  - quality gates
  - deployment configuration
- Key files:
  - `package.json`
  - `vitest.config.mts`
  - `Dockerfile`
  - `docker-compose.yml`
  - `vercel.json`
  - `scripts/*`

### Main Application: `excalidraw-app`
- Responsibility:
  - hosted product UI and runtime orchestration
  - app-specific integrations not suitable for the core npm package
- Important modules:
  - `App.tsx`
    - startup path selection
    - import/collab/share-link bootstrapping
    - app-wide UI composition
  - `collab/Collab.tsx`
    - collaboration lifecycle
    - websocket coordination
    - firebase save/load coordination
    - offline state handling
  - `collab/Portal.tsx`
    - socket event bridge
    - encryption before network emit
    - throttled file upload handling
  - `data/LocalData.ts`
    - local scene/file persistence
    - quota handling
    - image cleanup
  - `data/firebase.ts`
    - Firestore scene persistence
    - Firebase Storage file upload/load
  - `data/index.ts`
    - share-link import/export
    - collaboration link helpers
    - syncable element filtering
  - `components/AI.tsx`
    - Diagram-to-Code and TTD feature composition
  - `share/ShareDialog.tsx`
    - collaboration/share-link user entry point
  - `useHandleAppTheme.ts`
    - theme restoration and system-theme behavior
  - `app-language/*`
    - language detection and persisted selection
  - `sentry.ts`
    - runtime telemetry and error filtering

### Packages

#### `packages/excalidraw`
- Public React component package.
- Exports production/development bundles and type-only subpaths.
- Contains the reusable editor engine and UI used both by the app and external consumers.

#### `packages/element`
- Element creation, geometry, ordering, binding, collision, and element-specific logic.

#### `packages/math`
- Shared math/vector utilities used by drawing and geometry code.

#### `packages/common`
- Shared constants, utility helpers, feature flags, and cross-package primitives.

#### `packages/utils`
- Additional support utilities with a separate release cadence.

### Examples
- `examples/with-nextjs`
  - demonstrates client-only SSR-safe integration
- `examples/with-script-in-browser`
  - demonstrates standalone browser usage

## Runtime Topology

```text
Browser
  |
  |-- Excalidraw SPA (React + Vite bundle)
  |     |
  |     |-- Local state: localStorage
  |     |-- Local files/library/TTD: IndexedDB
  |     |-- Realtime sync: Socket.IO server
  |     |-- Scene persistence: Firestore
  |     |-- File persistence: Firebase Storage
  |     |-- Share links: Backend V2 GET/POST API
  |     |-- AI features: AI backend
  |     \-- Error telemetry: Sentry
  |
  \-- Static hosting: Vercel or Nginx container
```

## Infrastructure
- Hosting:
  - Vercel config for production deployment
  - optional Docker image for containerized deployment
- Container path:
  - build stage installs deps and runs `yarn build:app:docker`
  - runtime stage serves `excalidraw-app/build` via Nginx
- Firebase:
  - Firestore stores encrypted scene snapshots under `scenes/<roomId>`
  - Storage stores files under `/files/rooms/<roomId>/...` and `/files/shareLinks/<id>/...`
- External service assumptions:
  - collaboration server compatible with current Socket.IO event contract
  - share backend compatible with Backend V2 endpoints
  - AI backend exposes diagram-to-code and text-to-diagram APIs

## Data Storage Design

### Local Browser Storage
- `localStorage`
  - elements: `excalidraw`
  - app state: `excalidraw-state`
  - collab metadata: `excalidraw-collab`
  - theme: `excalidraw-theme`
  - version markers: `version-dataState`, `version-files`
- IndexedDB
  - `files-db/files-store` for image binaries
  - `excalidraw-library-db/excalidraw-library-store` for library data
  - `excalidraw-ttd-chats-db/excalidraw-ttd-chats-store` for TTD chats

### Remote Storage
- Firestore scene document fields:
  - `sceneVersion`
  - `ciphertext`
  - `iv`
- Firebase Storage object prefixes:
  - `files/rooms/<roomId>`
  - `files/shareLinks/<shareId>`

## Integration Points
- Socket.IO server:
  - room join/init
  - scene update broadcasts
  - volatile cursor and visibility broadcasts
- Share backend:
  - GET scene payload by id
  - POST encrypted scene payload
- AI backend:
  - `/v1/ai/diagram-to-code/generate`
  - `/v1/ai/text-to-diagram/chat-streaming`
- Sentry:
  - release tagging via `VITE_APP_GIT_SHA`
  - environment selection from hostname

## Critical Sequences

### App Startup
1. Polyfills and PWA install listener are registered early.
2. App checks URL for share-link, room link, or external URL import.
3. Local state is restored from browser storage.
4. If external data is requested, user may be prompted before overwrite.
5. If collaboration is active, scene comes from collaboration bootstrap and then reconciles with current editor state.

### Collaboration Save Path
1. Editor change is filtered to syncable elements.
2. Portal encrypts payload and emits scene update over WebSocket.
3. Firebase save path persists reconciled scene snapshot.
4. Files are uploaded separately and image status is updated to `saved` when appropriate.

### Share-Link Export Path
1. Scene is serialized as JSON.
2. Payload is compressed and encrypted client-side.
3. Backend stores opaque scene payload.
4. Files are uploaded to Firebase Storage using the share ID.
5. Client constructs a URL whose hash contains the decryption key.

## Architectural Constraints
- Browser storage is a first-class dependency, not a cache afterthought.
- Collaboration correctness relies on element reconciliation rather than authoritative server-side domain logic.
- Security model depends on client-side encryption and secret material staying in URL fragments.
- The frontend app and published package share internals through aliases during development, so path discipline matters.