---
name: friendly-ui
description: Redesigns a Vue 3 application's UI into a happy, friendly interface with a vertical navigation sidebar, warm colors, and fun fonts. Use this skill when the user asks to make the UI friendlier, happier, more fun, or wants a sidebar navigation layout.
---

# Friendly UI Redesign Skill

Redesign the Vue 3 app into a warm, friendly, and approachable interface. The result should feel like it was designed by someone who genuinely enjoys their job — not a bland enterprise dashboard.

## What This Skill Does

1. Replaces the horizontal top navigation with a fixed **vertical sidebar** on the left
2. Applies a **warm, happy color palette** (soft backgrounds, vibrant accents)
3. Imports and applies **fun, friendly fonts** from Google Fonts
4. Updates **card, badge, table, and button styles** to feel welcoming
5. Adjusts **layout** so the main content area sits to the right of the sidebar

---

## Step 1 — Read Before Changing

Read these files first to understand the current structure:
- `client/src/App.vue` — global layout, nav, and styles
- `client/src/main.js` — router and app setup
- One or two view files to understand the page structure

---

## Step 2 — Design System to Apply

### Fonts (Google Fonts)
Add this `<link>` to `client/index.html` in the `<head>`:
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&family=Fredoka+One&display=swap" rel="stylesheet">
```

- **Body/UI font**: `'Nunito', sans-serif` — rounded, friendly, highly legible
- **Display/heading font**: `'Fredoka One', cursive` — playful, bold, fun

### Color Palette
```
Background:     #f5f0ff   (soft warm purple-white)
Sidebar bg:     linear-gradient(180deg, #7c3aed 0%, #4f46e5 60%, #2563eb 100%)  (vivid purple-to-blue gradient)
Sidebar text:   #e0d9ff   (pale lavender)
Sidebar active: #ffffff   (white text)
Sidebar active bg: rgba(255,255,255,0.22)  (frosted glass highlight)
Sidebar hover:  rgba(255,255,255,0.12)     (subtle hover glow)

Accent primary: #7c3aed   (vibrant violet)
Accent success: #059669   (bold emerald)
Accent warning: #d97706   (strong amber)
Accent danger:  #e11d48   (vivid rose-red)
Accent info:    #0891b2   (bold cyan)
Accent sky:     #0284c7   (strong sky blue — for restocking)

Card bg:        #ffffff
Card border:    #ede9fe   (violet-tinted border)
Card shadow:    0 4px 28px rgba(124, 58, 237, 0.12)
Card radius:    18px

Text primary:   #2e1065   (deep violet)
Text secondary: #6b7280
```

### Sidebar Layout Spec
```
Sidebar width:  240px (fixed, full height)
Sidebar:        position fixed, left 0, top 0, bottom 0
Main content:   margin-left: 240px
```

---

## Step 3 — Update `client/index.html`

Add the Google Fonts link tags inside `<head>`. Read the file first.

---

## Step 4 — Delegate App.vue Rewrite to vue-expert

**MANDATORY: Use the `vue-expert` subagent** to rewrite `client/src/App.vue`.

Give the vue-expert these exact instructions:

> Rewrite `client/src/App.vue` to replace the horizontal top navigation with a fixed vertical sidebar on the left. Keep all existing functionality (router-links, FilterBar, ProfileMenu, LanguageSwitcher, ProfileDetailsModal, TasksModal, tasks logic) but restructure the layout as follows:
>
> **Layout structure:**
> ```
> .app
>   .sidebar (fixed left, 240px wide, full height)
>     .sidebar-logo (company name + subtitle stacked)
>     nav.sidebar-nav (router-links stacked vertically)
>     .sidebar-footer (LanguageSwitcher + ProfileMenu)
>   .main-wrapper (margin-left: 240px, flex-col)
>     FilterBar
>     main.main-content
>       router-view
>     ProfileDetailsModal
>     TasksModal
> ```
>
> **Sidebar nav link icons** — add a simple inline SVG icon before each label:
> - Overview: grid/squares icon
> - Inventory: box/package icon
> - Orders: clipboard/list icon
> - Finance: chart/coin icon
> - Demand Forecast: trending-up arrow icon
> - Restocking: refresh/restock icon
> - Reports: document/report icon
>
> **Styles to apply** (replace ALL existing styles):
> - Font: `'Nunito', sans-serif` everywhere; headings use `'Fredoka One', cursive`
> - Body background: `#f5f0ff`
> - Sidebar: `background: linear-gradient(180deg, #7c3aed 0%, #4f46e5 60%, #2563eb 100%)`, text `#e0d9ff`, active link `background: rgba(255,255,255,0.22); color: #fff`, hover `background: rgba(255,255,255,0.12)`
> - Sidebar nav links: `display: flex; align-items: center; gap: 0.75rem; padding: 0.75rem 1.25rem; border-radius: 12px; margin: 0.2rem 0.75rem; font-weight: 700; font-size: 0.938rem; text-decoration: none; transition: all 0.2s`
> - Sidebar SVG icons: `width: 20px; height: 20px; flex-shrink: 0`
> - Cards: `border-radius: 18px; border: 1px solid #ede9fe; box-shadow: 0 4px 28px rgba(124,58,237,0.12)`
> - `.stat-value` color: `#2e1065`
> - `.stat-card.success .stat-value`: `#059669`
> - `.stat-card.warning .stat-value`: `#d97706`
> - `.stat-card.danger .stat-value`: `#e11d48`
> - `.stat-card.info .stat-value`: `#0891b2`
> - `.stat-card.restocking .stat-value`: `#0284c7`
> - `.badge.success`: `background: #d1fae5; color: #065f46`
> - `.badge.warning`: `background: #fef3c7; color: #92400e`
> - `.badge.danger`: `background: #ffe4e6; color: #9f1239`
> - `.badge.info`: `background: #cffafe; color: #164e63`
> - `.badge.restocking`: `background: #e0f2fe; color: #075985`
> - `.badge.increasing`: `background: #d1fae5; color: #065f46`
> - `.badge.decreasing`: `background: #ffe4e6; color: #9f1239`
> - `.badge.stable`: `background: #ede9fe; color: #5b21b6`
> - `.page-header h2`: `font-family: 'Fredoka One', cursive; font-size: 2rem; color: #2e1065`
> - `th`: background `#f5f0ff`, color `#7c3aed`, border-bottom `2px solid #ede9fe`
> - `tbody tr:hover`: background `#f5f3ff`

---

## Step 5 — Update FilterBar Styles (if needed)

The FilterBar sits below the sidebar in the main-wrapper. Update its background to `#ffffff` with a bottom border `1px solid #ede9fe` to match the new palette. Delegate to vue-expert if modifying the `.vue` file.

---

## Step 6 — Verification

After making changes:
1. Check `client/index.html` has the Google Fonts link
2. Confirm `App.vue` has the sidebar layout structure
3. Confirm font-family references use `'Nunito'` and `'Fredoka One'`
4. Confirm sidebar is `position: fixed` so it stays while scrolling
5. Confirm `main-wrapper` has `margin-left: 240px`

The app should feel warm, rounded, and inviting — like a friendly coworker built it.
