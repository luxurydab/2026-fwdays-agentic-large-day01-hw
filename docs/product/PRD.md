# Product Requirements Document

## Document Status
- Reverse-engineered from the repository state on 2026-03-25.
- This is not treated as an original source-of-truth business document.

## Project Goal
- Provide a fast, approachable whiteboard and diagramming tool that works well for solo ideation, realtime collaboration, and embeddable use cases.
- Keep the core drawing experience local-first so the product remains useful even when network-backed features are unavailable.
- Serve both direct product users on excalidraw.com and integrators embedding the editor as a React package.

## Target Audience
- Individual creators sketching diagrams and notes quickly.
- Teams collaborating live in a shared room.
- Developers embedding Excalidraw into another React application.
- Power users experimenting with AI-assisted diagram generation and diagram-to-code conversion.
- Educators, facilitators, and workshop participants using a lightweight collaborative whiteboard.

## Core Jobs To Be Done
- Create and edit diagrams with minimal setup.
- Resume work after refresh, reconnect, or offline periods.
- Share a diagram either as a static link or a live collaborative room.
- Import/export drawings and associated assets.
- Reuse the editor as a package inside another product.

## Key Functions

### Core Canvas Editing
- Rich element editing via the Excalidraw editor component.
- Theme and language persistence.
- Welcome, menu, sidebar, and command palette surfaces.

### Local Persistence
- Autosave scene state to browser storage.
- Persist binary files separately from lightweight scene/app state.
- Recover state across reloads when possible.

### Realtime Collaboration
- Start a live room from the app.
- Join a room from a room URL.
- Sync scene updates, cursor state, idle state, and viewport bounds.
- Persist collaborative room state to Firebase for resilience.

### Shareable Links
- Export the current scene to a share backend.
- Generate a recoverable URL with client-held decryption key.
- Upload image files related to the shared scene.

### AI Features
- Diagram-to-Code from frame content.
- Text-to-Diagram chat workflow with persisted chat history.

### Embedding And Distribution
- Publish `@excalidraw/excalidraw` as a reusable package.
- Provide example integrations for Next.js and plain browser usage.

## Key Features
- The product combines local drawing, realtime collaboration, encrypted sharing, AI-assisted workflows, and package embeddability in one browser-based experience.

## Main User Flows

### Flow: Local Drawing
1. User opens the app.
2. Existing local scene is restored if present.
3. User edits the canvas.
4. App autosaves elements/app state/files in the browser.

### Flow: Start Collaboration
1. User opens Share dialog.
2. User starts a live session.
3. App generates room credentials and joins Socket.IO room.
4. User copies or system-shares the room link.
5. Scene continues syncing while also saving to Firebase.

### Flow: Open Shared Scene
1. User opens a `#json=` link.
2. App fetches encrypted payload from backend.
3. App decrypts and restores the scene.
4. User may be prompted before overwriting non-empty local content.

### Flow: AI Diagram-to-Code
1. User selects or works inside a frame.
2. App exports frame content to an image plus extracted text.
3. Payload is sent to the AI backend.
4. Generated HTML/code is returned for display or further usage.

## Technical Limitations
- Must work as a static frontend bundle with external services.
- Must stay usable even if collaboration or AI services are unavailable.
- Must keep decryption keys client-side for shared scenes and rooms.
- Must support modern browsers only; package/browser lists explicitly exclude older browsers.
- Collaboration depends on an external Socket.IO-compatible server that is not implemented in this repository.
- Share-link import/export depends on a separate backend that stores encrypted payloads.
- AI workflows depend on external AI services and are therefore optional extensions rather than guaranteed core functionality.
- Remote scene and file persistence relies on Firebase services and corresponding project configuration.

## Product Constraints
- Local-first behavior is a product requirement, which introduces browser quota, recovery, and synchronization complexity.
- Privacy for shared scenes relies on client-side encryption and correct handling of secrets in URL fragments.

## Non-Functional Requirements
- Fast editing and acceptable network use for multi-user sessions.
- Reasonable offline resilience.
- Safe failure behavior when storage quotas or dynamic import issues occur.
- Published package must remain consumable as a React dependency with CSS import.

## Assumptions
- Assumption: the collaboration server is operated outside this repo and matches the event contract in `Portal.tsx`.
- Assumption: the share-link backend stores encrypted opaque payloads only.
- Assumption: AI features are optional and non-blocking to the core editor experience.
- Assumption: Excalidraw+ links are intentional commercial funnel surfaces, not accidental references.

## Explicitly Observed Trade-Offs
- Local-first storage improves resilience but adds browser quota and sync complexity.
- Client-side encryption improves privacy posture but makes URL handling/security critical.
- Collaboration prioritizes practical reconciliation over strict server authority.
- AI is integrated as a feature extension rather than a core dependency.