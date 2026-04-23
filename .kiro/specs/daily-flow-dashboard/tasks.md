# Tasks

## Implementation Tasks

- [ ] 1. Set up test harness for vanilla JS logic
  - [ ] 1.1 Create a `tests/` directory alongside `index.html`
  - [ ] 1.2 Add `package.json` with Vitest and fast-check as dev dependencies
  - [ ] 1.3 Configure Vitest with jsdom environment for DOM testing
  - [ ] 1.4 Extract pure logic functions from `index.html` into a `src/dashboard.js` module that can be imported by tests
  - **Requirements:** 11.1, 11.3

- [ ] 2. Implement timer completion behaviour (audio alert + notification)
  - [ ] 2.1 Verify the `setInterval` callback in `toggleTimer()` plays audio via `new Audio(...).play()` when `timerSeconds` reaches 0
  - [ ] 2.2 Verify the callback shows a completion notification (currently `alert()`) and calls `resetTimer()`
  - [ ] 2.3 Write an example-based unit test for the zero-countdown path using fake timers
  - **Requirements:** 2.6

- [ ] 3. Write unit tests for greeting logic
  - [ ] 3.1 Test boundary hours: 0, 11 → "Good Morning!", 12, 17 → "Good Afternoon!", 18, 23 → "Good Evening!"
  - **Requirements:** 1.1, 1.2, 1.3

- [ ] 4. Write unit tests for timer logic
  - [ ] 4.1 Test `timerSeconds` initialises to 1500
  - [ ] 4.2 Test `updateTimerDisplay()` formats `0` as `"0:00"` and `1500` as `"25:00"`
  - [ ] 4.3 Test `resetTimer()` restores `timerSeconds` to 1500 and clears the interval
  - [ ] 4.4 Test `toggleTimer()` start/stop state transitions
  - **Requirements:** 2.1, 2.2, 2.4, 2.5

- [ ] 5. Write unit tests for task list logic
  - [ ] 5.1 Test `addTask()` with empty string makes no change
  - [ ] 5.2 Test `addTask()` with whitespace-only string makes no change
  - [ ] 5.3 Test `addTask()` with duplicate text (different case) makes no change
  - [ ] 5.4 Test `toggleTask()` flips `completed` flag
  - [ ] 5.5 Test `editTask()` updates task text and saves to localStorage
  - [ ] 5.6 Test `deleteTask()` removes the correct task by id
  - [ ] 5.7 Test `renderTasks()` sorts active tasks before completed tasks
  - **Requirements:** 3.1, 3.2, 3.3, 4.1, 4.2, 4.3, 5.1, 5.2

- [ ] 6. Write unit tests for quick links logic
  - [ ] 6.1 Test `addLink()` with empty name makes no change
  - [ ] 6.2 Test `addLink()` with empty URL makes no change
  - [ ] 6.3 Test `addLink()` prepends `https://` when URL lacks protocol
  - [ ] 6.4 Test `addLink()` leaves URL unchanged when it already starts with `http`
  - [ ] 6.5 Test `deleteLink()` removes the correct link by index
  - [ ] 6.6 Test default YouTube link is present when `links` key is absent from localStorage
  - **Requirements:** 7.1, 7.2, 7.3, 8.1, 8.4

- [ ] 7. Write unit tests for theme toggle
  - [ ] 7.1 Test `toggleTheme()` adds/removes `dark` class on `<html>`
  - [ ] 7.2 Test `toggleTheme()` writes correct value to `localStorage['theme']`
  - [ ] 7.3 Test dark mode is applied by default when `theme` key is absent
  - [ ] 7.4 Test light mode is applied on load when `localStorage['theme'] === 'light'`
  - **Requirements:** 9.1, 9.2, 9.3, 9.4

- [ ] 8. Write property-based tests using fast-check
  - [ ] 8.1 Property 1 — Greeting matches hour range: `fc.integer({min:0, max:23})` → correct greeting string
  - [ ] 8.2 Property 2 — Valid task addition grows list: non-empty non-duplicate text → list length +1, new task has correct text and `completed: false`
  - [ ] 8.3 Property 3 — Whitespace/empty inputs rejected: all-whitespace strings → task list unchanged
  - [ ] 8.4 Property 4 — Duplicate task rejected: existing task text with random casing → task list unchanged
  - [ ] 8.5 Property 5 — Active tasks before completed: mixed task arrays → after sort, all active precede all completed
  - [ ] 8.6 Property 6 — Task persistence round-trip: arbitrary task arrays → `JSON.parse(JSON.stringify(tasks))` deep-equals original
  - [ ] 8.7 Property 7 — Link persistence round-trip: arbitrary link arrays → `JSON.parse(JSON.stringify(links))` deep-equals original
  - [ ] 8.8 Property 8 — URL normalisation idempotent: arbitrary URL strings → correct prefix behaviour; applying twice is same as once
  - [ ] 8.9 Property 9 — Timer display format: integers in [0,1500] → output matches `/^\d+:\d{2}$/` with zero-padded seconds
  - [ ] 8.10 Property 10 — Theme toggle involution: any initial theme state → two toggles restore original state
  - **Requirements:** 1.1–1.3, 3.1–3.3, 4.1–4.2, 5.2, 6.1–6.2, 7.2, 8.2–8.3, 9.1–9.2, 2.2

- [ ] 9. Smoke / integration checks
  - [ ] 9.1 Verify `index.html` loads without JavaScript console errors
  - [ ] 9.2 Verify `localStorage` keys `tasks`, `links`, `theme` are written after first interaction
  - [ ] 9.3 Verify default YouTube link renders when `links` key is absent
  - [ ] 9.4 Verify responsive layout at narrow (<768 px) and wide (≥768 px) viewports (manual or Playwright)
  - **Requirements:** 8.4, 9.3, 9.4, 10.1, 10.2, 11.1–11.4
