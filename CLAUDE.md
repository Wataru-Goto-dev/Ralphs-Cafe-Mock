# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ralph's Café Mock is a two-file, vanilla HTML/CSS/JS simulation of a café ordering and kitchen display system. No build step, no dependencies (only Google Fonts via CDN), no package manager.

## Running the Project

Open both files in separate browser tabs on the same browser to simulate the full workflow:

```bash
# Option 1: Direct file open
open cafe_order_ralphs.html   # Customer ordering interface
open cafe_staff_ralphs.html   # Kitchen/staff display

# Option 2: Serve locally (recommended — avoids some localStorage quirks)
python3 -m http.server 8000
# Then open:
#   http://localhost:8000/cafe_order_ralphs.html  (customer)
#   http://localhost:8000/cafe_staff_ralphs.html  (kitchen)
```

## Architecture

Two self-contained HTML files share state via `localStorage`:

| File | Role |
|------|------|
| `cafe_order_ralphs.html` | Customer interface — menu, cart, order history |
| `cafe_staff_ralphs.html` | Kitchen display — kanban board (Received → Preparing → Ready) |

### Shared Data Model

Orders are stored as JSON in `localStorage["ralphs_orders"]`. Order counter is in `localStorage["ralphs_counter"]`.

```js
// Order object shape
{
  num: "001",          // zero-padded 3-digit string
  table: "T3",
  items: [{ name, emoji, qty }],
  status: "received",  // received | preparing | ready | done
  time: <ISO timestamp>
}
```

### Cross-Tab Synchronization

Both files listen to the browser `storage` event to react to changes made by the other tab. The staff interface also polls `localStorage` every 3 seconds as a same-tab fallback.

### Customer Interface Key Functions

- `selectCat(cat)` — switch menu category
- `addToCart(id)` / `changeQty(id, delta)` — cart management
- `placeOrder()` — write new order to localStorage
- `renderHistory()` — refresh order history from shared state

### Kitchen Interface Key Functions

- `loadOrders()` — read and parse orders from localStorage
- `advance(num)` — progress an order through the status pipeline
- `buildCard(o)` — render one kanban card (includes urgency logic: ≥8 min = urgent)
- `render()` — redraw the full kanban board and stats bar

## Design System

Both files share a consistent upscale café aesthetic:
- **Colors:** dark green `#1B3A2D`, cream/ivory, gold `#B89A5A`
- **Fonts:** Playfair Display, EB Garamond, Cormorant Garamond (Google Fonts)
- **Customer layout:** 390px mobile mockup; **Staff layout:** full-width desktop
