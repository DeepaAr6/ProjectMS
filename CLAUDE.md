# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

PMS is an IT PMO Kanban board, built as a demo/training tool for a fictitious bank. The whole app is `index.html`: markup, a `<style>` block and a `<script>` block. `PMSystem.txt` is an empty leftover.

Hard constraints (from the original brief, keep them):
- Vanilla HTML/CSS/JS only: no frameworks, no CDN, no npm, no build step, no external fonts or images.
- No persistence: no localStorage, sessionStorage, IndexedDB or cookies. A refresh resets to the seed data, and the UI says so.
- The only network call is FormSubmit (`notifyNewTask()`). It must never break the board.
- Don't use a real bank's logo or branding. The wordmark is just "IT PMO".

## Running

Double-click `index.html`. There's no build, lint or test suite. FormSubmit emails only arrive when `FORMSUBMIT_ENDPOINT` is set to a real address (activate it once from the confirmation email). They may also need the page served over http rather than file://.

## Architecture

- `state = { tasks, filters, ui, nextId }` is the single source of truth. `ui` holds which card has its Move menu or Delete confirmation open.
- Every change updates `state`, then calls `renderBoard()`, which rebuilds the card lists, counts, summary strip and filter status. `renderCard()` returns an HTML string, and every user string goes through `escapeHtml()`.
- Operations: `addTask()`, `moveTask()`, `deleteTask()`, `applyFilters()`, `showToast()`, `notifyNewTask()`.
- The config constants at the top of the script are `FORMSUBMIT_ENDPOINT`, `TASK_ID_PREFIX` and `EMAIL_SUBJECT_PREFIX`.
- Seed due dates are offsets from today, so the overdue demo cards stay overdue.
