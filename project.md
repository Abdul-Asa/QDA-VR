# QDA-VR: Exploring VR as a Tool for Thematic Analysis

**Author:** Abdullah Shehu
**Institution:** University of Southampton — Electronics and Computer Science, Faculty of Engineering and Physical Sciences
**Degree:** Software Engineering (BEng), First Class
**Supervisors:** Prof. Richard Gomer, Prof. David Millard
**Date:** April 2025

---

## Overview

QDA-VR is a web-based qualitative data analysis platform built as a final-year undergraduate project. It enables researchers to conduct thematic analysis — the process of identifying and interpreting patterns in qualitative data — across three rendering modes: a traditional 2D infinite canvas, a 3D WebGL preview, and fully immersive VR/AR via the WebXR Device API.

Thematic analysis is widely used in academic research but remains time-consuming and space-inefficient. Existing computer-assisted tools (NVivo, Atlas.ti) present steep learning curves and are desktop-bound. QDA-VR explores whether spatial cognition and immersive environments can make the coding and theme development stages more intuitive, while also supporting real-time collaboration between researchers.

---

## Why I'm Proud of This Code

The piece of the codebase I'm most proud of is the **infinite canvas with the qualitative coding editor system** — specifically the interplay between the custom TipTap mark extension (`theme-mark.ts`) and the canvas orchestration hook (`useCanvas.ts`). Together, they solve the central domain problem of the entire project: letting researchers apply, layer, and manage overlapping thematic codes on text within a pannable, zoomable workspace.

### The Problem

In thematic analysis, researchers highlight passages of text and assign one or more "codes" (thematic labels) to them. A single sentence might be tagged with three different codes simultaneously. This creates a fundamental tension with how rich-text editors work — ProseMirror (TipTap's engine) treats marks as exclusive by type. You can't natively have two instances of the same mark on one text range.

On top of that, the codes need to be visually distinguishable when overlapping, manageable as first-class entities (create, rename, recolor, delete, group), and their lifecycle needs to stay consistent across every open editor on the canvas — potentially dozens of text nodes at once.

### The Solution

#### `theme-mark.ts` — A Custom TipTap Mark Extension

Rather than fighting ProseMirror's mark model, I designed the `themeMark` extension to store **parallel arrays** of code IDs and colors on a single mark instance:

```typescript
themeId: string | string[]   // e.g. ["code-1", "code-2", "code-3"]
color:   string | string[]   // e.g. ["#e74c3c", "#3498db", "#2ecc71"]
```

When a researcher applies a second code to already-coded text, the bubble menu reads the existing mark's arrays, appends the new entry, and re-applies the mark as a single unit. The HTML serialization uses `data-theme-id` and `data-theme-color` attributes with JSON encoding, so the document is fully portable.

The rendering adapts to the number of codes: single-code spans get a solid colored underline via `--theme-gradient`, while multi-code spans generate a segmented CSS gradient so each code's color is visually present. This is ~130 lines that solve a genuinely hard problem cleanly within TipTap's extension API.

#### `useCanvas.ts` — The Canvas Orchestration Hook

This ~860-line hook is the central nervous system of the application. It unifies:

- **Viewport management:** zoom-to-cursor (trackpad pinch and scroll wheel), button zoom, pan mode, and fit-to-content reset via bounding-box calculation.
- **Node interaction:** drag handling for both mouse and touch, with scale-aware delta computation so dragging feels correct at any zoom level.
- **Qualitative code lifecycle:** CRUD for codes and code groups, plus the critical `deleteCode` function.
- **Editor registry:** a `Map<nodeId, Editor>` that tracks every active TipTap editor instance on the canvas.

The `deleteCode` function is where the mark extension and the canvas hook meet. When a researcher deletes a code, the function must maintain consistency across every open document:

1. It iterates over every registered editor.
2. For each editor, it walks the ProseMirror document tree via `descendants()`.
3. For each text node carrying a `themeMark`, it checks whether the deleted code's ID is present in the `themeId` array.
4. If found, it splices the ID and its corresponding color out of both arrays.
5. If that was the last code on the span, it removes the mark entirely with `unsetThemeMark()`. Otherwise, it updates the mark in place with the remaining codes.

This ensures referential integrity across all open documents without requiring a full re-render or a centralised annotation store outside the editors.

#### `codegroup-menu.tsx` — The Bubble Menu

The user-facing piece that ties it together is the bubble menu that appears on text selection. It reads existing marks on the selection, shows toggle switches for every available code (grouped and searchable), and handles the array manipulation to add or remove codes from a span while preserving all others. The `handleCodeToggle` function maintains paired `{id, color}` tuples so the arrays never fall out of sync.

### What Makes It Work

What I find satisfying about this design is that it respects the constraints of its dependencies rather than working around them. ProseMirror marks aren't designed for multi-value annotations, but by treating the mark's attributes as arrays and managing the lifecycle at the canvas level, the system gets overlapping codes without monkey-patching the editor or introducing a separate annotation layer. The mark extension stays simple and composable; the complexity lives in the orchestration hook where it can be reasoned about in one place.

---

## Architecture

### Dual Data Flow

The app supports both single-user and multiplayer modes through a strategy pattern in state management:

- **Single-user:** Jotai's `atomWithStorage` persists nodes, viewport state, codes, and code groups to `localStorage`.
- **Multiplayer:** Liveblocks replaces the persistence layer — `useStorage` and `useMutation` provide the same read/write semantics against a shared CRDT document with real-time presence (live cursors, drag indicators).

The `isMultiplayer` prop toggles which data source the `CanvasNodes` component renders from, keeping node and editor components agnostic to the backing store.

### Coordinate System

All positions (nodes, multiplayer cursors) are stored in **canvas coordinates**. The CSS transform `translate(panOffsetX, panOffsetY) scale(scale)` maps canvas space to screen space. For multiplayer cursors, the conversion `(clientX - rect.left - panOffsetX) / scale` normalises screen events to canvas coordinates before broadcasting via Liveblocks presence, and the inverse transform renders remote cursors correctly regardless of each participant's viewport.

### 2D → VR Bridge

The transition from 2D canvas to VR is driven by a state field `viewport.is3D` (`null | "3D" | "VR" | "AR"`). When entering VR:

1. Nodes are laid out on a cylindrical arc via `calculateArcPositions` for comfortable viewing.
2. TipTap JSON content is translated to React Three UIKit primitives via `convertVrJsonToUIKit`, preserving bold text and rendering multi-code theme marks as segmented color underlines in 3D space.
3. Panels billboard toward the user's head using `useFrame` with quaternion math and `MathUtils.damp` for smooth rotation.
4. The `vrFrom` field remembers which immersive mode the user came from, enabling seamless round-trips — exit to edit a node in 2D, then return to VR with state preserved.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14, React 18, TypeScript |
| 3D / VR | Three.js, React Three Fiber, Drei, React Three XR, React Three UIKit |
| Collaboration | Liveblocks (CRDT storage + presence) |
| Text Editor | TipTap (ProseMirror) |
| State | Jotai (`atomWithStorage`) |
| Styling | Tailwind CSS |
| UI Components | Radix UI, @dnd-kit |
| File Processing | Mammoth (DOCX), React-PDF, marked (Markdown), XLSX |

---

## Project Structure

```
QDA-VR/
├── app/                          # Next.js App Router
│   ├── page.tsx                  # Single-user canvas
│   └── [id]/page.tsx             # Multiplayer room
├── components/
│   ├── canvas/
│   │   ├── index.tsx             # Canvas wrapper — switches 2D/VR
│   │   ├── useCanvas.ts          # Core hook — viewport, nodes, codes, editors
│   │   ├── store.ts              # Jotai atoms with localStorage persistence
│   │   ├── types.ts              # Node, Viewport, Code, CodeGroup types
│   │   ├── constants.ts          # Viewport and node dimension constants
│   │   ├── utils.ts              # File reading (mammoth, marked)
│   │   ├── node/                 # TextNode, FileNode components
│   │   ├── text-editor/
│   │   │   ├── extensions/
│   │   │   │   └── theme-mark.ts # Custom multi-code TipTap mark
│   │   │   └── toolbar/
│   │   │       └── codegroup-menu.tsx  # Bubble menu for applying codes
│   │   ├── dialogs/              # CodeManager, NodeManager, Import/Export
│   │   ├── tools/                # Sidebar, Controls
│   │   ├── multiplayer/          # Liveblocks room, cursors, shared nodes
│   │   └── 3d/                   # VRCanvas, NodePanel, environment scenes
│   └── ui/                       # Radix-based UI primitives
├── hooks/                        # useMediaQuery
└── lib/                          # Utility functions
```
