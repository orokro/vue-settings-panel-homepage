# 🫳 vue-pick-n-plop

> A sophisticated, directive-driven drag-and-drop library for Vue 3. Designed for complex UI interactions, featuring world-space animations, robust multi-selection, integrated sorting with midpoint-threshold detection, and a global drag layer that defies `overflow: hidden`.

---

## 📚 Table of Contents

- 🚀 Introduction
- 📦 Installation
- ⚡ Quick Start
- 🏗️ Architecture Overview
- 🔌 The `PNP` Plugin
- 🖱️ Directives
  - 🤏 `v-pnp-draggable`
  - 🎯 `v-pnp-dropzone`
  - ⠿ `v-pnp-draghandle`
- 👻 Drag Modes
  - 📦 `self`
  - 👥 `clone`
  - 🧩 `component` (Custom Ghost)
  - 🔤 `string`
- ↕️ Sorting & Reordering
- 👯 Multi-Select (Groups)
- 🧠 The `PNPDragManager`
- 🌫️ The `PNPDragLayer`
- ✅ Validation & Keys
- ⌨️ Cancel & Modifiers
- 📱 Touch Support
- 🎨 CSS Highlighting
- 🧼 Wrap Up

---

## 🚀 Introduction

`vue-pick-n-plop` (PNP) is not a basic wrapper around the HTML5 Drag and Drop API. Instead, it is a custom implementation built on Pointer Events, offering a unified experience across mouse, touch, and stylus. 

### Core Philosophy

- **Directive-First**: Implementation is handled primarily via attributes, keeping your templates clean and your logic decoupled.
- **Manager-Managed**: A central `PNPDragManager` tracks all active drags, validated targets, and coordinate transforms.
- **Global Layer**: All dragged items are rendered in a single, top-level `PNPDragLayer`. This ensures that \"ghost\" items are never clipped by parent containers with `overflow: hidden`.
- **Non-Destructive Sorting**: App data is only mutated at the *end* of a drag. During the drag, the library manages visual placeholders (lines or spaces) to indicate the drop position.

---

## 📦 Installation

```bash
npm install vue-pick-n-plop
```

### Peer Dependencies

PNP requires **Vue 3** (`^3.2.0`) and **gdraghelper** (for coordinate math and pointer normalization).

---

## ⚡ Quick Start

1. **Register the plugin**:
```js
import { createApp } from 'vue'
import PNP from 'vue-pick-n-plop'
import App from './App.vue'

const app = createApp(App)
app.use(PNP)
app.mount('#app')
```

2. **Mount the Drag Layer** at your app root:
```vue
<!-- App.vue -->
<template>
  <PNPDragLayer :z-index=\"9999\" />
  <router-view />
</template>
```

3. **Wire up a drag**:
```vue
<template>
  <!-- The Draggable -->
  <div v-pnp-draggable=\"{ keys: 'item', ctx: { id: 1 } }\">
    Drag Me
  </div>

  <!-- The Dropzone -->
  <div v-pnp-dropzone=\"{ keys: 'item', onDropped: (drag) => console.log(drag) }\">
    Drop Here
  </div>
</template>
```

---

## 🏗️ Architecture Overview

The library operates on a \"Manager-managed\" lifecycle:

1.  **Directives** (`v-pnp-draggable`) watch for pointer interactions.
2.  On start, they notify the **`PNPDragManager`**.
3.  The Manager identifies valid **`PNPDropZones`** based on `keys`.
4.  The **`PNPDragLayer`** reactively mounts and follows the cursor.
5.  On drop, the Manager calculates collisions and fires callbacks.

---

## 🖱️ Directives

### 🤏 `v-pnp-draggable`

Attaches drag initiation logic to an element.

```js
v-pnp-draggable=\"{
  keys: 'file|image',
  ctx: { id: 45, name: 'report.pdf' },
  dragItem: 'clone',
  dragThreshold: 5,
  onDragStart: (ctx, groupCtx, modifiers) => { ... }
}\"
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `keys` | `String` | — | Piped string (e.g., `'foo\|bar'`). Must match at least one key on a dropzone. |
| `ctx` | `Any` | — | The data payload of the item. Passed to all callbacks. |
| `dragItem` | `String\|Component` | `'self'` | The visual mode: `'self'`, `'clone'`, or a Vue Component. |
| `groupCtx` | `Array` | `null` | Array of data objects representing a multi-selection. |
| `dragThreshold` | `Number` | `5` | Pixels the pointer must move before the drag actually starts. |
| `onDragStart` | `Function` | — | `(ctx, groupCtx, modifiers) => void` |
| `onDropped` | `Function` | — | `(success, dragCtx, dropCtx, groupCtx, modifiers) => void` |
| `requireHandle` | `Boolean` | `false` | If true, drag only starts from a `v-pnp-draghandle`. |

---

### 🎯 `v-pnp-dropzone`

Defines a target area for draggables.

```js
v-pnp-dropzone=\"{
  keys: 'file',
  sortable: true,
  orientation: 'vertical',
  placeholder: 'line',
  onSortDrop: (dragCtx, dropCtx, from, to) => { ... }
}\"
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `keys` | `String` | — | Piped string. Items with matching keys are accepted. |
| `ctx` | `Any` | — | The data payload of the zone. |
| `sortable` | `Boolean` | `false` | Enables integrated list reordering logic. |
| `orientation` | `'vertical'\|'horizontal'` | `'vertical'` | Axis used for sorting midpoint calculations. |
| `placeholder` | `'space'\|'line'\|'dashed'` | `'space'` | Visual style of the insertion placeholder. |
| `onDropped` | `Function` | — | `(dragCtx, dropCtx, groupCtx, modifiers) => void` |
| `onSortDrop` | `Function` | — | `(dragCtx, dropCtx, fromIndex, toIndex, groupCtx, modifiers) => void` |
| `validate` | `Function` | — | `(dragCtx) => Boolean`. Dynamic drop validation. |

---

### ⠿ `v-pnp-draghandle`

Restricts drag initiation to a specific child element of a `v-pnp-draggable`.

```vue
<div v-pnp-draggable=\"{ requireHandle: true }\">
  <span v-pnp-draghandle>⠿</span>
  <span>I am just text</span>
</div>
```

---

## 👻 Drag Modes

### 📦 `self`
The original DOM element is physically appended to the `PNPDragLayer`. Vue's DOM reconciliation is paused for this element until it returns home after the drop.

### 👥 `clone`
The element is cloned via `cloneNode(true)`. The original stays in place (or is hidden if sorting). This is the safest mode for most UIs.

### 🧩 `component` (Custom Ghost)
Pass a Vue component to `dragItem`. It will be mounted in the drag layer and receive the following props:
- `ctx`: The primary item data.
- `groupCtx`: The full selection array.
- `delta`: `{ x, y }` distance from start.
- `startMouse` / `currentMouse`: `{ x, y }` screen coordinates.

### 🔤 `string`
Pass a string (e.g., `'🍪 YUM!'`). The drag layer will simply render this text in a badge following the cursor.

---

## ↕️ Sorting & Reordering

PNP handles list reordering without requiring \"live\" data mutations during the drag.

- **Midpoint Threshold**: A swap is only triggered when the cursor crosses the vertical/horizontal midpoint of a sibling. This prevents the \"oscillation\" bug common in many libraries.
- **Placeholders**: The library inserts a temporary DOM element (styleable via the `placeholder` option) to mark where the item will land.
- **rAF Throttling**: Collision math is gated by `requestAnimationFrame` for 60fps performance even in long lists.

---

## 👯 Multi-Select (Groups)

To drag multiple items at once:
1. Maintain your own selection state in your app.
2. Pass that selection array to `groupCtx` on the draggable.
3. Use `onDropped` to move the entire collection.

If `showGroupCount: true` is set on the draggable, a `+N` badge will automatically appear on clone-mode ghosts.

---

## 🧠 The `PNPDragManager`

The manager is the engine of the library. It is provided globally by the plugin.

```js
import { usePNPDragging } from 'vue-pick-n-plop';
const manager = usePNPDragging();

// Manager State (Reactive Refs)
manager.isDragging.value;   // true/false
manager.activeDrag;         // The full state of the current drag
manager.hoveredZoneId.value // ID of the zone currently under the mouse
```

---

## 🌫️ The `PNPDragLayer`

A required component that must be mounted once at the root of your application.

```vue
<PNPDragLayer :z-index=\"10000\" />
```

- **Props**: `zIndex` (Number). Default is `9001`.
- **Optimization**: It uses `v-show` and `:key` remounting to ensure custom ghost animations start fresh on every drag.

---

## ✅ Validation & Keys

The library uses a two-tier validation system:
1. **Keys**: Fast, static filtering. A draggable with `keys: 'A|B'` can drop on a zone with `keys: 'A'` or `keys: 'B'`.
2. **Functional Validation**: Use the `validate` function on a dropzone for complex logic (e.g., \"don't allow move if disk is full\").

---

## 🎨 CSS Highlighting

The library automatically applies reactive classes to dropzones:

| Class | Applied When... |
|-------|----------------|
| `.pnp-dropzone-valid` | A valid item is currently being dragged anywhere on screen. |
| `.pnp-dropzone-hovered` | A valid item is currently hovering *over* this specific zone. |

```css
.my-zone.pnp-dropzone-valid {
  border: 2px dashed #4a90d9;
}
.my-zone.pnp-dropzone-hovered {
  background: rgba(74, 144, 217, 0.1);
}
```

---

MIT Licensed · Made with ❤️ by [Greg Miller]