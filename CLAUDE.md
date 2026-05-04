# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file, mobile-responsive food tracking web app. No build step, no dependencies, no package manager — everything is in `index.html` as inline `<style>` and `<script>` blocks.

## Running the app

Open `index.html` directly in a browser:
```
open "/Users/kalyanramanathan/Desktop/Claude/Food Analysis/index.html"
```

No server needed. The app runs from a `file://` URL.

## Architecture

All code lives in `index.html`:

- **CSS** — CSS custom properties for the design system (colors, radius, tab height). Mobile-first, max-width 480px, dark theme.
- **HTML** — Four `<section class="screen">` elements (analyze, log, stats, advice) toggled via `.active`. Fixed bottom tab bar (`#tab-bar`) with 64px height; screens pad `padding-bottom` to clear it.
- **JS** — Plain ES2020, no framework, `'use strict'`.

### Data layer (localStorage)
| Key | Contents |
|-----|----------|
| `foodlog_entries` | JSON array of entry objects |
| `foodlog_settings` | API key + daily macro/calorie goals |
| `foodlog_advice_cache` | Last AI advice + timestamp (6-hour TTL) |

Entry object shape: `{ id, timestamp, date (YYYY-MM-DD), imageDataURL, name, description, calories, protein_g, carbs_g, fat_g, serving_note, confidence }`.

### Claude API integration
- Direct browser → `https://api.anthropic.com/v1/messages` (requires `anthropic-dangerous-direct-browser-access: true` header)
- Model: `claude-sonnet-4-6`
- **Food analysis**: sends a base64-encoded image (compressed to max 1024px / JPEG 0.82 via canvas) and requests JSON-only response with `{ name, description, calories, protein_g, carbs_g, fat_g, serving_note, confidence }`
- **Advice**: sends a plain-text 7-day log summary, receives markdown prose
- API key is stored in `foodlog_settings` and entered by the user via the Settings modal

### Key JS functions
- `compressImage(file)` — canvas resize + JPEG encode, returns `{ base64, mediaType, dataURL }`
- `analyzeFood()` — calls Claude vision, parses JSON response (with regex fallback), calls `renderAnalysisResult()`
- `fetchAdvice()` — builds log summary via `buildLogSummary()`, calls Claude, saves to cache
- `renderMarkdown(text)` — minimal renderer: `##` → `<h3>`, `**x**` → `<strong>`, `-` → `<ul><li>`
- `renderBarChart(entries, metric)` — pure CSS/JS 7-day bar chart, no library; bar heights computed proportionally against max day value
- `switchTab(name)` — shows/hides screens, triggers per-tab render functions

### Design tokens
```
--bg: #0f0f1a       --card: #242440      --primary: #f5a623
--protein: #4a90d9  --carbs: #27ae60     --fat: #e67e22    --cal: #e74c3c
```
Progress bars use a CSS custom property pattern: `--fill` on the element + `::after { width: var(--fill) }`.
