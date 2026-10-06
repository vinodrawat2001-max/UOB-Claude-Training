# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This folder holds training material for "Agentic AI Applications with Claude Code" (the PDF) plus one exercise app, [index.html](index.html): an "IT PMO | Project Delivery Board", a Kanban-style task board. It is not a git repository and has no build system, package manager, linter or tests. Open the file directly in a browser to run it.

## Architecture of index.html

Everything lives in one file: inline `<style>` (design tokens in `:root`, then sections commented by area), static markup, and one inline `<script>` at the bottom. There are no external dependencies.

- **State**: a single `state = { tasks, filters }` object. It is in-memory only, with no persistence. `seedTasks()` generates demo tasks with dates relative to today via `dateOffset()`, so data resets on every reload.
- **Rendering**: `renderBoard()` rebuilds the whole board and the summary strip from `state` using template strings. `renderCard()` produces each card. All user-supplied text must go through `escapeHtml()`.
- **Events**: the board uses delegation (`handleBoardClick`, `handleBoardChange` for the move `<select>`, and the drag handlers `handleDragStart/Over/Drop`). Cards carry `data-action` and `data-task-id` attributes. Add new card actions by extending `handleBoardClick` and `renderCard`.
- **Constants**: `STATUSES`, `PROJECTS` and `CATEGORIES` drive the columns, the filters and the form `<select>` options (`populateOptions()`). `FIELD_IDS` plus the `task<Field>` / `error-<field>` element id convention link the form validation (`validateTaskForm`, `displayErrors`) to the DOM. A new form field must follow that naming.
- **New-task flow**: `addTask` validates, pushes to `state.tasks`, re-renders and shows a toast. It then calls `notifyNewTask`, which POSTs to the FormSubmit endpoint `FORMSUBMIT_ENDPOINT` (currently the placeholder `YOUR_EMAIL@example.com`). FormSubmit needs a one-time email confirmation before delivery starts, as the comment above the script notes.
- **Accessibility**: the app uses `.sr-only` text, a toast live region, `focus-visible` outlines and a native `<dialog>` for task entry. Keep these when changing the UI.
