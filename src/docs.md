# 🎛️ vue-settings-panel

> A flexible, high-performance, and fully themeable settings panel for Vue 3 applications. Designed for complex configuration requirements with support for hierarchical categories, advanced search, real-time input sanitization, conditional visibility, and eleven distinct input types.

---

## 📚 Table of Contents

- 🚀 Introduction
- 📦 Installation
- ⚡ Quick Start
- 🧱 The `<VueSettingsPanel />` Component
- 🗂️ The Specification Object
  - 🔧 Global Options
  - 📁 Category Object
  - 📂 SubCategory Object
  - ⚙️ Setting Configuration
- 🧩 Input Types (`TYPES`)
  - ✅ Boolean
  - 🔤 String
  - 🔢 Number
  - 📋 Select
  - 🎚️ FloatRange
  - 🎨 Color
  - 🖼️ Image
  - 📅 Date
  - 🕐 Time
  - ⌨️ KeyInput
  - 🔌 GenericInput
- 🛠️ `createSettings` Utility
- 🎨 Theming System
  - 🖌️ Full Theme Reference
  - 🌑 Dark Theme Example
- 🔍 Search Behavior
- 👁️ Conditional Visibility
- 🧹 Lint (Input Sanitization)
- ✅ Validate (Input Validation)
- 📌 Setting Placement (`cats` and `mount`)
- 💾 Saving & Restoring Settings
- 🏗️ Architecture Overview
- 📐 Responsive Behavior
- 🔧 Development
- 🧼 Wrap Up

---

## 🚀 Introduction

`vue-settings-panel` provides a full-featured, drop-in settings UI for Vue 3 applications. Rather than building settings screens by hand — wiring up inputs, managing state, handling validation, and so on — you define a **specification object** that describes what your settings look like, and the panel renders the entire UI for you.

### Core Concepts

- **Specification**: A plain JavaScript object that defines the structure — categories, subcategories, and the settings themselves.
- **Settings Object**: A reactive Vue object that holds the current values of every setting. Created and initialized by the `createSettings` utility.
- **Types**: The panel ships with eleven built-in input types (toggle, text, number, slider, color picker, etc.) that are referenced via the `TYPES` export.
- **Theming**: The panel is entirely styled via CSS variables. Pass a partial theme object to override just the parts you care about.

👉 You define the structure. The panel handles rendering, search, validation, layout, and reactivity.

---

## 📦 Installation

```bash
npm install vue-settings-panel
```

You must also import the bundled stylesheet once in your application:

```js
import 'vue-settings-panel/dist/style.css'
```

### Peer Dependencies

The panel requires **Vue 3** (`^3.0.0`) as a peer dependency. It does not bundle Vue itself.

Icons are rendered using **Material Icons**. If you plan to use icon slugs (like `"settings"` or `"tune"`) for your category icons, include Material Icons in your project:

```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" />
```

Alternatively, you can use a full image URL string as a category icon instead.

---

## ⚡ Quick Start

Here is the minimal setup to get the panel running:

```vue
<script setup>
import { VueSettingsPanel, TYPES, createSettings } from 'vue-settings-panel'
import 'vue-settings-panel/dist/style.css'

const mySpec = {
  categories: [
    { name: "General", slug: "gen", icon: "settings" }
  ],
  settings: {
    userName: {
      name: "Username",
      cats: ["gen"],
      type: TYPES.String,
      default: "Guest"
    },
    notifications: {
      name: "Enable Notifications",
      cats: ["gen"],
      type: TYPES.Boolean,
      default: true
    }
  }
}

const settingsState = createSettings(mySpec)

const handleChange = (newSettings) => {
  console.log("Settings updated:", newSettings)
}
</script>

<template>
  <div style="height: 600px; width: 800px;">
    <VueSettingsPanel
      :settings="settingsState"
      :specification="mySpec"
      @settings-changed="handleChange"
    />
  </div>
</template>
```

> ⚠️ **Important:** The panel fills its parent container. Always give the parent a defined `height` and `width`.

---

## 🧱 The `<VueSettingsPanel />` Component

This is the root component. Mount it once and pass your specification and settings state.

```html
<VueSettingsPanel
  :settings="settingsState"
  :specification="mySpec"
  :themeColors="myTheme"
  @settings-changed="handleChange"
/>
```

### Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `settings` | `Object` | ✅ Yes | — | The reactive settings state object (from `createSettings`). |
| `specification` | `Object` | ✅ Yes | — | The spec object that defines the panel's structure. |
| `themeColors` | `Object` | No | `{}` | Partial or full theme override (deep-merged with defaults). |

### Events

| Event | Payload | Description |
|-------|---------|-------------|
| `settings-changed` | `Object` (full settings) | Emitted whenever any setting's value changes. |

### Slots

There are no slots. The panel is fully controlled by the specification.

---

## 🗂️ The Specification Object

The specification is a plain JavaScript object (not reactive) that defines the entire layout and behavior of the panel.

```js
const mySpec = {
  sidePanel: { ... },
  search: { ... },
  categories: [ ... ],
  settings: { ... }
}
```

---

### 🔧 Global Options

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `sidePanel` | `Object` | — | Controls sidebar behavior. |
| `sidePanel.enabled` | `Boolean` | `true` | Show or hide the left sidebar navigation. When `false`, a sticky search bar appears at the top of the main content column instead. |
| `sidePanel.autoCollapse` | `Boolean` | `false` | When `true`, the sidebar automatically collapses to icon-only mode on small screens (< 768px). |
| `search` | `Object` | — | Controls how the panel behaves during search. |
| `search.nonMatchedCategories` | `'gray'` \| `'collapse'` | — | What to do with categories that don't match the search query. `'gray'` dims them to 40% opacity; `'collapse'` hides them entirely. |
| `search.nonMatchSettings` | `'gray'` \| `'collapse'` | — | What to do with individual settings that don't match the search query. |
| `categories` | `Array` | — | Array of Category objects (see below). |
| `settings` | `Object` | — | Key-value map of Setting configurations (see below). |

**Example:**

```js
const mySpec = {
  sidePanel: {
    enabled: true,
    autoCollapse: true
  },
  search: {
    nonMatchedCategories: 'collapse',
    nonMatchSettings: 'gray'
  },
  categories: [ ... ],
  settings: { ... }
}
```

---

### 📁 Category Object

Categories appear as top-level items in the sidebar and as titled sections in the main content area.

```js
{
  name: "Audio",
  slug: "audio",
  desc: "Sound output and input configuration",
  icon: "volume_up",
  show: (settings) => settings.audioEnabled,
  categories: [ ... ]  // optional subcategories
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `String` | ✅ Yes | Display name shown in the sidebar and as the section header. |
| `slug` | `String` | ✅ Yes | Unique identifier used to associate settings with this category via the `cats` array. |
| `desc` | `String` | No | Subtitle or description shown below the category header in the main content area. |
| `icon` | `String` | No | Either a Material Icons slug (e.g. `"settings"`) or a full image URL. |
| `show` | `Function` | No | `(settings) => Boolean`. Controls conditional visibility. When it returns `false`, the category and all its settings are hidden. Evaluated reactively. |
| `categories` | `Array` | No | Array of SubCategory objects nested inside this category. One level of nesting is supported. |

---

### 📂 SubCategory Object

Subcategories nest inside a parent category. They appear as sub-items in the sidebar and as indented sub-sections in the main column.

```js
{
  name: "Buffer Settings",
  slug: "buffer",
  desc: "Low-level audio buffer tuning",
  icon: "tune",
  show: (settings) => settings.advancedMode
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `String` | ✅ Yes | Display name. |
| `slug` | `String` | ✅ Yes | Unique identifier. Referenced in `cats` as `'parentSlug.subSlug'`. |
| `desc` | `String` | No | Subtitle or description for the sub-section. |
| `icon` | `String` | No | Material Icons slug or image URL. |
| `show` | `Function` | No | `(settings) => Boolean`. Conditional visibility. |

> ⚠️ **Nesting limit**: Only one level of subcategory nesting is supported. SubCategories cannot contain further nested categories.

---

### ⚙️ Setting Configuration

Each key in `spec.settings` is the setting's **data key** — the property name on the settings state object.

```js
settings: {
  audioVolume: {
    name: "Master Volume",
    desc: "Controls the overall output level",
    cats: ["audio", "audio.buffer"],
    type: TYPES.FloatRange,
    opts: { min: 0, max: 100, step: 1, showInput: true },
    default: 80,
    tags: ["volume", "loudness", "output"],
    mount: "right",
    show: (settings) => settings.audioEnabled,
    lint: (val) => Math.max(0, Math.min(100, val)),
    validate: (val) => val >= 0 || "Volume cannot be negative"
  }
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `String` | ✅ Yes | The primary label displayed to the left of the input. |
| `desc` | `String` | No | Helper text displayed below the name in smaller, muted type. |
| `cats` | `Array<String>` | ✅ Yes | Slugs of the categories and/or subcategories where this setting should appear. A setting can appear in multiple locations simultaneously. Use `'categorySlug'` to place it in a top-level category, or `'categorySlug.subcategorySlug'` to place it inside a subcategory. |
| `type` | `Object` | ✅ Yes | A reference to one of the `TYPES` export values (e.g. `TYPES.String`, `TYPES.Boolean`). |
| `opts` | `Object` | No | Type-specific options. Available options vary by type — see the [Input Types](#-input-types-types) section. |
| `default` | `Any` | ✅ Yes* | The initial value for this setting. Required when using `createSettings`. |
| `tags` | `Array<String>` | No | Additional strings used to improve search discovery. These are not displayed to the user as labels, but if a tag matches the search query, the setting is considered a match. Matched tags are shown as dark labels under the setting name. |
| `mount` | `'right'` \| `'bottom'` | No | Controls where the input widget is positioned relative to the label. `'right'` (default) places it to the right on the same row. `'bottom'` places it below the label at full width. Multiline string inputs always use `'bottom'` regardless of this setting. |
| `show` | `Function` | No | `(settings) => Boolean`. Conditional visibility. When it returns `false`, the setting row is hidden. Evaluated reactively as other settings change. |
| `lint` | `Function` | No | `(value) => sanitizedValue`. Called on every keystroke/change. Use it to enforce formatting in real time (e.g., strip spaces, force lowercase). The return value is applied as the new setting value. |
| `validate` | `Function` | No | `(value) => true \| "Error String"`. Called on blur (when the input loses focus). Return `true` for a valid value, or a non-empty string to display an error message below the setting name. Validation is feedback-only — it does not block the value from being stored. |

---

## 🧩 Input Types (`TYPES`)

Import the `TYPES` object to reference input types in your specification:

```js
import { TYPES } from 'vue-settings-panel'
```

The `TYPES` object contains eleven named type definitions. Each has a `name`, a `slug`, a `defaultValue`, and an internal component reference.

---

### ✅ Boolean

Renders a toggle switch. The value is `true` or `false`.

```js
{
  name: "Enable Notifications",
  type: TYPES.Boolean,
  default: false
}
```

| Property | Value |
|----------|-------|
| `TYPES.Boolean.slug` | `"boolean"` |
| `TYPES.Boolean.defaultValue` | `false` |
| `opts` | None |
| Data format | `Boolean` |

---

### 🔤 String

Renders a single-line text input or a multi-line textarea.

```js
{
  name: "API Endpoint",
  type: TYPES.String,
  opts: {
    placeholder: "https://api.example.com",
    multiline: false,
    rows: 3
  },
  default: ""
}
```

| Property | Value |
|----------|-------|
| `TYPES.String.slug` | `"string"` |
| `TYPES.String.defaultValue` | `""` |
| Data format | `String` |

**`opts` options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `placeholder` | `String` | `""` | Placeholder text shown when the input is empty. |
| `multiline` | `Boolean` | `false` | When `true`, renders a `<textarea>` instead of a single-line input. Also forces `mount: 'bottom'` regardless of the setting's `mount` value. |
| `rows` | `Number` | — | Number of visible rows for the textarea (only applies when `multiline: true`). |

---

### 🔢 Number

Renders a numeric input with optional min/max/step constraints.

```js
{
  name: "Max Connections",
  type: TYPES.Number,
  opts: {
    min: 1,
    max: 100,
    step: 1
  },
  default: 10
}
```

| Property | Value |
|----------|-------|
| `TYPES.Number.slug` | `"number"` |
| `TYPES.Number.defaultValue` | `0` |
| Data format | `Number` |

**`opts` options:**

| Option | Type | Description |
|--------|------|-------------|
| `min` | `Number` | Minimum allowed value. |
| `max` | `Number` | Maximum allowed value. |
| `step` | `Number` | Step increment for the spin buttons. |

---

### 📋 Select

Renders a dropdown `<select>` or a set of radio buttons.

```js
{
  name: "Theme",
  type: TYPES.Select,
  opts: {
    options: ["light", "dark", "system"],
    radio: false
  },
  default: "system"
}
```

| Property | Value |
|----------|-------|
| `TYPES.Select.slug` | `"select"` |
| `TYPES.Select.defaultValue` | `""` |
| Data format | `Any` (matches the value type from your `options` array) |

**`opts` options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `options` | `Array` | — | **Required.** The list of choices. Can be an array of primitive values (strings, numbers) or objects. |
| `radio` | `Boolean` | `false` | When `true`, renders radio buttons instead of a dropdown. |

> **Tip:** When using `radio: true`, consider setting `mount: 'bottom'` for readability, especially with more than 3 options.

---

### 🎚️ FloatRange

Renders a range slider with optional numeric input display.

```js
{
  name: "Master Volume",
  type: TYPES.FloatRange,
  opts: {
    min: 0,
    max: 100,
    step: 0.5,
    showInput: true
  },
  default: 75
}
```

| Property | Value |
|----------|-------|
| `TYPES.FloatRange.slug` | `"float-range"` |
| `TYPES.FloatRange.defaultValue` | `0` |
| Data format | `Number` |

**`opts` options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `min` | `Number` | — | Minimum value of the slider range. |
| `max` | `Number` | — | Maximum value of the slider range. |
| `step` | `Number` | — | Increment step for fine-grained control. |
| `showInput` | `Boolean` | `false` | When `true`, also renders a numeric input field alongside the slider so the user can type exact values. |

---

### 🎨 Color

Renders a native color picker. The value is stored as a hex string.

```js
{
  name: "Accent Color",
  type: TYPES.Color,
  default: "#00abae"
}
```

| Property | Value |
|----------|-------|
| `TYPES.Color.slug` | `"color"` |
| `TYPES.Color.defaultValue` | `"#000000"` |
| `opts` | None |
| Data format | `String` (hex, e.g. `"#ff0000"`) |

---

### 🖼️ Image

Renders an image uploader with a preview and a clear button.

```js
{
  name: "Profile Picture",
  type: TYPES.Image,
  opts: {
    format: "base64"
  },
  default: ""
}
```

| Property | Value |
|----------|-------|
| `TYPES.Image.slug` | `"image"` |
| `TYPES.Image.defaultValue` | `""` |
| Data format | `String` (base64 data URL or file path, depending on `format`) |

**`opts` options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `format` | `'base64'` \| `'path'` | — | `'base64'` reads the file as a base64-encoded data URL and stores the full data string. `'path'` stores only the filename. Use `'path'` when you handle the actual file upload yourself. |

---

### 📅 Date

Renders a native date picker. The value is stored as a `YYYY-MM-DD` string.

```js
{
  name: "Start Date",
  type: TYPES.Date,
  default: ""
}
```

| Property | Value |
|----------|-------|
| `TYPES.Date.slug` | `"date"` |
| `TYPES.Date.defaultValue` | `""` |
| `opts` | None |
| Data format | `String` (`YYYY-MM-DD`) |

---

### 🕐 Time

Renders a native time picker. The value is stored as an `HH:mm` string.

```js
{
  name: "Daily Backup Time",
  type: TYPES.Time,
  default: "03:00"
}
```

| Property | Value |
|----------|-------|
| `TYPES.Time.slug` | `"time"` |
| `TYPES.Time.defaultValue` | `""` |
| `opts` | None |
| Data format | `String` (`HH:mm`) |

---

### ⌨️ KeyInput

Renders a keyboard shortcut capture widget. Pressing keys while the widget is active records the combination. Modifier keys (Ctrl, Shift, Alt, Meta) are tracked separately from the main key.

```js
{
  name: "Save Shortcut",
  type: TYPES.KeyInput,
  opts: {
    onInputRequest: undefined  // optional external capture
  },
  default: { shift: false, ctrl: true, alt: false, meta: false, key: "s" }
}
```

| Property | Value |
|----------|-------|
| `TYPES.KeyInput.slug` | `"key-input"` |
| `TYPES.KeyInput.defaultValue` | `{ shift: false, ctrl: false, alt: false, meta: false, key: '' }` |
| Data format | `Object` — `{ shift: Boolean, ctrl: Boolean, alt: Boolean, meta: Boolean, key: String }` |

**`opts` options:**

| Option | Type | Description |
|--------|------|-------------|
| `onInputRequest` | `Function` | Optional external capture callback. Called with a `set(value)` function when the widget enters capture mode. Useful for Electron apps or custom key-binding drivers where native browser key events aren't sufficient. |

**Data shape detail:**

```js
{
  shift: Boolean,   // Shift modifier
  ctrl:  Boolean,   // Ctrl modifier
  alt:   Boolean,   // Alt modifier
  meta:  Boolean,   // Meta/Command modifier
  key:   String     // The main key (e.g. "s", "F5", "ArrowUp")
}
```

---

### 🔌 GenericInput

A fully external-driven capture widget. It displays a placeholder and a capture button. When the user clicks capture, your `onInputRequest` callback is called, and you provide the value. Useful for MIDI input, gamepad buttons, custom device capture, or any input that can't be captured by the browser keyboard events.

```js
{
  name: "MIDI Trigger",
  type: TYPES.GenericInput,
  opts: {
    placeholder: "Click to assign MIDI",
    onInputRequest: (set) => {
      waitForMidiInput().then((midiNote) => {
        set(midiNote)
      })
    }
  },
  default: ""
}
```

| Property | Value |
|----------|-------|
| `TYPES.GenericInput.slug` | `"generic-input"` |
| `TYPES.GenericInput.defaultValue` | `""` |
| Data format | `Any` — entirely determined by what you pass to `set()` |

**`opts` options:**

| Option | Type | Description |
|--------|------|-------------|
| `onInputRequest` | `Function` | **Required for useful behavior.** Called with a `set(value)` function when capture is initiated. While waiting for input, the widget displays a "Waiting for input…" state. Call `set(value)` with the captured value to finalize. |
| `placeholder` | `String` | Text shown in the widget before any value is captured. |

---

## 🛠️ `createSettings` Utility

```js
import { createSettings } from 'vue-settings-panel'

const settingsState = createSettings(spec, initialData)
```

`createSettings` builds a **reactive Vue object** from your specification, pre-filled with the correct default values.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `spec` | `Object` | ✅ Yes | Your specification object. |
| `initialData` | `Object` | No | Optional initial values, e.g. loaded from `localStorage` or a server. Only keys that exist in `spec.settings` are applied — unknown keys are silently ignored. |

### How it works

1. Iterates over every key in `spec.settings`.
2. Sets the initial value to `setting.default` (or `null` if no default is provided).
3. Overlays any matching keys from `initialData`, overwriting the defaults.
4. Returns the result wrapped in Vue's `reactive()`.

### Example: Loading saved settings

```js
const savedData = JSON.parse(localStorage.getItem("myAppSettings") || "{}")
const settingsState = createSettings(mySpec, savedData)
```

### Example: Saving on change

```vue
<VueSettingsPanel
  :settings="settingsState"
  :specification="mySpec"
  @settings-changed="(s) => localStorage.setItem('myAppSettings', JSON.stringify(s))"
/>
```

> **Note:** `createSettings` returns a Vue `reactive()` object. You can access and modify individual values directly (e.g. `settingsState.volume = 50`) and the panel will react to the change. You can also pass it directly to `JSON.stringify` because `reactive()` objects serialize like plain objects.

---

## 💾 Saving & Restoring Settings

Since `@settings-changed` gives you the full settings object, persistence is straightforward:

```js
// On change — save to localStorage
const handleChange = (settings) => {
  localStorage.setItem("appSettings", JSON.stringify(settings))
}

// On app load — restore from localStorage
const saved = JSON.parse(localStorage.getItem("appSettings") || "{}")
const settingsState = createSettings(mySpec, saved)
```

You can swap `localStorage` for any storage mechanism — `IndexedDB`, a server call, Electron's `store`, etc.

---

## 🎨 Theming System

The panel is styled via CSS custom properties injected at the root `.vue-settings-panel` element. You control the theme by passing a `themeColors` object to the component.

```html
<VueSettingsPanel
  :settings="settingsState"
  :specification="mySpec"
  :themeColors="myTheme"
/>
```

The `themeColors` object is **deep-merged** with the built-in defaults — you only need to specify the values you want to override. All other values fall back to the defaults.

```js
const myTheme = {
  mainColumn: {
    bgColor: "#1a1a2e",
    textColor: "#e0e0e0"
  },
  toggle: {
    activeBgColor: "#6c63ff"
  }
}
```

---

### 🖌️ Full Theme Reference

Below is the complete theme object with every available key and its default value:

```js
const defaultTheme = {

  // ─── Left Column (Sidebar) ─────────────────────────────────────────────────
  leftColumn: {
    bgColor: '#f5f5f5',                   // Sidebar background
    categoriesBoxBgColor: 'transparent',  // Background of the categories list container
    categoriesBoxBorder: 'none',          // Border around the categories list container
    categoryColor: '#333',                // Icon color for unselected categories
    categoryTextColor: '#333',            // Text color for unselected categories
    selectedCategoryBgColor: '#e0e0e0',   // Background of the currently selected category
    selectedCategoryTextColor: '#000',    // Text color of the selected category
    searchBgColor: '#fff',                // Search bar background
    searchXColor: '#666',                 // Color of the search clear (×) button
    searchTextColor: '#000',              // Color of text typed in the search bar
  },

  // ─── Main Column (Content Area) ────────────────────────────────────────────
  mainColumn: {
    bgColor: '#fff',                      // Main content area background
    textColor: '#333',                    // General text color
    categoryHeaderColor: '#222',          // Background of category title headers
    categoryHeaderTextColor: '#fff',      // Text color inside category headers
    categoryBorder: '1px solid #eee',     // Border around each category section
    categoryBgColor: '#fff',              // Background of the category content area
    categoryTextColor: '#333',            // Text color inside category sections
    settingsRowBgColor: 'transparent',    // Background of individual setting rows
    settingsRowNameColor: '#000',         // Color of the setting's primary label
    settingsRowDescColor: '#666',         // Color of the setting's description text
    settingsRowBorder: '1px solid #f0f0f0', // Border between setting rows
    subcategoryHeaderColor: '#444',       // Background of subcategory title headers
    subcategoryHeaderTextColor: '#fff',   // Text color inside subcategory headers
    subcategoryBorder: 'none',            // Border around subcategory sections
    subcategoryBgColor: '#f9f9f9',        // Background of subcategory content areas
    subCategoryTextColor: '#333',         // Text color inside subcategory sections
    attentionColor: '#00abae',            // Pulse/outline color when a subcategory is focused via sidebar click
  },

  // ─── Toggle Switch ─────────────────────────────────────────────────────────
  toggle: {
    bgColor: '#ccc',          // Background when toggle is OFF
    thumbColor: '#fff',       // Color of the sliding thumb
    activeBgColor: '#4caf50'  // Background when toggle is ON
  },

  // ─── Text / Number Inputs ──────────────────────────────────────────────────
  input: {
    borderColor: '#ccc',          // Default input border color
    bgColor: '#fff',              // Input background
    textColor: '#333',            // Input text color
    focusBorderColor: '#4caf50'   // Input border color when focused
  },

  // ─── Range Slider ──────────────────────────────────────────────────────────
  range: {
    thumbColor: '#4caf50',  // Color of the slider thumb handle
    trackColor: '#ccc'      // Color of the slider track
  }
}
```

---

### 🌑 Dark Theme Example

```js
const darkTheme = {
  leftColumn: {
    bgColor: '#1e1e1e',
    categoryColor: '#aaa',
    categoryTextColor: '#aaa',
    selectedCategoryBgColor: '#37373d',
    selectedCategoryTextColor: '#fff',
    searchBgColor: '#2d2d2d',
    searchXColor: '#aaa',
    searchTextColor: '#fff'
  },
  mainColumn: {
    bgColor: '#121212',
    textColor: '#e0e0e0',
    categoryHeaderColor: '#1f1f1f',
    categoryHeaderTextColor: '#fff',
    categoryBorder: '1px solid #2a2a2a',
    categoryBgColor: '#1a1a1a',
    categoryTextColor: '#ccc',
    settingsRowBgColor: 'transparent',
    settingsRowNameColor: '#fff',
    settingsRowDescColor: '#888',
    settingsRowBorder: '1px solid #222',
    subcategoryHeaderColor: '#1f1f1f',
    subcategoryHeaderTextColor: '#ccc',
    subcategoryBgColor: '#161616',
    subCategoryTextColor: '#bbb',
    attentionColor: '#bb86fc'
  },
  toggle: {
    bgColor: '#444',
    thumbColor: '#fff',
    activeBgColor: '#bb86fc'
  },
  input: {
    borderColor: '#444',
    bgColor: '#2d2d2d',
    textColor: '#e0e0e0',
    focusBorderColor: '#bb86fc'
  },
  range: {
    thumbColor: '#bb86fc',
    trackColor: '#444'
  }
}
```

---

## 🔍 Search Behavior

When the sidebar is enabled, a search bar appears at the top of the left column. When the sidebar is disabled, the search bar appears sticky at the top of the main content area.

Search matches against:
- **Category names**
- **Category descriptions**
- **Setting names**
- **Setting descriptions**
- **Setting tags** (the `tags` array)
- **Subcategory names**

### Search Highlighting

Matched text is highlighted with a yellow background. If a setting is matched through a tag (not through its visible text), the matched tag is displayed as a dark badge beneath the setting name.

### Non-Match Behavior

Control what happens to non-matching items with the `search` spec option:

```js
search: {
  nonMatchedCategories: 'gray',    // 'gray' | 'collapse'
  nonMatchSettings: 'collapse'     // 'gray' | 'collapse'
}
```

- `'gray'`: The item remains visible but is dimmed (40% opacity).
- `'collapse'`: The item is hidden entirely.

These two options are independent — you can gray out settings while collapsing categories, or vice versa.

---

## 👁️ Conditional Visibility

Categories, subcategories, and individual settings can all use a `show` function to conditionally appear or disappear based on the current state of other settings.

```js
// A setting that only shows when another setting is enabled
{
  name: "Buffer Size",
  type: TYPES.Number,
  cats: ["audio"],
  default: 512,
  show: (settings) => settings.audioEnabled
}
```

```js
// A category that only appears in advanced mode
{
  name: "Developer",
  slug: "dev",
  icon: "code",
  show: (settings) => settings.advancedMode
}
```

The `show` function:
- Receives the full `settings` reactive object as its only argument.
- Should return a `Boolean`.
- Is **evaluated reactively** — when any setting changes, all `show` functions are re-evaluated automatically.
- Works identically for categories, subcategories, and settings.

---

## 🧹 Lint (Input Sanitization)

The `lint` function runs on **every keystroke or change event**, before the value is committed to the settings state. Use it to enforce a specific format in real time.

```js
{
  name: "CSS Class Name",
  type: TYPES.String,
  lint: (val) => val.toLowerCase().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, '')
}
```

```js
{
  name: "Port Number",
  type: TYPES.Number,
  lint: (val) => Math.max(1, Math.min(65535, val))
}
```

The return value of `lint` is what gets stored. The function should always return a value — returning `undefined` or `null` may cause unexpected behavior.

---

## ✅ Validate (Input Validation)

The `validate` function runs on **blur** (when the input loses focus). It provides feedback to the user but does **not** block the value from being stored.

```js
{
  name: "Email Address",
  type: TYPES.String,
  validate: (val) => {
    if (!val.includes('@')) return "Please enter a valid email address"
    return true
  }
}
```

- Return `true` to indicate the value is valid (no message shown).
- Return a non-empty `String` to display an error message below the setting name.

> **Note:** Validation is advisory only. If you need to reject invalid values, use `lint` to sanitize them before they're stored, or handle the `@settings-changed` event and reject the update in your application logic.

---

## 📌 Setting Placement (`cats` and `mount`)

### `cats` — Category Assignment

A setting can appear in **multiple locations** simultaneously by listing more than one slug in its `cats` array.

```js
cats: ["general", "audio", "audio.buffer"]
```

- `"general"` — appears in the top-level "General" category.
- `"audio"` — also appears in the top-level "Audio" category.
- `"audio.buffer"` — also appears inside the "Buffer Settings" subcategory within "Audio".

This is useful for surfacing the same setting in both a general overview and a detailed sub-section.

### `mount` — Widget Positioning

| Value | Behavior |
|-------|----------|
| `'right'` (default) | The input widget is placed to the right of the label on the same row. The widget has a max-width of ~40% of the row. |
| `'bottom'` | The input widget is placed below the label at full row width. Good for wide inputs like radio groups or long text areas. |

> Multiline string inputs (`opts.multiline: true`) always behave as `mount: 'bottom'` regardless of the `mount` setting.

---

## 🏗️ Architecture Overview

The panel is composed of several internal components that you don't interact with directly, but it's useful to understand the layout:

```
VueSettingsPanel (root)
├── LeftColumn (sidebar)
│   ├── SearchBar
│   └── CategoriesList
│       └── CategoryItem (per category)
│           └── SubCategoryItem (per subcategory)
└── MainColumn (content area)
    ├── SearchBar (sticky, only when sidebar disabled)
    └── CategoryBox (per category)
        ├── SettingRow (top-level settings)
        └── SubCategoryBox (per subcategory)
            └── SettingRow (subcategory settings)
```

Each `SettingRow` renders the setting's label, description, validation error, and the type-specific input component.

### State Flow

- The `settings` prop is a reactive object owned by your application.
- When a user changes a value, the input component calls an `update` handler.
- The handler emits `settings-changed` from the root component.
- Conditional `show` functions, search filtering, and tag highlighting all read from the same reactive `settings` object.

### Provide/Inject

State is shared down the component tree via Vue's `provide`/`inject` system, not through prop-drilling. The root `VueSettingsPanel` provides:
- The `settings` object
- The `specification`
- The current search query
- The selected category slug

---

## 📐 Responsive Behavior

### Sidebar Collapse

The sidebar (`LeftColumn`) can be in one of two states:
- **Expanded**: 280px wide, showing category names, icons, and the search bar.
- **Collapsed**: 60px wide, showing icons only.

Collapse is triggered by:
- The user clicking the collapse toggle button in the sidebar.
- **Automatic collapse** when `sidePanel.autoCollapse: true` is set and the viewport width drops below 768px.

### Subcategory Selection Animation

When a user clicks a subcategory in the sidebar, the main column smoothly scrolls to the corresponding subcategory box and plays a brief attention pulse animation — a 0.8-second outline flash using `attentionColor`.

### Custom Scrollbars

The main column has a scrollable content area with custom-styled scrollbars that match the panel's theme.

---

## 🔧 Development

```bash
# Install dependencies
npm install

# Run the interactive demo app with hot-module replacement
npm run dev

# Build the library for publishing to npm
npm run build

# Build the demo app for static deployment
npm run build:demo

# Preview the production build of the demo app
npm run preview
```

### Build Output

Running `npm run build` produces the following in `dist/`:

| File | Format | Use |
|------|--------|-----|
| `vue-settings-panel.es.js` | ESM | For `import` in bundled projects |
| `vue-settings-panel.umd.js` | UMD | For `require` or CDN `<script>` usage |
| `style.css` | CSS | Required stylesheet — must be imported separately |

---

## 🧼 Wrap Up

`vue-settings-panel` handles all the boilerplate of a settings screen so you can focus on your application. Define your settings in a spec, initialize state with `createSettings`, mount the component, and you're done.

From there, everything scales: add subcategories, wire up conditional visibility, persist with `localStorage`, customize the theme, hook up MIDI or gamepad input via `GenericInput` — it all builds on the same simple foundation.

---

MIT Licensed · Made with ❤️ by [Greg Miller]
