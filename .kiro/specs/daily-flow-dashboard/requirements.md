# Requirements Document

## Introduction

The Daily Flow Dashboard is a self-contained, single-file productivity tool that runs entirely in the browser with no backend or build tooling. It gives users a unified view of their day through four widgets: a time-based greeting, a Pomodoro focus timer, a to-do list, and a quick-links bookmark panel. All state is persisted to `localStorage` so the dashboard is ready to use on every page load. The UI adapts to any screen size and supports both dark and light colour schemes.

---

## Glossary

- **Dashboard**: The Daily Flow Dashboard application delivered as a single `index.html` file.
- **Focus_Timer**: The countdown widget that runs a 25-minute Pomodoro session.
- **Task_List**: The to-do widget that stores and displays the user's tasks.
- **Task**: A single to-do item with a text label and a completion state.
- **Quick_Links**: The bookmark widget that stores named URL shortcuts.
- **Link**: A single bookmark entry consisting of a display name and a URL.
- **Theme**: The active colour scheme of the Dashboard — either `dark` or `light`.
- **localStorage**: The browser-native key-value store used for all persistence.
- **Greeting**: The time-sensitive salutation displayed in the page header.

---

## Requirements

### Requirement 1: Time-Based Greeting

**User Story:** As a user, I want to see a personalised greeting when I open the dashboard, so that the page feels contextually relevant to the time of day.

#### Acceptance Criteria

1. WHEN the Dashboard loads and the local hour is between 00:00 and 11:59, THE Dashboard SHALL display the greeting "Good Morning!".
2. WHEN the Dashboard loads and the local hour is between 12:00 and 17:59, THE Dashboard SHALL display the greeting "Good Afternoon!".
3. WHEN the Dashboard loads and the local hour is between 18:00 and 23:59, THE Dashboard SHALL display the greeting "Good Evening!".

---

### Requirement 2: Focus Timer

**User Story:** As a user, I want a 25-minute countdown timer, so that I can work in focused Pomodoro sessions without leaving the dashboard.

#### Acceptance Criteria

1. THE Focus_Timer SHALL initialise at 25 minutes (1500 seconds) on every page load.
2. WHEN the user activates the Start control, THE Focus_Timer SHALL begin counting down one second per second and display the remaining time in `MM:SS` format.
3. WHILE the Focus_Timer is running, THE Focus_Timer SHALL update the displayed time every second.
4. WHEN the user activates the Stop control while the timer is running, THE Focus_Timer SHALL pause the countdown and restore the Start control label.
5. WHEN the user activates the Reset control, THE Focus_Timer SHALL stop any active countdown and restore the display to `25:00`.
6. WHEN the countdown reaches zero, THE Focus_Timer SHALL play an audio alert, display a completion notification to the user, and reset to `25:00`.

---

### Requirement 3: To-Do List — Adding Tasks

**User Story:** As a user, I want to add tasks to my to-do list, so that I can track what I need to accomplish during the day.

#### Acceptance Criteria

1. WHEN the user submits a non-empty task text, THE Task_List SHALL add a new Task with that text and a `not completed` state.
2. WHEN the user submits a task whose text matches an existing Task text (case-insensitive), THE Task_List SHALL reject the submission, apply a shake animation to the input field, and display a duplicate-warning message for 2500 milliseconds.
3. IF the user submits an empty or whitespace-only string, THEN THE Task_List SHALL ignore the submission and make no changes.

---

### Requirement 4: To-Do List — Completing and Editing Tasks

**User Story:** As a user, I want to mark tasks complete and correct their text inline, so that I can keep my list accurate without extra steps.

#### Acceptance Criteria

1. WHEN the user toggles the checkbox of an active Task, THE Task_List SHALL mark that Task as completed and apply a strikethrough style to its text.
2. WHEN the user toggles the checkbox of a completed Task, THE Task_List SHALL mark that Task as active and remove the strikethrough style.
3. WHEN the user edits the inline text of a Task and the field loses focus, THE Task_List SHALL persist the updated text to localStorage.

---

### Requirement 5: To-Do List — Deleting and Sorting Tasks

**User Story:** As a user, I want to delete tasks I no longer need and see active tasks at the top, so that my list stays clean and prioritised.

#### Acceptance Criteria

1. WHEN the user activates the delete control on a Task, THE Task_List SHALL remove that Task from the list and from localStorage.
2. THE Task_List SHALL always render active Tasks before completed Tasks, regardless of the order in which they were added or completed.

---

### Requirement 6: To-Do List — Persistence

**User Story:** As a user, I want my tasks to survive a page reload, so that I do not lose my list between browser sessions.

#### Acceptance Criteria

1. WHEN any Task is added, toggled, edited, or deleted, THE Task_List SHALL write the current task array to localStorage under the key `tasks`.
2. WHEN the Dashboard loads, THE Task_List SHALL read the `tasks` key from localStorage and render all previously saved Tasks.

---

### Requirement 7: Quick Links — Adding and Opening Links

**User Story:** As a user, I want to save named URL shortcuts, so that I can reach my most-used sites with a single click.

#### Acceptance Criteria

1. WHEN the user submits a non-empty name and a non-empty URL, THE Quick_Links widget SHALL add a new Link and render it as a clickable anchor that opens in a new browser tab.
2. WHEN the submitted URL does not begin with `http`, THE Quick_Links widget SHALL prepend `https://` before storing the URL.
3. IF the user submits with an empty name or an empty URL, THEN THE Quick_Links widget SHALL ignore the submission and make no changes.

---

### Requirement 8: Quick Links — Deleting Links and Persistence

**User Story:** As a user, I want to remove links I no longer need and have my links available after a reload, so that my bookmark list stays relevant across sessions.

#### Acceptance Criteria

1. WHEN the user activates the delete control on a Link, THE Quick_Links widget SHALL remove that Link from the list and from localStorage.
2. WHEN any Link is added or deleted, THE Quick_Links widget SHALL write the current links array to localStorage under the key `links`.
3. WHEN the Dashboard loads, THE Quick_Links widget SHALL read the `links` key from localStorage and render all previously saved Links.
4. WHEN the Dashboard loads for the first time and no `links` key exists in localStorage, THE Quick_Links widget SHALL render a default Link with the name "YouTube" pointing to `https://youtube.com`.

---

### Requirement 9: Dark / Light Theme Toggle

**User Story:** As a user, I want to switch between dark and light colour schemes, so that I can use the dashboard comfortably in any lighting condition.

#### Acceptance Criteria

1. WHEN the user activates the theme toggle, THE Dashboard SHALL switch from the current Theme to the opposite Theme and update the toggle icon accordingly (🌙 for dark, ☀️ for light).
2. WHEN the Theme changes, THE Dashboard SHALL persist the selected Theme to localStorage under the key `theme`.
3. WHEN the Dashboard loads and the `theme` key in localStorage is `light`, THE Dashboard SHALL apply the light Theme on startup.
4. WHEN the Dashboard loads and no `theme` key exists in localStorage, THE Dashboard SHALL apply the dark Theme by default.

---

### Requirement 10: Responsive Layout

**User Story:** As a user, I want the dashboard to be usable on any screen size, so that I can access it from a desktop, tablet, or mobile device.

#### Acceptance Criteria

1. THE Dashboard SHALL render all widgets in a single-column layout on viewports narrower than the `md` Tailwind breakpoint (768 px).
2. THE Dashboard SHALL render the Focus Timer and Quick Links widgets in the first column and the Task List widget spanning two columns on viewports at or wider than the `md` breakpoint.

---

### Requirement 11: Self-Contained Delivery

**User Story:** As a user, I want to run the dashboard by opening a single HTML file, so that I have no dependency on a server, build tool, or internet connection beyond the initial CDN load.

#### Acceptance Criteria

1. THE Dashboard SHALL be delivered as a single `index.html` file with no external local dependencies.
2. THE Dashboard SHALL load Tailwind CSS from the official CDN and configure dark-mode support via the `class` strategy.
3. THE Dashboard SHALL implement all interactivity using vanilla JavaScript with no third-party JavaScript libraries.
4. THE Dashboard SHALL store all user data exclusively in localStorage with no network requests for data persistence.
