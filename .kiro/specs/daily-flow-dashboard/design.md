# Design Document: Daily Flow Dashboard

## Overview

The Daily Flow Dashboard is a single-file, zero-dependency productivity tool delivered as `index.html`. It runs entirely in the browser with no server, no build step, and no external JavaScript libraries. All state is persisted to `localStorage`. The UI is built with HTML5, styled with Tailwind CSS (loaded from CDN with the `class` dark-mode strategy), and driven by vanilla JavaScript.

The dashboard exposes four widgets:

| Widget | Purpose |
|---|---|
| Greeting | Time-sensitive salutation in the page header |
| Focus Timer | 25-minute Pomodoro countdown with audio alert on completion |
| Task List | To-do list with add, toggle, inline-edit, delete, and sort |
| Quick Links | Named URL bookmarks that open in a new tab |

The existing `index.html` already implements the full feature set. This spec documents the design so that future changes, tests, and enhancements have a clear reference.

---

## Architecture

Because the entire application lives in a single HTML file, there is no module system, bundler, or import graph. The architecture is a classic **event-driven DOM application**:

```
┌─────────────────────────────────────────────────────┐
│                    index.html                       │
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌────────────────┐  │
│  │  HTML    │   │  CSS     │   │  JavaScript    │  │
│  │ Structure│   │ (Tailwind│   │  (inline       │  │
│  │          │   │  + custom│   │   <script>)    │  │
│  └──────────┘   └──────────┘   └───────┬────────┘  │
│                                        │            │
│              ┌─────────────────────────┤            │
│              │                         │            │
│        ┌─────▼──────┐         ┌────────▼───────┐   │
│        │  DOM State  │         │  localStorage  │   │
│        │  (live UI)  │◄───────►│  (persistence) │   │
│        └────────────┘         └────────────────┘   │
└─────────────────────────────────────────────────────┘
```

**Data flow:**
1. On `window.onload`, state is read from `localStorage` and rendered into the DOM.
2. User interactions trigger JavaScript functions that mutate in-memory arrays (`tasks`, `links`) or scalar state (`timerSeconds`, `timerInterval`).
3. After every mutation, the relevant render function re-draws the affected DOM section and writes the updated state back to `localStorage`.

There is no virtual DOM, no reactive framework, and no component lifecycle — the render functions are simple imperative DOM writes (`innerHTML`, `innerText`).

---

## Components and Interfaces

### 1. Greeting Component

**DOM:** `#greeting` (`<h1>`)

**Trigger:** Called once on `window.onload`.

**Interface:**
```js
updateGreeting() → void
```

Reads `new Date().getHours()` and sets `#greeting.innerText` to one of three strings based on the hour range.

---

### 2. Theme Toggle Component

**DOM:** `<html class="dark">`, `#theme-icon` (`<span>`)

**Trigger:** User clicks the theme toggle button.

**Interface:**
```js
toggleTheme() → void
```

Toggles the `dark` class on `document.documentElement`, updates `#theme-icon` emoji, and writes `'dark'` or `'light'` to `localStorage['theme']`.

---

### 3. Focus Timer Component

**DOM:** `#timer-display`, `#timer-btn`

**State (module-level variables):**
```js
let timerSeconds = 1500;   // current countdown value
let timerInterval = null;  // setInterval handle, null when stopped
```

**Interface:**
```js
updateTimerDisplay() → void   // formats timerSeconds as MM:SS, writes to #timer-display
toggleTimer()        → void   // starts or stops the countdown interval
resetTimer()         → void   // clears interval, resets timerSeconds to 1500, updates display
```

When the countdown reaches zero, `toggleTimer`'s interval callback plays an audio alert via the Web Audio / `<Audio>` API, shows a browser `alert()`, and calls `resetTimer()`.

---

### 4. Task List Component

**DOM:** `#task-input`, `#task-list` (`<ul>`), `#duplicate-warning`

**State (module-level variable):**
```js
let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
// Task shape: { id: number, text: string, completed: boolean }
```

**Interface:**
```js
addTask()                      → void  // validates, deduplicates, pushes, renders
renderTasks()                  → void  // sorts, re-draws #task-list, saves to localStorage
toggleTask(id: number)         → void  // flips task.completed, renders
editTask(id: number, text: string) → void  // updates task.text, saves to localStorage
deleteTask(id: number)         → void  // filters tasks array, renders
```

Sorting rule: active tasks (`completed === false`) always appear before completed tasks.

---

### 5. Quick Links Component

**DOM:** `#links-container`, `#link-name`, `#link-url`

**State (module-level variable):**
```js
let links = JSON.parse(localStorage.getItem('links'))
         || [{ name: 'YouTube', url: 'https://youtube.com' }];
// Link shape: { name: string, url: string }
```

**Interface:**
```js
addLink()              → void  // validates, normalises URL, pushes, renders
renderLinks()          → void  // re-draws #links-container, saves to localStorage
deleteLink(index: number) → void  // splices links array, renders
```

URL normalisation: if the submitted URL does not start with `'http'`, `'https://'` is prepended.

---

## Data Models

### Task

```js
{
  id:        number,   // Date.now() at creation time — unique identifier
  text:      string,   // task description (non-empty, trimmed)
  completed: boolean   // false = active, true = completed
}
```

**localStorage key:** `'tasks'`  
**Serialisation:** `JSON.stringify(tasks)` / `JSON.parse(...)`

---

### Link

```js
{
  name: string,  // display label (non-empty, trimmed)
  url:  string   // fully-qualified URL (always starts with 'http')
}
```

**localStorage key:** `'links'`  
**Serialisation:** `JSON.stringify(links)` / `JSON.parse(...)`  
**Default value (first load):** `[{ name: 'YouTube', url: 'https://youtube.com' }]`

---

### Theme

**localStorage key:** `'theme'`  
**Values:** `'dark'` | `'light'`  
**Default (key absent):** dark mode (the `<html>` element carries `class="dark"` in the source)

---

### Timer State

The timer is ephemeral — it is not persisted to `localStorage`. On every page load the timer resets to `1500` seconds (`25:00`).

```js
timerSeconds: number   // 0–1500, decremented each second while running
timerInterval: number | null  // setInterval handle; null when stopped
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Greeting matches hour range

*For any* local hour value (0–23), `updateGreeting()` SHALL produce exactly one of the three greeting strings, and the string SHALL correspond to the correct hour range: "Good Morning!" for 0–11, "Good Afternoon!" for 12–17, and "Good Evening!" for 18–23.

**Validates: Requirements 1.1, 1.2, 1.3**

---

### Property 2: Valid task addition grows the list

*For any* task list state and any non-empty, non-duplicate task text, calling `addTask()` SHALL result in the task list containing exactly one more item than before, and that item SHALL have the submitted text and `completed: false`.

**Validates: Requirements 3.1, 6.1**

---

### Property 3: Whitespace-only and empty inputs are rejected

*For any* string composed entirely of whitespace characters (including the empty string), submitting it as a task SHALL leave the task list unchanged.

**Validates: Requirements 3.3**

---

### Property 4: Duplicate task submission is rejected

*For any* task list containing at least one task, submitting a text that matches any existing task text (case-insensitively) SHALL leave the task list unchanged.

**Validates: Requirements 3.2**

---

### Property 5: Task list always renders active tasks before completed tasks

*For any* task list containing a mix of active and completed tasks, after any render operation the active tasks SHALL all appear before the completed tasks in the rendered order.

**Validates: Requirements 5.2**

---

### Property 6: Task persistence round-trip

*For any* sequence of add, toggle, edit, and delete operations, serialising the `tasks` array to `localStorage` and then deserialising it SHALL produce an array that is deeply equal to the original.

**Validates: Requirements 6.1, 6.2**

---

### Property 7: Link persistence round-trip

*For any* sequence of add and delete link operations, serialising the `links` array to `localStorage` and then deserialising it SHALL produce an array that is deeply equal to the original.

**Validates: Requirements 8.2, 8.3**

---

### Property 8: URL normalisation is idempotent

*For any* URL string that already begins with `'http'`, the normalisation step SHALL leave the URL unchanged. *For any* URL string that does not begin with `'http'`, the normalisation step SHALL prepend exactly `'https://'` once, and applying normalisation a second time SHALL produce the same result.

**Validates: Requirements 7.2**

---

### Property 9: Timer display format

*For any* integer value of `timerSeconds` in the range [0, 1500], `updateTimerDisplay()` SHALL produce a string matching the pattern `MM:SS` where the seconds component is always zero-padded to two digits.

**Validates: Requirements 2.2, 2.3**

---

### Property 10: Theme toggle is an involution

*For any* current theme state, calling `toggleTheme()` twice in succession SHALL restore the `dark` class on `<html>` and the `localStorage['theme']` value to their original state.

**Validates: Requirements 9.1, 9.2**

---

## Error Handling

| Scenario | Handling |
|---|---|
| Empty task input | `addTask()` returns early; no DOM or storage change |
| Whitespace-only task input | `text.trim()` produces `''`; same early-return path |
| Duplicate task input | Shake animation on `#task-input`, `#duplicate-warning` shown for 2500 ms, no state change |
| Empty link name or URL | `addLink()` returns early; no DOM or storage change |
| URL missing protocol | `'https://'` prepended before storage |
| `localStorage` parse error | `JSON.parse` wrapped in `|| []` / `|| [default]` fallback; malformed data silently replaced with default |
| Timer audio playback failure | `new Audio(...).play()` is fire-and-forget; failure is silent (browser may block autoplay) |
| Timer reaches zero | Interval cleared, audio played, `alert()` shown, `resetTimer()` called |

---

## Testing Strategy

### Approach

Because the application is a single HTML file with no module system, tests must either:

1. **Extract pure functions** into a testable module (or use a test harness that loads the HTML), or
2. **Use a browser-based test runner** (e.g., Playwright, Cypress) that exercises the real DOM.

The recommended approach for unit and property tests is to extract the pure logic functions into a separate JS file (or use a `<script type="module">` pattern) so they can be imported by a test runner such as **Vitest** or **Jest**.

For property-based testing, use **fast-check** (JavaScript/TypeScript PBT library).

---

### Unit Tests

Unit tests cover specific examples, edge cases, and error conditions:

- `updateGreeting()` returns the correct string for boundary hours (0, 11, 12, 17, 18, 23)
- `addTask()` with empty string makes no change
- `addTask()` with whitespace-only string makes no change
- `addTask()` with duplicate text (different case) makes no change and triggers warning
- `renderTasks()` sorts active tasks before completed tasks
- `toggleTask()` flips `completed` and re-renders
- `editTask()` updates text and saves to localStorage
- `deleteTask()` removes the correct task by id
- `addLink()` with empty name or URL makes no change
- `addLink()` prepends `https://` when URL lacks protocol
- `deleteLink()` removes the correct link by index
- `toggleTheme()` toggles `dark` class and updates localStorage
- Timer initialises at `1500` on page load
- `resetTimer()` restores `timerSeconds` to `1500` and clears interval

---

### Property-Based Tests

Use **fast-check** with a minimum of **100 iterations** per property. Each test is tagged with a comment referencing the design property it validates.

**Tag format:** `// Feature: daily-flow-dashboard, Property {N}: {property_text}`

| Property | Generator inputs | Assertion |
|---|---|---|
| P1: Greeting matches hour range | `fc.integer({ min: 0, max: 23 })` | Output is one of three strings; correct string for range |
| P2: Valid task addition grows list | `fc.array(taskArb)` + `fc.string()` (non-empty, non-duplicate) | `tasks.length` increases by 1; new task has correct text and `completed: false` |
| P3: Whitespace/empty rejected | `fc.string()` filtered to all-whitespace | Task list unchanged |
| P4: Duplicate rejected | `fc.array(taskArb, {minLength:1})` + pick existing text with random casing | Task list unchanged |
| P5: Active before completed | `fc.array(taskArb)` with mixed `completed` values | After sort, all `completed: false` items precede all `completed: true` items |
| P6: Task persistence round-trip | `fc.array(taskArb)` | `JSON.parse(JSON.stringify(tasks))` deep-equals `tasks` |
| P7: Link persistence round-trip | `fc.array(linkArb)` | `JSON.parse(JSON.stringify(links))` deep-equals `links` |
| P8: URL normalisation idempotent | `fc.string()` | Already-`http` URLs unchanged; others get `https://` prepended exactly once |
| P9: Timer display format | `fc.integer({ min: 0, max: 1500 })` | Output matches `/^\d{2}:\d{2}$/`; seconds component zero-padded |
| P10: Theme toggle involution | Any initial theme state | Two calls restore original state |

---

### Integration / Smoke Tests

- Dashboard loads without JavaScript errors
- `localStorage` keys `tasks`, `links`, `theme` are written on first interaction
- Default YouTube link is present when `links` key is absent from localStorage
- Dark mode is applied by default when `theme` key is absent
- Light mode is applied on load when `localStorage['theme'] === 'light'`
- Timer audio fires when countdown reaches zero (manual / Playwright test)
