# HomeBuilder 3D - Project Context

> Created: 2026-04-08
> Status: active (Ready for Development)
> Last Updated: 2026-04-08

## Overview

- **Name**: HomeBuilder 3D
- **Path**: C:/Repos/HomeBuilder 3D
- **Description**: Web-based 3D house designer with drag-drop functionality for furniture placement
- **Repository**: false
- **Author**: Architect Agent (Architecture Document dated 2026-04-06)

## Technologies

- React 19 + TypeScript
- Vite
- React Three Fiber (3D Engine)
- Zustand (State Management)
- .NET 9 (Backend - planned)
- PostgreSQL (Database - planned)

## Confirmed Requirements

| Decision | Value | Notes |
|----------|-------|-------|
| Grid resolution | 0.5m | Snapping increment |
| House layouts (MVP) | 2 | Fixed floor plans |
| Catalog size | 100 objects | For MVP launch |
| Object variants | Color variants | Per-object customization |
| CAD export | DWG | Industry standard |
| Blueprint export | PDF | Printable format |
| Asset format | glTF/GLB | With Draco compression |
| Target browsers | Chrome, Safari, Firefox, Edge | All major browsers |
| Performance metrics | 60fps only | No additional benchmarks |
| Anonymous storage | Local only | No server save |
| Design sharing | URL hash | Stateless sharing |
| Max objects | 500 | Performance target |

## Functional Requirements

### Core Features (MVP - Must Have)

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-01 | Fixed square/grid-based house structure | P0 | pending |
| FR-02 | 3D viewport with orbit camera | P0 | pending |
| FR-03 | Object catalog UI | P0 | pending |
| FR-04 | Drag-drop object placement | P0 | pending |
| FR-05 | Object repositioning | P0 | pending |
| FR-06 | Object rotation | P0 | pending |
| FR-07 | Grid snapping (0.5m) | P0 | pending |
| FR-08 | Object selection (click to select) | P0 | pending |
| FR-09 | Basic materials/colors | P0 | pending |

### Advanced Features (Post-MVP)

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-10 | User authentication | P1 | future |
| FR-11 | Multiple house layouts | P1 | pending |
| FR-12 | Object variants (colors/sizes) | P1 | pending |
| FR-13 | Undo/redo system | P1 | pending |
| FR-14 | CAD/blueprint export | P1 | future |
| FR-15 | Design sharing via URL | P1 | pending |
| FR-16 | Object deletion | P1 | pending |
| FR-17 | Save design to database | P1 | pending |
| FR-18 | Load saved design | P1 | pending |

### Non-Functional Requirements

- Performance: Target 60 FPS, max 500 objects
- Availability: Web-based, browser runtime
- Security: Client-side only for MVP
- Target devices: Desktop-only for MVP

## Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Framework | React 19 + TypeScript | Project requirement |
| Build Tool | Vite | Modern, fast builds |
| 3D Engine | React Three Fiber | React integration, ecosystem |
| 3D Helpers | @react-three/drei | Ready-made components |
| State Management | Zustand | Simple, performant, TypeScript |
| Styling | Tailwind CSS | Speed, consistency |
| Backend | .NET 9 (planned) | Project requirement |
| Database | PostgreSQL (planned) | Project requirement |
| API Style | REST | Simplicity for MVP |
| Asset CDN | Cloudflare R2 or AWS S3 (planned) | Cost-effective, fast |
| 3D Format | glTF/GLB | Standard, compressible |

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    React Frontend                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │
│  │  Catalog │  │   View   │  │    Zustand Store     │  │
│  │   Panel  │  │  Controls│  │  - selectedObject    │  │
│  └────┬─────┘  └────┬─────┘  │  - objects[]          │  │
│       │             │        │  - mode (edit/view)   │  │
│       ▼             ▼        └───────────┬──────────┘  │
│  ┌─────────────────────────────────────────────┐       │
│  │           React Three Fiber Canvas          │       │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │       │
│  │  │ Scene   │  │ Lights  │  │ Orbit       │  │       │
│  │  │ Objects │  │         │  │ Controls    │  │       │
│  │  └─────────┘  └─────────┘  └─────────────┘  │       │
│  └─────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼ REST API (Future)
┌─────────────────────────────────────────────────────────┐
│                    .NET 9 Backend (Planned)             │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │
│  │ Catalog  │  │ Design  │  │   PostgreSQL DB      │  │
│  │ Service  │  │ Service │  │   - designs         │  │
│  └──────────┘  └──────────┘  │   - catalog          │  │
│                               └──────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Folder Structure

```
src/
├── domain/                    # Business logic layer
│   ├── entities/              # Core domain entities
│   │   └── FurnitureObject.ts
│   ├── valueObjects/          # Value objects
│   │   └── Position.ts
│   ├── repositories/          # Repository interfaces
│   │   └── IDesignRepository.ts
│   └── services/              # Domain services
│       └── GridService.ts
├── application/               # Application layer
│   ├── useCases/              # Use case implementations
│   │   ├── PlaceObject.ts
│   │   └── MoveObject.ts
│   ├── interfaces/           # Application interfaces
│   │   └── IObjectService.ts
│   ├── dtos/                 # Data transfer objects
│   │   └── FurnitureObjectDTO.ts
│   └── services/             # Application services
│       └── SceneService.ts
├── infrastructure/            # Infrastructure layer
│   ├── persistence/          # Data persistence
│   │   └── LocalStorageAdapter.ts
│   ├── external/             # External services
│   │   └── AssetLoader.ts
│   └── config/               # Configuration
│       └── gridConfig.ts
├── presentation/             # Presentation layer
│   ├── controllers/          # React controllers
│   │   └── EditorController.tsx
│   └── handlers/             # Event handlers
│       └── DragHandler.ts
├── 3d/                       # 3D-specific code
│   ├── components/           # R3F components
│   │   ├── Scene.tsx
│   │   ├── FurnitureObject.tsx
│   │   ├── Grid.tsx
│   │   └── Lights.tsx
│   ├── hooks/               # 3D-specific hooks
│   │   ├── useDragAndDrop.ts
│   │   ├── useSelection.ts
│   │   └── useTransform.ts
│   ├── utils/               # 3D utilities
│   │   ├── gridUtils.ts
│   │   └── objectUtils.ts
│   └── shaders/             # Custom shaders (if needed)
├── ui/                       # UI components
│   ├── components/          # React UI components
│   │   ├── CatalogPanel.tsx
│   │   ├── Toolbar.tsx
│   │   └── PropertiesPanel.tsx
│   └── hooks/              # UI hooks
│       └── useUIState.ts
├── store/                   # Zustand store
│   ├── sceneStore.ts        # Scene state
│   ├── selectionStore.ts    # Selection state
│   └── historyStore.ts      # Undo/redo state
└── assets/                  # Static assets
    ├── models/              # 3D models (glTF)
    └── textures/            # Texture files
```

## 3D Rendering Strategy

### Technology: React Three Fiber

**Decision**: React Three Fiber (R3F) with @react-three/drei

**Rationale**:
1. Team alignment: Project uses React + TypeScript. R3F is the natural choice.
2. Development speed: Declarative patterns match React mental model.
3. Ecosystem: `drei` provides ready-made solutions for orbit controls, transform handles, loaders.
4. Performance: Sufficient for MVP requirements (60fps, 500 objects).
5. Future-proof: Active development, growing community.

### Interaction Model

| Interaction | Approach | Implementation |
|-------------|----------|----------------|
| Selection | Click to select | Raycasting on pointer events |
| Movement | Drag on plane | Move object on floor plane |
| Rotation | Keyboard shortcuts (R key) | 15°/45° increments |
| Grid snapping | Default to 0.5m | Hold Shift for free movement |
| Camera | Orbit controls | @react-three/drei OrbitControls |

### Edit/View Modes

| Mode | Capabilities |
|------|--------------|
| Edit | Select, move, rotate, delete objects |
| View | Orbit, zoom, pan only. No editing. |

## Data Model

### Scene Graph Structure

```typescript
interface Design {
  id: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
  houseLayoutId: string;
  objects: FurnitureObject[];
  camera: CameraState;
  settings: SceneSettings;
}

interface FurnitureObject {
  id: string;
  catalogObjectId: string;
  position: { x: number; y: number; z: number };
  rotation: { x: number; y: number; z: number };
  scale: { x: number; y: number; z: number };
  properties: ObjectProperties;
  zIndex: number;
}

interface SceneSettings {
  gridSize: number;        // 0.5m
  snapEnabled: boolean;    // true
}
```

## Performance Strategy

| Target | Strategy |
|--------|----------|
| Frame rate | 60fps - RequestAnimationFrame, efficient rendering |
| Draw calls | <100 per frame - Instancing for repeated objects |
| Geometry | <100K triangles total - LOD for complex furniture |
| Texture memory | <100MB - Compressed textures, lazy load |
| Initial load | <3 seconds - Progressive loading, skeleton UI |

### Optimization Techniques

1. **Instancing**: Same object repeated 10+ times - THREE.InstancedMesh
2. **LOD**: Complex objects at distance - THREE.LOD
3. **Frustum culling**: Always (default) - Built-in Three.js
4. **Object pooling**: Frequent add/remove - Reuse object instances

## Risks & Mitigations

| Risk | Severity | Mitigation |
|------|----------|------------|
| 3D performance issues | High | Early prototype with representative scene |
| Asset pipeline complexity | Medium | Standardize on glTF, use established pipelines |
| Cross-browser WebGL support | Low | Test on Chrome, Firefox, Safari early |
| Memory leaks in 3D | High | Strict cleanup in React useEffect |
| State sync React ↔ Three.js | High | Use Zustand as single source of truth |

## Decisions Log

| ID | Decision | Date | Rationale |
|----|----------|------|------------|
| D-01 | Using React + TypeScript | 2026-04-08 | User requirement |
| D-02 | Web-based browser runtime | 2026-04-08 | User requirement |
| D-03 | Must support real-time interaction | 2026-04-08 | Core feature requirement |
| D-04 | 3D Engine: React Three Fiber | 2026-04-08 | Best React integration |
| D-05 | State Management: Zustand | 2026-04-08 | Simple, performant |
| D-06 | Grid resolution: 0.5m | 2026-04-06 | Product requirement |
| D-07 | Max objects: 500 | 2026-04-06 | Performance target |
| D-08 | Desktop-only MVP | 2026-04-06 | Reduce testing burden |
| D-09 | Local storage for MVP | 2026-04-06 | No auth complexity |

## Development Priority

| Priority | Action |
|----------|--------|
| 1 | Set up R3F Canvas with basic scene |
| 2 | Implement grid and floor plane |
| 3 | Create furniture object components |
| 4 | Implement drag-drop placement |
| 5 | Add selection and transform controls |
| 6 | Build catalog panel UI |
| 7 | Implement save/load (local storage) |
| 8 | Performance testing |

## Team

| Role | Name | Contact |
|------|------|---------|
| Lead Developer | User | - |

## Notes

This project is now **ready for development** following the completion of the architecture document. The frontend foundation is set up with React + TypeScript + Vite. Next step is installing React Three Fiber and building the 3D prototype.