# Project Context: homebuilder-3d

> Created: 2026-04-08
> Status: active
> Last Updated: 2026-04-09

## Overview

- **Name**: HomeBuilder 3D
- **Description**: Web-based 3D house designer with drag-and-drop furniture placement
- **Path**: C:/Repos/HomeBuilder 3D
- **Repository**: false
- **Author**: Architect Agent

## Core Context

### Mission

Deliver a browser-based 3D home design experience for arranging furniture inside fixed house layouts with fast, intuitive interaction.

### Technical Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | React 19 + TypeScript | Primary application runtime |
| Build Tool | Vite | Fast local builds |
| 3D Engine | React Three Fiber | Scene rendering |
| 3D Helpers | @react-three/drei | Controls and helpers |
| State Management | Zustand | Shared editor state |
| Styling | Tailwind CSS | UI styling |
| Backend | .NET 9 (planned) | Post-MVP |
| Database | PostgreSQL (planned) | Post-MVP |
| API Style | REST | Planned server integration |

### Current Architecture

Current MVP architecture is frontend-first:

```text
Catalog UI + Editor UI -> Zustand state -> React Three Fiber scene -> local persistence
```

Planned future expansion:

```text
Browser client -> REST API -> backend services -> PostgreSQL
```

### Active Decisions

| ID | Decision | Date | Rationale |
|----|----------|------|-----------|
| D-01 | Use React + TypeScript | 2026-04-08 | Project requirement |
| D-02 | Use React Three Fiber | 2026-04-08 | Best fit for React-based 3D delivery |
| D-03 | Use Zustand | 2026-04-08 | Simple, performant shared state |
| D-04 | Grid snapping is 0.5m by default | 2026-04-06 | Product requirement |
| D-05 | Desktop-only MVP | 2026-04-06 | Reduce early complexity and testing burden |
| D-06 | Local storage for MVP | 2026-04-06 | Avoid auth and backend dependency for first release |
| D-07 | Maximum target scene size is 500 objects | 2026-04-06 | Performance target |

## Delivery Context

### Confirmed Requirements

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-01 | Fixed square/grid-based house structure | P0 | pending |
| FR-02 | 3D viewport with orbit camera | P0 | pending |
| FR-03 | Object catalog UI | P0 | pending |
| FR-04 | Drag-and-drop object placement | P0 | pending |
| FR-05 | Object repositioning | P0 | pending |
| FR-06 | Object rotation | P0 | pending |
| FR-07 | Grid snapping at 0.5m | P0 | pending |
| FR-08 | Object selection by click | P0 | pending |
| FR-09 | Basic materials and colors | P0 | pending |
| FR-10 | Multiple layouts beyond MVP | P1 | pending |
| FR-11 | Undo and redo | P1 | pending |
| FR-12 | Design sharing by URL | P1 | pending |
| FR-13 | Save and load to backend | P1 | future |
| FR-14 | CAD and blueprint export | P1 | future |

### Active Constraints

- Desktop-only MVP
- Browser runtime only for MVP
- Anonymous storage remains local-only in MVP
- MVP uses two fixed layouts
- Initial catalog target is 100 objects
- Performance target is 60 FPS with up to 500 objects

### Known Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| 3D performance issues in representative scenes | high | Prototype early with realistic object counts |
| Memory leaks in 3D lifecycle code | high | Enforce cleanup discipline around scene resources and effects |
| React and Three.js state divergence | high | Keep Zustand as the single source of truth |
| Asset pipeline complexity | medium | Standardize on glTF/GLB and document import flow |
| Cross-browser WebGL behavior | medium | Test early on Chrome, Safari, Firefox, and Edge |

### Open Questions

| ID | Question | Status | Next Step |
|----|----------|--------|-----------|
| Q-01 | Which asset CDN should support future hosted models: Cloudflare R2 or AWS S3? | open | Decide when backend scope begins |
| Q-02 | What object categories are required for the initial 100-item catalog? | open | Clarify product inventory scope |
| Q-03 | What precision is needed for export quality in post-MVP CAD and blueprint output? | open | Defer until export work is scheduled |

### Pending Tasks

| ID | Task | Owner | Status |
|----|------|-------|--------|
| T-01 | Set up R3F canvas with a basic scene | frontend-developer | pending |
| T-02 | Implement grid and floor plane | frontend-developer | pending |
| T-03 | Create furniture object components | frontend-developer | pending |
| T-04 | Implement drag-and-drop placement | frontend-developer | pending |
| T-05 | Add selection and transform controls | frontend-developer | pending |
| T-06 | Build catalog panel UI | frontend-developer | pending |
| T-07 | Implement local save and load | frontend-developer | pending |
| T-08 | Run performance testing on representative scenes | performance-optimizer | pending |

## Historical Context

### Archived Decisions

No archived decisions yet.

### Resolved Risks

No resolved risks recorded yet.

### Learnings

No reusable learnings recorded yet.

## Notes

Interaction model currently assumes:

- click-to-select
- drag on plane for movement
- keyboard shortcut for rotation
- default snapping with optional free movement override
- orbit controls for camera movement
