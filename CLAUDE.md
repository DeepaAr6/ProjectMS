# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

PMS is an IT PMO Kanban board, built as a demo/training tool for a fictitious bank. The whole app is `index.html`: markup, a `<style>` block and a `<script>` block. `PMSystem.txt` is an empty leftover.

Git remote: https://github.com/DeepaAr6/ProjectMS (branch `main`).

## Hard constraints (from the original brief; keep them)

- Vanilla HTML/CSS/JS in that one file: no frameworks, no CDN, no npm, no build step, no external fonts or images. Icons are inline SVG or Unicode.
- No persistence: no localStorage, sessionStorage, IndexedDB or cookies. A refresh resets to the seed data, and the UI says so.
- The only network call is FormSubmit (`notifyNewTask()`), wrapped in try/catch. It must never break the board.
- Don't use a real bank's logo or branding. The wordmark is just "IT PMO".
- No `alert()`/`confirm()`: validation errors are inline, deletes use the in-card "Delete? Yes / No", feedback goes to toasts.
- No `!important`. Colours and spacing come from the CSS custom properties on `:root`.

## Running and checking

Double-click `index.html`. There's no build, lint or test suite. To syntax-check the script:

```bash
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/check.js && node --check /tmp/check.js
```

FormSubmit emails only arrive when `FORMSUBMIT_ENDPOINT` is set to a real address, activated once via the confirmation email. They may also need the page served over http rather than file://. While the endpoint still holds the `YOUR_EMAIL@example.com` placeholder, `notifyNewTask()` throws before any request is made.

## Architecture

- `state = { tasks, filters, ui, nextId }` is the single source of truth. `ui` holds which card has its Move menu or Delete confirmation open, and whether a notification is in flight.
- Every change updates `state`, then calls `renderBoard()`. It rebuilds the card lists, column counts, summary strip and filter status, and it takes `{ focus: selector }` to restore keyboard focus after the DOM is replaced. Don't mutate card DOM anywhere else.
- `renderCard()` returns an HTML string. Every user-supplied value must go through `escapeHtml()`.
- Card buttons use one delegated click handler on `#board`, keyed by `data-action`. Drag and drop uses native HTML5 events on the columns; the column's `data-status` is the target status.
- Operations: `addTask()`, `moveTask()`, `deleteTask()`, `applyFilters()`, `showToast()`, `notifyNewTask()`.
- Config constants sit at the top of the script: `FORMSUBMIT_ENDPOINT`, `TASK_ID_PREFIX`, `EMAIL_SUBJECT_PREFIX`, and the timeouts.
- `STATUSES`, `PROJECTS`, `CATEGORIES` and `PRIORITIES` drive the form selects, the filters and validation. The four columns themselves are static markup, so adding a status means editing both `STATUSES` and the HTML.
- Seed due dates are offsets from today (`daysFromToday`), so the overdue demo cards stay overdue.

## Deployment

`.github/workflows/pages.yml` deploys `index.html` (only) to GitHub Pages on every push to `main`: https://deepaar6.github.io/ProjectMS/. It needs repo Settings → Pages → Source set to "GitHub Actions" once.
