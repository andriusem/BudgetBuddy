
## 1 · Purpose & Scope

* **Audience** Single user (the owner) – no registration or sharing in v 1.
* **Goal** Track expenses, balances and category stats in **EUR**, in a dark-mode, swipe-driven PWA that mirrors the supplied Figma screens **pixel-perfectly**.
* **Later** Add GPT-4o chat that can answer questions using every row in the DB.

---

## 2 · Primary UI Screens (refer to the Figma images)

| Route                                       | Name                                  | Core widgets                                                                                                                     | Navigation notes                                                                                                      |
| ------------------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `/`                                         | **Home**                              | Four coloured balance tiles + doughnut showing **current balance share** of each tile. Inner label = *sum of all four balances*. | Swipe left (touch) or hover-chevrons (desktop) to `/categories`.                                                      |
| `/categories`                               | **Category bar stack**                | Horizontal bars for the **10 merged buckets** (Food, Housing & Utilities, etc.) with € amount on each bar.                       | Tap a bar → drill to its sub-categories (e.g. Food → Groceries / Meals-out). Double-tap a sub-bar → back to top list. |
| `/weeks`                                    | **Weekly bar chart**                  | Bars for Week 1–Week 5 totals (current month).                                                                                   | Swipe left again to `/trend`.                                                                                         |
| `/trend`                                    | **Week-over-week line chart**         | Two lines: “current week” vs “previous week”. X-axis labels: Week 1, 2, 3, 4, 5. Y-axis: € (hundreds).                           |                                                                                                                       |
| `/expense/new` *(modal route)*              | **Add Expense**                       | Source is picked first by hitting the **“+” icon**, then choosing a tile. Form fields:                                           |                                                                                                                       |
|  • Merchant Name (text)                     |                                       |                                                                                                                                  |                                                                                                                       |
|  • Amount (€ only, numeric)                 |                                       |                                                                                                                                  |                                                                                                                       |
|  • Week (Week 1-5 dropdown) – no date field |                                       |                                                                                                                                  |                                                                                                                       |
|  • Sub-category (dropdown, hard-coded list) | Save ➜ `/expense/:id` (receipt view). |                                                                                                                                  |                                                                                                                       |
| `/expense/:id`                              | **Receipt card**                      | Details of the saved expense. Swipe right ↔ opens the same form pre-filled for editing.                                          |                                                                                                                       |
| `/history`                                  | **History list**                      | Scrollable list of all transactions (grouped by week). Tap row ➜ `/expense/:id`.                                                 |                                                                                                                       |
| `wallet edit` (no route)                    | **Tile quick-edit**                   | Tap **wallet icon** → tap a tile → an in-place input lets you overwrite its current balance. Tapping wallet again saves.         |                                                                                                                       |

**Gestures**

* Horizontal swipe = dashboard paging (only on the four main charts).
* Hover→desktop arrow buttons appear full-height at screen edges and mimic swipe.

**Theme** Dark-mode **only** – no toggle.

---

## 3 · Data Model (plain-English version)

| Entity               | Fields / Notes                                                                                                                                                                    |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Source** (4 tiles) | `id`, `name`, `color`, `sort_order`, *(balance is derived)*                                                                                                                       |
| **Category**         | 10 top-level buckets + hard-coded children (e.g. Food → Groceries / Meals-out).                                                                                                   |
| **Transaction**      | `id`, `kind` *(expense, income, adjustment, transfer)*, `title`, `amount` (positive), `source_id`, `target_id` *(for transfers only)*, `category_id`, `week` (1-5), `created_at`. |

### Ledger-is-truth rules

* Tile balance **always** equals the sum of transactions applied to that source.
* Wallet quick-edit writes an **“adjustment”** transaction (kind = adjustment, positive or negative).
* Salary on the 29ᵗʰ of each month is logged as an **income** transaction.
* Money moved between tiles uses one **transfer** row: `source_id` (from) + `target_id` (to).
* Expense rows always carry a `source_id` and a `category_id` (sub-category).
* Weeks only – no explicit date field required.

---

## 4 · Hard-coded 10 Buckets & Sub-categories

| Bucket                           | Sub-categories used in charts/forms                        |
| -------------------------------- | ---------------------------------------------------------- |
| **1. Housing & Utilities**       | Bills, phone, Revolut fee, DIY gas etc.                    |
| **2. Food**                      | Groceries, Meals-out                                       |
| **3. Health & Wellness**         | Health-insurance, Dentist, Psychologist, Supplements       |
| **4. Transport (Car)**           | Car gas, Maintenance                                       |
| **5. Travel & Accommodation**    | Plane tickets, Airbnb                                      |
| **6. Subscriptions & Bank Fees** | ChatGPT, Claude, YouTube, Windsurf, Gemini, overdraft fees |
| **7. Clothing & Accessories**    | Clothing, Haircuts                                         |
| **8. Personal-care & Household** | Toiletries, cat food, printer ink, plant soil              |
| **9. Recreational Substances**   | Alcohol, Tobacco, Weed                                     |
| **10. Gifts & Charity**          | Presents                                                   |

*(List is fixed; no UI for adding new categories in v 1.)*

---

## 5 · Tech Stack

| Layer                 | Tooling                                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| Front-end             | **React + Vite** → smallest bundles, ES modules                                                          |
| Styling               | **Tailwind CSS** + **shadcn/ui** components                                                              |
| Animations            | **Framer Motion** (swipes & drill-down)                                                                  |
| Data fetching & cache | **@supabase-js** + **TanStack Query** (with IndexedDB persistence, purge > 50 MB or > 180 days)          |
| Charts                | **Recharts** (pie, bar, line)                                                                            |
| Back-end              | **Supabase** Postgres 16 + Row-Level Security ready for multi-user (but disabled in v 1)                 |
| Exports               | Supabase **Edge Function** generates full-DB CSV (sources, categories, transactions) and streams to user |
| PWA                   | Vite PWA plugin + Capacitor for install prompt (no biometric lock in v 1)                                |
| Analytics             | **None** (privacy first; can add self-hosted PostHog later)                                              |
| Future GPT\*\*        | Edge Function proxy to OpenAI; will query the full `transactions` table (all rows).                      |

---

## 6 · Non-functional Requirements

* **Offline-first:** must create/edit transactions offline and sync when back online.
* **Performance:** All dashboards must render < 150 ms on mid-range Android/iOS.
* **Bundle size:** aim for < 250 kB gzipped JS on first load.
* **Accessibility:** basic screen-reader labels; colour palette WCAG AA contrast.
* **Currency:** input & display **€**, no multi-currency.
* **No alerts / push notifications** in v 1.

---

## 7 · Open Items (need no further input)

* **Security:** none for v 1 (no auth / biometric gate).
* **Budget alerts:** deliberately omitted.
* **Receipts/photos:** not required.
* **Theme switch:** dark-only.

---

## 8 · Delivery Milestones (suggested)

1. **DB & API** Tables, RLS off, Edge CSV function ✔
2. **Tile / balance engine** Ledger maths, wallet quick-edit, transfers ✔
3. **Add & edit expense flow** Form → receipt → history list ✔
4. **Dashboards** Pie, category bar + drill, weekly bar, trend line ✔
5. **Offline & PWA shell** Service Worker, install prompt ✔
6. **Polish & Figma pixel-fit** Colours, spacing, hover arrows, swipe physics ✔
7. **Full-DB CSV export** Edge zip streaming, tested on mobile ✔

*(GPT integration is **out of Milestone 1**; schedule after v 1 goes live.)*

---

### End of brief.  All developer questions should be answerable by this document; if new edge-cases arise, loop back to the product owner.
