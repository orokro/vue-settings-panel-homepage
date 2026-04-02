## 📚 Table of Contents

- [🚀 Introduction](#-introduction)
- [📦 Installation](#-installation)
- [⚡ Quick Start](#-quick-start)
- [🏗️ Architecture Overview](#️-architecture-overview)
- [🔌 The PNP Plugin](#-the-pnp-plugin)
- [🖱️ Directives](#️-directives)
  - [`v-pnp-draggable`](#-v-pnp-draggable)
  - [`v-pnp-dropzone`](#-v-pnp-dropzone)
  - [`v-pnp-draghandle`](#-v-pnp-draghandle)
- [👻 Drag Modes](#-drag-modes)
  - [`self`](#-self)
  - [`clone`](#-clone)
  - [Component (Custom Ghost)](#-component-custom-ghost)
  - [String](#-string)
- [↕️ Sorting & Reordering](#️-sorting--reordering)
- [👯 Multi-Select (Groups)](#-multi-select-groups)
- [🎨 Highlighting & Visual Feedback](#-highlighting--visual-feedback)
- [🧠 The PNPDragManager](#-the-pnpdragmanager)
- [🌫️ The PNPDragLayer](#️-the-pnpdraglayer)
- [✅ Validation & Keys](#-validation--keys)
- [⌨️ Cancel & Modifiers](#️-cancel--modifiers)
- [📱 Touch Support](#-touch-support)
- [🔁 Multiple Instances](#-multiple-instances)
- [🧼 Wrap Up](#-wrap-up)

---

## 🚀 Introduction

`vue-pick-n-plop` (PNP) is not a basic wrapper around the HTML5 Drag and Drop API. Instead, it is a custom implementation built on Pointer Events, offering a unified experience across mouse, touch, and stylus.

### Core Philosophy

- **Directive-First**: Implementation is handled primarily via attributes, keeping your templates clean and your logic decoupled.
- **Manager-Managed**: A central `PNPDragManager` tracks all active drags, validated targets, and coordinate transforms.
- **Global Layer**: All dragged items are rendered in a single, top-level `PNPDragLayer`. This ensures that "ghost" items are never clipped by parent containers with `overflow: hidden`.
- **Non-Destructive Sorting**: App data is only mutated at the *end* of a drag. During the drag, the library manages visual placeholders to indicate the drop position.

---

## 📦 Installation

```bash
npm install vue-pick-n-plop
```

### Peer Dependencies

PNP requires **Vue 3** (`^3.2.0`) and **gdraghelper** (for coordinate math and pointer normalization).

---

## ⚡ Quick Start

**1. Register the plugin:**

```js
import { createApp } from 'vue'
import PNP from 'vue-pick-n-plop'
import App from './App.vue'

const app = createApp(App)
app.use(PNP)
app.mount('#app')
```

**2. Mount the Drag Layer at your app root:**

```vue
<!-- App.vue -->
<template>
  <PNPDragLayer :z-index="9999" />
  <router-view />
</template>
```

**3. Wire up a drag:**

```vue
<template>
  <div v-pnp-draggable="{ keys: 'item', ctx: { id: 1 } }">
    Drag Me
  </div>

  <div v-pnp-dropzone="{ keys: 'item', onDropped: (drag) => console.log(drag) }">
    Drop Here
  </div>
</template>
```

---

## 🏗️ Architecture Overview

The library operates on a "Manager-managed" lifecycle:

1. **Directives** (`v-pnp-draggable`) watch for pointer interactions.
2. On start, they notify the **`PNPDragManager`**.
3. The Manager identifies valid **`PNPDropZones`** based on `keys` and optional validation functions.
4. The **`PNPDragLayer`** reactively mounts and follows the cursor.
5. On drop, the Manager calculates collisions and fires callbacks.

---

## 🔌 The PNP Plugin

The plugin is installed with `app.use(PNP, options?)`. It registers all three directives globally, the `PNPDragLayer` component, and provides the manager singleton under the key `'pnp-manager'`.

Plugin-level options are applied to the manager at install time and set the global defaults for every drag in your app:

```js
app.use(PNP, {
  cancelKey: 'Escape',       // Key to cancel a drag. Set to null to disable.
  rightClickCancel: true,    // Right-click during a drag cancels it.
  useTouch: false,           // Enable pointer-events for touch / stylus input.
  dragThreshold: 5,          // Default pixel threshold before a drag is initiated.
})
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `cancelKey` | `String\|null` | `'Escape'` | Keyboard key that cancels an in-progress drag. Any valid `KeyboardEvent.key` string. Pass `null` to disable. |
| `rightClickCancel` | `Boolean` | `true` | Right-clicking during a drag cancels it and suppresses the context menu. |
| `useTouch` | `Boolean` | `false` | Enables touch and stylus support via pointer events. See [Touch Support](#-touch-support). |
| `dragThreshold` | `Number` | `5` | Pixels the pointer must move before a drag starts. Prevents accidental drags from clicks. |

Options can also be changed at runtime via `manager.setOptions(opts)` — see [The PNPDragManager](#-the-pnpdragmanager). Changes take effect on the next drag and do not interrupt a drag already in progress.

---

## 🖱️ Directives

### 🤏 `v-pnp-draggable`

Attaches drag initiation logic to an element.

```vue
<div v-pnp-draggable="{
  keys: 'file|image',
  ctx: { id: 45, name: 'report.pdf' },
  dragItem: 'clone',
  dragThreshold: 5,
  showGroupCount: true,
  highlight: 'on-start',
  onDragStart: (ctx, groupCtx, modifiers) => { ... },
  onDropped: (success, dragCtx, dropCtx, groupCtx, modifiers) => { ... },
}">
  Drag me
</div>
```

#### Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `keys` | `String` | — | Pipe-separated key string (e.g., `'foo\|bar'`). Must share at least one key with a dropzone for that zone to be considered valid. |
| `ctx` | `Any` | `{}` | The data payload for this drag. Passed to all callbacks as `dragCtx`. |
| `dragItem` | `'self'\|'clone'\|String\|Component` | `'self'` | Controls what visual ghost is shown during the drag. See [Drag Modes](#-drag-modes). |
| `groupCtx` | `Array\|null` | `null` | Array of data objects representing a multi-selection. When set, callbacks receive the full array. See [Multi-Select](#-multi-select-groups). |
| `dragThreshold` | `Number` | manager default (`5`) | Pixels the cursor must move before the drag begins. Per-item override of the plugin-level default. Set to `0` to start drags immediately. |
| `showGroupCount` | `Boolean` | `false` | In `clone` mode, shows a red badge with `groupCtx.length` when more than one item is selected. |
| `requireHandle` | `Boolean` | `false` | When `true`, drag initiation is restricted to child elements with `v-pnp-draghandle`. Clicks elsewhere on the element are ignored. |
| `highlight` | `'on-hover'\|'on-start'` | `'on-hover'` | Controls when `.pnp-dropzone-valid` is applied to valid zones. `'on-start'` highlights all valid zones the moment the drag begins; `'on-hover'` only highlights the zone currently under the cursor. |
| `useHighlightBorder` | `Boolean` | `false` | When `true`, applies an inline `border` style to valid dropzones instead of (or in addition to) a CSS class. |
| `highlightBorderStyle` | `String` | `'2px solid #42b883'` | The CSS border value used when `useHighlightBorder` is `true`. |
| `validate` | `Function` | — | `(zoneCtx) => Boolean`. Runs on the *draggable* side to validate a specific zone's `ctx`. Used with `validateOnStart`. |
| `validateOnStart` | `Boolean` | `false` | When `true`, `validate` (and any `validate` on the dropzone) runs once at drag start rather than on every hover. |
| `validateByKeys` | `Boolean` | `true` | Set to `false` to skip key-based zone filtering entirely. Useful when all zones should be considered valid regardless of keys. |
| `onDragStart` | `Function` | — | `(ctx, groupCtx, modifiers) => void`. Fired once when the drag begins (after the threshold is crossed). |
| `onDropped` | `Function` | — | `(success, dragCtx, dropCtx, groupCtx, modifiers) => void`. Fired when the drag ends, whether or not it was a successful drop. `success` is `false` if cancelled or dropped outside a valid zone. |

#### Callback Signatures

`onDragStart(ctx, groupCtx, modifiers)`
- `ctx` — the draggable's `ctx` value.
- `groupCtx` — the multi-select array, or `null`.
- `modifiers` — `{ shiftKey, ctrlKey, altKey, metaKey }` captured from the initiating event.

`onDropped(success, dragCtx, dropCtx, groupCtx, modifiers)`
- `success` — `true` if dropped on a valid zone, `false` if cancelled or missed.
- `dragCtx` — the draggable's `ctx` value.
- `dropCtx` — the dropzone's `ctx` value, or `null` if the drop failed.
- `groupCtx` — the multi-select array, or `null`.
- `modifiers` — `{ shiftKey, ctrlKey, altKey, metaKey }`.

> **Note:** The draggable's `onDropped` always fires (success or not), which makes it ideal for detecting cancellations.

---

### 🎯 `v-pnp-dropzone`

Defines a target area for draggables.

```vue
<div v-pnp-dropzone="{
  keys: 'file',
  ctx: { folderId: 42 },
  sortable: true,
  orientation: 'vertical',
  placeholder: 'line',
  onDropped: (dragCtx, dropCtx, groupCtx, modifiers) => { ... },
  onSortDrop: (dragCtx, dropCtx, fromIndex, toIndex, groupCtx, modifiers) => { ... },
  validate: (dragCtx) => dragCtx.type === 'image',
}">
  Drop Here
</div>
```

#### Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `keys` | `String` | — | Pipe-separated key string. Accepts draggables whose `keys` share at least one value with this string. |
| `ctx` | `Any` | — | The data payload of this zone. Passed to all callbacks as `dropCtx`. |
| `sortable` | `Boolean` | `false` | Enables integrated list reordering. When a draggable originating from this zone is dragged, a placeholder tracks the insertion point. |
| `orientation` | `'vertical'\|'horizontal'` | `'vertical'` | Axis used for midpoint calculations during sorting. Use `'horizontal'` for row-based lists. |
| `placeholder` | `'space'\|'line'\|'dashed'` | `'space'` | Visual style of the sort insertion placeholder. See [Sorting & Reordering](#️-sorting--reordering). |
| `validate` | `Function` | — | `(dragCtx) => Boolean`. Called per-item to dynamically allow or reject specific draggables (e.g., "reject if disk is full"). Runs on hover by default, or at drag start if `validateOnStart` is set on the draggable. |
| `onDropped` | `Function` | — | `(dragCtx, dropCtx, groupCtx, modifiers) => void`. Fired on a successful drop onto this zone. |
| `onSortDrop` | `Function` | — | `(dragCtx, dropCtx, fromIndex, toIndex, groupCtx, modifiers) => void`. Fired on a successful sort-drop, providing the 0-based `fromIndex` and `toIndex`. |

#### Callback Signatures

`onDropped(dragCtx, dropCtx, groupCtx, modifiers)`
- `dragCtx` — the draggable's `ctx`.
- `dropCtx` — this zone's `ctx`.
- `groupCtx` — the multi-select array, or `null`.
- `modifiers` — `{ shiftKey, ctrlKey, altKey, metaKey }`.

`onSortDrop(dragCtx, dropCtx, fromIndex, toIndex, groupCtx, modifiers)`
- `dragCtx`, `dropCtx`, `groupCtx`, `modifiers` — same as above.
- `fromIndex` — the 0-based index of the dragged item at drag start.
- `toIndex` — the 0-based index where the item should be inserted after the drag.

> **Tip:** Both `onDropped` and `onSortDrop` can be used together on a sortable zone. `onDropped` fires for every successful drop (including cross-zone), while `onSortDrop` fires only for in-zone reorders (or cross-zone drops into a sortable zone) and provides index information.

---

### ⠿ `v-pnp-draghandle`

Restricts drag initiation to a specific child element of a `v-pnp-draggable`. Useful for rows or cards where only a grip icon should initiate the drag, not the entire element.

```vue
<div v-pnp-draggable="{ keys: 'item', ctx: item, requireHandle: true }">
  <span v-pnp-draghandle class="drag-grip">⠿</span>
  <span>Editable text — clicking here won't start a drag</span>
  <button>Some action button</button>
</div>
```

Multiple handles can be registered on a single draggable:

```vue
<div v-pnp-draggable="{ requireHandle: true, ... }">
  <span v-pnp-draghandle>⠿</span>
  <img v-pnp-draghandle src="..." /> <!-- This image is also a valid handle -->
  <p>Not a handle</p>
</div>
```

> **Important:** `v-pnp-draghandle` has no effect unless the parent `v-pnp-draggable` has `requireHandle: true`. The directive walks up the DOM at mount time to find its ancestor; a warning is logged if no `v-pnp-draggable` ancestor is found.

---

## 👻 Drag Modes

The `dragItem` option on `v-pnp-draggable` controls what the user sees being dragged. There are four modes.

---

### 📦 `self`

```vue
<div v-pnp-draggable="{ keys: 'item', ctx: item, dragItem: 'self' }">
  Drag me
</div>
```

The original DOM element is physically detached from its position in the DOM and appended to the `PNPDragLayer`'s self-container, making it follow the cursor. Vue's reconciliation is paused for this element during the drag.

When the drag ends (whether a successful drop or a cancellation), the element is automatically reinserted at its original position in the DOM using the recorded `originalParent` and `originalNextSibling` references.

**When to use:** When you want the actual live element (with all its Vue reactivity and event listeners) to appear to "float" while being dragged. Works well when the source list hides the item visually while it's in flight.

**Caveat:** Because the element is physically removed from the DOM, any CSS that depends on its original parent context (flex/grid layout, inherited styles) will no longer apply while it's in the drag layer.

---

### 👥 `clone`

```vue
<div v-pnp-draggable="{ keys: 'item', ctx: item, dragItem: 'clone' }">
  Drag me
</div>
```

A deep DOM clone (`cloneNode(true)`) of the element is created and rendered inside the `PNPDragLayer`. The original element stays in place — it may be hidden by a sortable zone's placeholder logic but is not moved in the DOM.

The clone is given `width: 100%` and `height: 100%` relative to the drag layer slot (which has the same pixel dimensions as the original element at drag-start), so it looks identical.

**When to use:** The safest and most common mode. Leaves the original DOM untouched and works correctly with `overflow: hidden` parents, since the ghost lives in the global layer.

**Group count badge:** When `showGroupCount: true` is set and `groupCtx.length > 1`, a red badge is automatically overlaid at the top-right of the clone showing the number of selected items.

```vue
<div v-pnp-draggable="{
  keys: 'task',
  ctx: task,
  dragItem: 'clone',
  groupCtx: selectedTasks,
  showGroupCount: true,
}">
  Task card
</div>
```

---

### 🧩 Component (Custom Ghost)

Pass any Vue component to `dragItem`. It will be mounted inside the `PNPDragLayer` and receives live reactive props from the drag state.

```vue
<script setup>
import MyDragGhost from './MyDragGhost.vue';
</script>

<template>
  <div v-pnp-draggable="{
    keys: 'card',
    ctx: { id: 1, label: 'Hello' },
    dragItem: MyDragGhost,
  }">
    Drag me
  </div>
</template>
```

The component receives the following props automatically:

| Prop | Type | Description |
|------|------|-------------|
| `ctx` | `Any` | The primary dragged item's `ctx` value. |
| `groupCtx` | `Array\|null` | The full multi-selection array, or `null` for a single-item drag. |
| `delta` | `{ x: Number, y: Number }` | The cumulative offset in pixels from the drag start position. |
| `startMouse` | `{ x: Number, y: Number }` | The screen coordinates (viewport-relative) where the drag began. |
| `currentMouse` | `{ x: Number, y: Number }` | The current screen coordinates of the pointer (live, updates every frame). |

**The component is remounted on every drag** — the drag layer uses `:key="manager.dragId"` which increments with each new drag. This ensures that any `onMounted` animations or transitions trigger fresh every time, not just on the first drag.

**Example: animated file pile ghost**

The FileBrowserDemo shows a powerful use of this pattern. The custom ghost captures the `getBoundingClientRect()` of each selected file in `onDragStart`, enriches `groupCtx` with those rects, then uses them to position each card at its real screen location on first render. A single `requestAnimationFrame` delay triggers a CSS transition that eases all cards into a fanned stack under the cursor:

```vue
<script setup>
import { ref, onMounted, markRaw, defineComponent, h } from 'vue';

const MyGhost = markRaw(defineComponent({
  props: ['ctx', 'groupCtx', 'delta', 'startMouse', 'currentMouse'],
  setup(props) {
    const animated = ref(false);

    onMounted(() => {
      // Paint at world-space positions, then trigger the pile transition
      requestAnimationFrame(() => { animated.value = true; });
    });

    return () => h('div', { style: { /* ... */ } }, [
      /* render cards using animated.value + props.groupCtx[n]._rect */
    ]);
  },
}));
</script>

<template>
  <div v-pnp-draggable="{
    keys: 'file',
    ctx: file,
    groupCtx: selectedFiles,
    dragItem: MyGhost,
    onDragStart: (ctx, gc) => {
      // Capture real screen positions for the ghost to use
      gc.forEach(item => {
        item._rect = document.getElementById(`file-${item.id}`).getBoundingClientRect();
        item._isAnchor = item.id === ctx.id;
      });
    },
  }">
    <!-- file card -->
  </div>
</template>
```

> **Important:** Wrap your component with `markRaw()` to prevent Vue from making it reactive (component definitions should not be reactive objects). Not doing so can cause warnings and unexpected re-renders.

---

### 🔤 String

Any string value other than `'self'` or `'clone'` is treated as literal text and rendered in a default drag badge that follows the cursor.

```vue
<div v-pnp-draggable="{ keys: 'item', ctx: item, dragItem: '🍪 YUM!' }">
  Cookie
</div>
```

The string is rendered inside a `div.pnp-drag-item-default` positioned at the same size as the original element. This is useful for simple placeholder labels or emoji-only drag ghosts without any extra setup.

---

## ↕️ Sorting & Reordering

PNP handles list reordering without requiring "live" data mutations during the drag. The source array is only updated once in your `onSortDrop` callback when the drag ends.

### Enabling sorting

Add `sortable: true` to a dropzone. Direct children of that zone that have `v-pnp-draggable` applied are treated as sortable siblings.

```vue
<div v-pnp-dropzone="{
  keys: 'item',
  ctx: listMeta,
  sortable: true,
  orientation: 'vertical',
  placeholder: 'line',
  onSortDrop: (dragCtx, dropCtx, fromIndex, toIndex) => {
    const item = items.splice(fromIndex, 1)[0];
    items.splice(toIndex, 0, item);
  },
}">
  <div
    v-for="item in items"
    :key="item.id"
    v-pnp-draggable="{ keys: 'item', ctx: item }"
  >
    {{ item.label }}
  </div>
</div>
```

### Placeholder styles

The `placeholder` option controls the visual indicator that appears during the drag to show where the item will land.

| Value | Description |
|-------|-------------|
| `'space'` (default) | An invisible element sized identically to the dragged item. Sibling items shift as if the dragged item is still in its destination slot. |
| `'line'` | A thin 3px colored line (blue, `#4a90d9`) that appears between items. The line is horizontal for vertical lists and vertical for horizontal lists. Good for compact lists. |
| `'dashed'` | A dashed-border ghost sized to the dragged item with a subtle blue fill. Combines the visual weight of a space placeholder with the clarity of a visible marker. |

### How midpoint-threshold detection works

A reorder is only triggered when the cursor crosses the **midpoint** of a sibling element. This prevents the "oscillation" bug common in many sortable libraries, where an item flickers back and forth if you hold the cursor near a boundary.

The algorithm:

1. Each frame (gated by `requestAnimationFrame` for 60fps performance), the current cursor position is compared to the midpoints of all sortable siblings.
2. The placeholder is moved to insert before the first sibling whose midpoint is "below" (or "to the right" of) the cursor, or appended at the end if none qualify.
3. The actual DOM data mutation only happens in your `onSortDrop` callback after the pointer is released.

### Sorting across zones

Cross-zone sorting works automatically — if a draggable from one sortable zone is dragged over a different sortable zone, the placeholder migrates into the new zone. `onSortDrop` on the target zone receives the correct `toIndex` for the new zone.

### Nested sortable zones

Draggables inside nested sortable sub-zones are excluded from the parent zone's sibling calculations. PNP walks up the DOM to identify nesting, so you can safely nest sortable lists without interference.

---

## 👯 Multi-Select (Groups)

PNP supports dragging a set of items simultaneously via the `groupCtx` option. PNP doesn't manage your selection state — you maintain it in your component — but it passes the selection through to all callbacks.

### Basic pattern

```vue
<script setup>
import { ref } from 'vue';

const selectedIds = ref(new Set());
const items = ref([...]);

const toggle = (id) => {
  const next = new Set(selectedIds.value);
  next.has(id) ? next.delete(id) : next.add(id);
  selectedIds.value = next;
};

const draggableOpts = (item) => ({
  keys: 'task',
  ctx: item,
  dragItem: 'clone',
  showGroupCount: true,
  // If this item is selected, pass the whole selection as groupCtx.
  // If it's not selected, pass null to drag just this one item.
  groupCtx: selectedIds.value.has(item.id)
    ? items.value.filter(i => selectedIds.value.has(i.id))
    : null,
  onDragStart: () => {
    // If you drag an unselected item, clear the selection
    if (!selectedIds.value.has(item.id)) selectedIds.value = new Set();
  },
});
</script>

<template>
  <div
    v-for="item in items"
    :key="item.id"
    v-pnp-draggable="draggableOpts(item)"
    @click="toggle(item.id)"
    :class="{ selected: selectedIds.has(item.id) }"
  >
    {{ item.label }}
  </div>

  <div
    v-pnp-dropzone="{
      keys: 'task',
      onDropped: (dragCtx, dropCtx, groupCtx) => {
        // groupCtx is the full selection array, or null for a single-item drag
        const ids = groupCtx ? groupCtx.map(i => i.id) : [dragCtx.id];
        // ... move them all
      },
    }"
  >
    Drop here
  </div>
</template>
```

### The `showGroupCount` badge

When `dragItem: 'clone'` and `showGroupCount: true`, the clone ghost automatically gets a red badge in its top-right corner showing `groupCtx.length`. This is rendered by the drag layer and requires no extra markup.

### World-space animation with `onDragStart`

`onDragStart` fires immediately when the drag begins, *before* the ghost is rendered. This makes it the ideal place to capture `getBoundingClientRect()` for each selected element and store those rects on your `groupCtx` objects so a custom ghost component can use them to animate from each item's real on-screen position.

---

## 🎨 Highlighting & Visual Feedback

### CSS classes

The library automatically applies and removes CSS classes on dropzone elements:

| Class | Applied When... |
|-------|----------------|
| `.pnp-dropzone-valid` | This zone is valid for the currently dragged item. |
| `.pnp-dropzone-hovered` | The currently dragged item is hovering directly over this zone. |

By default (with `highlight: 'on-hover'`), `.pnp-dropzone-valid` is only applied to the zone that is currently under the cursor. Set `highlight: 'on-start'` on the **draggable** to apply `.pnp-dropzone-valid` to all valid zones immediately when the drag begins:

```vue
<div v-pnp-draggable="{
  keys: 'item',
  ctx: item,
  highlight: 'on-start',
}">
  Drag me to see all valid targets highlight
</div>
```

### Styling the states

```css
/* All valid targets are outlined while a compatible item is being dragged */
.my-dropzone.pnp-dropzone-valid {
  outline: 2px dashed #4a90d9;
  outline-offset: -2px;
}

/* The specific zone currently under the cursor */
.my-dropzone.pnp-dropzone-hovered {
  background: rgba(74, 144, 217, 0.08);
}
```

### Inline border highlight

For simpler cases, PNP can apply an inline `border` style to valid zones automatically using options on the draggable:

```vue
<div v-pnp-draggable="{
  keys: 'item',
  ctx: item,
  useHighlightBorder: true,
  highlightBorderStyle: '2px solid #42b883',
}">
```

The original inline border is saved and restored when the drag ends.

---

## 🧠 The PNPDragManager

The manager is the engine of the library. It is provided globally by the plugin and can be accessed in any component via the `usePNPDragging` composable:

```js
import { usePNPDragging } from 'vue-pick-n-plop';

// Must be called inside a component setup() context
const manager = usePNPDragging();
```

### Reactive state

| Property | Type | Description |
|----------|------|-------------|
| `manager.isDragging` | `Ref<Boolean>` | `true` while a drag is in progress. |
| `manager.hoveredZoneId` | `Ref<String\|null>` | The UUID of the dropzone currently under the cursor, or `null`. |
| `manager.dragId` | `Ref<Number>` | Increments on every drag. Used by the drag layer to remount components. |
| `manager.activeDrag` | `Reactive<Object>` | Full state of the current drag. See below. |

### `activeDrag` object

All properties are reactive and update in real time during the drag:

| Property | Type | Description |
|----------|------|-------------|
| `el` | `HTMLElement\|null` | The source DOM element being dragged. |
| `keys` | `String[]` | The parsed keys array from the draggable's `keys` option. |
| `ctx` | `Any` | The draggable's `ctx` value. |
| `groupCtx` | `Array\|null` | The multi-select array, or `null`. |
| `startMouse` | `{ x, y }` | Viewport-relative pointer position at drag start. |
| `currentMouse` | `{ x, y }` | Current viewport-relative pointer position (updated every frame). |
| `delta` | `{ x, y }` | Cumulative offset in pixels from the drag start position. |
| `initialRect` | `DOMRect\|null` | The bounding rect of the source element at drag start. |
| `currentDropZone` | `Object\|null` | The dropzone the cursor is currently over, or `null`. |
| `validDropZones` | `Object[]` | All dropzones that passed key-matching (and optional validation) for this drag. |
| `modifiers` | `{ shiftKey, ctrlKey, altKey, metaKey }` | Modifier key state captured at drag start. Cleared on window blur to prevent desync. |
| `sortPlaceholder` | `HTMLElement\|null` | The placeholder DOM element currently in use for sorting. |
| `sortFromIndex` | `Number` | The 0-based index of the dragged item at drag start (`-1` if not sorting). |

### Methods

**`manager.setOptions(opts)`**

Merges partial options into the manager config. Takes effect on the next drag:

```js
manager.setOptions({
  cancelKey: null,       // Disable Escape-to-cancel
  rightClickCancel: false,
  dragThreshold: 10,
});
```

**`manager.cancelDrag()`**

Programmatically cancels the current drag. Clears `currentDropZone` so the drag resolves with `success: false`. Callbacks still fire.

```js
const manager = usePNPDragging();

// Cancel the drag if the user hits a certain key combination
window.addEventListener('keydown', (e) => {
  if (e.key === 'q') manager.cancelDrag();
});
```

**`manager.startDrag(el, options, event?)`**

You can trigger a drag programmatically instead of waiting for a user interaction. Rarely needed, but useful for testing or custom input devices.

### Global event buses

Listen for drag start/end events globally across the entire app, regardless of which component initiated the drag:

```js
const manager = usePNPDragging();

// Fires when any drag starts
manager.onStartEventBus.on((ctx, groupCtx, modifiers) => {
  console.log('Drag started:', ctx);
});

// Fires when any drag ends (success or cancellation)
manager.onDroppedEventBus.on(({ success, dragCtx, dropCtx, groupCtx, modifiers }) => {
  console.log('Drag ended:', success, dragCtx);
});

// Remember to clean up
onUnmounted(() => {
  manager.onStartEventBus.off(myHandler);
  manager.onDroppedEventBus.off(myOtherHandler);
});
```

---

## 🌫️ The PNPDragLayer

A required component that must be mounted **once** near the root of your application. All drag ghost elements are rendered inside it.

```vue
<!-- App.vue -->
<template>
  <PNPDragLayer :z-index="10000" />
  <router-view />
</template>
```

### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `zIndex` | `Number\|String` | `9001` | The CSS `z-index` of the drag layer. Should be high enough to sit above all other content. |

### How it works

- The layer is a `position: fixed; inset: 0` overlay with `pointer-events: none`, so it never interferes with your UI.
- It uses `v-show` (not `v-if`) so it is always in the DOM, which allows the self-container to be available for direct DOM appending in `'self'` mode.
- Custom component ghosts are wrapped with `:key="manager.dragId"`, so the component is fully remounted on each new drag. This ensures `onMounted` animations and Vue transitions play fresh every time.
- Ghost elements are positioned using `position: fixed` at `initialRect.left + delta.x`, `initialRect.top + delta.y`, matching the dimensions of the source element exactly.

### Multiple instances

While only one `PNPDragLayer` is typically needed, the library uses a reference counter (`hasDragLayer`) so you can safely mount multiple layers in different parts of your component tree. Dragging is only permitted while at least one layer is mounted.

---

## ✅ Validation & Keys

PNP uses a two-tier validation system to control which draggables can be dropped on which zones.

### Tier 1: Key matching (static)

Keys are pipe-separated strings. A draggable and a dropzone are compatible if their key sets share at least one value.

```
draggable keys: 'file|image'
dropzone  keys: 'file'          ← compatible (both have 'file')
dropzone  keys: 'video'         ← not compatible
dropzone  keys: 'image|video'   ← compatible (both have 'image')
```

This filtering happens once at drag start and is very fast. Set `validateByKeys: false` on a draggable to skip key matching entirely and treat all registered zones as candidates.

### Tier 2: Functional validation (dynamic)

Use `validate` on a **dropzone** for complex per-item logic:

```vue
<div v-pnp-dropzone="{
  keys: 'file',
  validate: (dragCtx) => dragCtx.size < 100 * 1024 * 1024, // reject files > 100MB
  onDropped: handleDrop,
}">
```

Use `validate` on a **draggable** to validate against zone context:

```vue
<div v-pnp-draggable="{
  keys: 'file',
  ctx: file,
  validate: (zoneCtx) => zoneCtx.acceptedTypes.includes(file.type),
}">
```

Functional validation **does not run by default**. You must opt into it by setting `validateOnStart: true` on either the draggable or the dropzone. When set, validation runs once at drag start (as part of the valid-zone computation) rather than being re-evaluated on every hover.

Add it to the **dropzone**:

```vue
<div v-pnp-dropzone="{
  keys: 'file',
  validateOnStart: true,
  validate: (dragCtx) => dragCtx.size < 100 * 1024 * 1024,
  onDropped: handleDrop,
}">
```

Or add it to the **draggable** (applies globally for that drag across all zones):

```vue
<div v-pnp-draggable="{
  keys: 'item',
  ctx: item,
  validateOnStart: true,
  validate: (zoneCtx) => zoneCtx.acceptedTypes.includes(item.type),
}">
```

---

## ⌨️ Cancel & Modifiers

### Cancelling a drag

By default, two gestures cancel an in-progress drag:

- **`Escape` key** — configurable via `cancelKey` (set to `null` to disable).
- **Right-click** — configurable via `rightClickCancel: false`. When triggered, the context menu is suppressed for that interaction.

Cancellation resolves the drag with `success: false`, so `onDropped` on the draggable still fires with `success = false`.

Configure these at install time or at runtime:

```js
// At install time
app.use(PNP, { cancelKey: null, rightClickCancel: false })

// At runtime
const manager = usePNPDragging();
manager.setOptions({ cancelKey: 'Escape', rightClickCancel: true });
```

Detecting a cancellation in your draggable callback:

```js
onDropped: (success, dragCtx, dropCtx) => {
  if (!success) {
    console.log('Drag was cancelled or missed');
    return;
  }
  // handle successful drop...
}
```

### Modifier keys

PNP captures modifier key state from the initiating `mousedown`/`pointerdown` event and passes it through to all callbacks. This lets you implement different behaviors depending on held keys.

```js
onDropped: (success, dragCtx, dropCtx, groupCtx, modifiers) => {
  if (!success) return;

  if (modifiers.altKey) {
    // Copy instead of move
  } else if (modifiers.shiftKey) {
    // Move and mark as urgent
  } else {
    // Normal move
  }
}
```

Available modifiers: `shiftKey`, `ctrlKey`, `altKey`, `metaKey`.

> **Window blur safety:** Modifier state is cleared if the window loses focus during a drag (e.g., Alt-Tab while holding Alt). This prevents a desync where a key is permanently stuck as "held" because the `keyup` event was missed.

---

## 📱 Touch Support

Touch and stylus support is disabled by default to avoid interfering with normal scroll behavior. Enable it at the plugin level:

```js
app.use(PNP, { useTouch: true })
```

Or at runtime:

```js
manager.setOptions({ useTouch: true });
```

When `useTouch` is enabled, `pointerdown` events from touch and pen inputs trigger drags (pointer type `'mouse'` is still handled by the `mousedown` path). `preventDefault()` is called on the pointer event to suppress the browser's synthesised `mousedown` that would otherwise fire ~300ms later.

The drag threshold (`dragThreshold`) applies equally to touch input, so short taps don't accidentally start drags.

---

## 🔁 Multiple Instances

The plugin provides the manager singleton under the key `'pnp-manager'`. All directives and the composable resolve the same instance through Vue's `inject` mechanism.

For advanced use cases where you need two completely independent drag systems on the same page (e.g., two separate boards that should not interfere), you can provide a custom manager before installing the plugin:

```js
import PNP from 'vue-pick-n-plop';
import { PNPDragManager } from 'vue-pick-n-plop/PNPDragManager';

const myManager = new PNPDragManager({ cancelKey: null });

app.provide('pnp-manager', myManager); // must come before app.use(PNP)
app.use(PNP);                          // plugin skips provide() since key already exists
```

The `PNPDragManager` constructor accepts the same options object as the plugin:

```js
new PNPDragManager({
  cancelKey: 'Escape',
  rightClickCancel: true,
  useTouch: false,
  dragThreshold: 5,
})
```

---

## 🧼 Wrap Up

### CSS class reference

| Class | Element | Description |
|-------|---------|-------------|
| `.pnp-drag-layer` | `PNPDragLayer` root div | The fixed full-screen overlay. |
| `.pnp-drag-item-self-container` | Ghost container | Wraps the element in `'self'` mode. |
| `.pnp-drag-item-clone` | Ghost container | Wraps the cloned DOM in `'clone'` mode. |
| `.pnp-clone-content` | Inner div | The actual clone content within the clone container. |
| `.pnp-drag-item-component` | Ghost container | Wraps the custom Vue component in component mode. |
| `.pnp-drag-item-default` | Ghost container | Wraps the text content in string mode. |
| `.pnp-group-count-badge` | Badge span | The red count badge on multi-select clone ghosts. |
| `.pnp-dropzone-valid` | Any dropzone | Applied to valid zones during a compatible drag. |
| `.pnp-dropzone-hovered` | Any dropzone | Applied to the zone currently under the cursor. |

### Checklist

- `<PNPDragLayer />` is mounted at the app root (required — dragging is silently disabled without it).
- `keys` on draggable and dropzone share at least one common value.
- Callback signatures match the expected arguments (draggable's `onDropped` receives `success` as the first arg; dropzone's `onDropped` does not).
- When using component mode, the component is wrapped with `markRaw()`.
- When using `requireHandle`, at least one child has `v-pnp-draghandle`.
- `onSortDrop` is used (instead of `onDropped`) when you need the `fromIndex`/`toIndex` values.

---

MIT Licensed · Made with ❤️ by [Greg Miller]
