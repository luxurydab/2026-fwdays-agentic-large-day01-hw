# Domain Glossary

## Product Terms

### Excalidraw
- The product brand and the core editor experience.
- Also the name of the published React package when used as `@excalidraw/excalidraw`.

### Scene
- The current drawing state as a whole.
- Usually consists of elements, app state, and binary files.

### Element
- A drawable object on the canvas.
- Examples: rectangle, text, arrow, image, frame.

### App State
- UI and editor state that is not itself a drawable element.
- Examples: theme, sidebar state, zoom, selected tool, dialog/error state.

### Binary File
- Non-JSON payload associated with elements, mainly image data.
- Stored locally in IndexedDB and remotely in Firebase Storage.

### Library
- A reusable collection of drawing items/components saved for later insertion.

### Collaboration Room
- A live multi-user editing session represented by a room ID and a room encryption key.

### Shareable Link
- A link to a remotely stored encrypted scene snapshot.
- Uses `#json=<id>,<key>` in the URL hash.

### Room Link
- A collaboration URL using `#room=<roomId>,<roomKey>`.

### Portal
- Internal name for the collaboration networking layer that wraps Socket.IO behavior.

### Syncable Element
- An element eligible to be broadcast/saved remotely.
- Invisible tiny elements are excluded.
- Deleted elements remain syncable for a limited retention window.

### Reconciliation
- The merge process used when applying remote scene updates against local scene state.

### Excalidraw+
- Commercial/hosted product surface referenced from the OSS app.
- Appears in export and AI-related upsell flows.

### TTD
- Text-to-Diagram.
- In this repo it refers to chat-driven diagram generation/persistence functionality.

### Diagram-to-Code
- AI-assisted feature that converts a selected frame and its child elements into generated HTML/code output.

### Frame
- A container-like canvas construct used for grouping content and for AI/export workflows.

## Internal Naming Conventions
- `LocalData`
  - browser persistence layer
- `Portal`
  - socket/collaboration transport wrapper
- `FileManager`
  - file status, fetch, save orchestration
- `appJotaiStore`
  - app-scoped atom store wrapper; project lint rules discourage direct imports from raw `jotai`
- `build:app`
  - build the hosted application
- `build:packages`
  - build the publishable workspace packages

## Abbreviations
- SPA: single-page application
- PWA: progressive web app
- TTD: text-to-diagram
- OSS: open source software
- IDB: IndexedDB
- WS: WebSocket-related naming in collaboration constants

## Ambiguity To Avoid
- "Excalidraw" can mean either the product or the package; specify "app" or "package" when precision matters.
- "Share" can mean either live collaboration or static share-link export; treat them as different workflows.
- "Persistence" is split across browser storage, Firestore, and Firebase Storage; do not assume a single source of truth.