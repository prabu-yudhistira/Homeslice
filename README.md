# Homeslice — Website

A static website for **Homeslice Eatery & Bistro**, a Western restaurant in Surakarta, Indonesia.

- **Live site:** [prabu-yudhistira.github.io/Homeslice](https://prabu-yudhistira.github.io/Homeslice)
- **Deploy:** GitHub Pages (automatic on push to `main`)
- **Backend:** Shared Supabase project with the [POS cashier app](https://github.com/prabu-yudhistira/homeslice-pos)

---

## 📄 Site Structure

### `index.html` — Main Website

The landing page showcasing the restaurant. Built as a single HTML file with no build step.

**Sections (scroll-snapped, full-screen):**

| # | Section | Description |
|---|---------|-------------|
| 0 | **Hero** | Full-screen looping video background with tagline and CTA buttons |
| 1 | **Our Story** | Restaurant background and philosophy with collage photo |
| 2 | **The Space** | Photo gallery of the dining room atmosphere |
| 3–4 | **Signature Dishes** | Two featured dishes (full-screen, alternating layout) |
| 5 | **Best Dishes Carousel** | Horizontal scroll carousel of the full dish gallery |
| 6 | **Menu** | Full menu grouped by category (rendered from `_data/menu.json`) |
| 7 | **Find Us** | Address, hours, embedded Google Map, contact, social links |

**Features:**
- Scroll-snap navigation (each section = one full viewport height)
- Animated section entrances (fade/slide on scroll into view)
- Lightbox for dish photos (click any photo to zoom)
- Sticky navigation bar (hides on scroll, reappears on cursor near top)
- Navigation dot indicators (right side)
- Brown + gold color scheme matching restaurant branding

**CTAs:**
- 27 buttons / links pointing to `order.html` (nav bar, hero, signature dishes, menu section, location)
- 2 bare WhatsApp links kept for direct contact (not ordering)

**Technology:**
- Vanilla HTML/CSS/JS (no framework, no build step)
- Google Fonts: Cormorant Garamond (serif headings) + Montserrat (sans body)
- CSS custom properties for theming
- Menu content is baked in at build time from `_data/menu.json` via `scripts/build-menu.js`

---

### `order.html` — Ordering & Reservations

A single-page app for online orders and table reservations. Reads live data from Supabase on every load.

**Two tabs:**
1. **Order** — Browse the live menu, add items/variants to cart, pay a 50% deposit to confirm
2. **Reserve a Table** — Book a table with party size, contact, date/time, and pay Rp 50,000/person deposit

**Key Features:**

#### Menu Browsing
- Category filter chips — sourced live from the `categories` table
- Menu item cards with photo, name, description, and price
- **Variants** (e.g., burger sizes, pizza sizes) — fetched live from `menu_item_variants`; each variant shown as a selectable button
- Items with no price and no variants are hidden automatically
- Photos from Supabase Storage (uploaded via POS app) display correctly alongside static assets
- Sticky cart bar at the bottom (appears once 1+ item added), shows item count + running total

#### Order Checkout (cart sheet)
- Review cart, adjust quantities
- Customer details: name, phone, delivery address
- **Subtotal → Service → Tax → Total** breakdown (rates loaded live from `store_settings`)
- Deposit = 50% of total (calculated server-side)
- Order confirmation screen shows a generated order code (e.g., `W#0042`)

#### Table Reservation
- Guest name, party size, phone
- Preferred date + time picker
- Deposit = Rp 50,000 × party size
- Confirmation shows a booking reference code (e.g., `R#0012`)

#### Deposit & Payment Proof
- Deposit amount and bank account details shown (configured in `store_settings`)
- Two ways to send proof:
  - **WhatsApp button** — opens WhatsApp with pre-filled message (order code + deposit amount)
  - **Upload image** → saved to Supabase Storage (`proofs` bucket) → triggers a Telegram alert to the restaurant
- No payment gateway — manual bank transfer only

**Technology:**
- Vanilla HTML/CSS/JS (single file, no framework)
- Supabase JS SDK (anon key, no login required)
- Live menu + category data (reads on every page load — POS edits appear immediately)
- Tax/service rates loaded live from `store_settings`
- Responsive mobile-first design

**Data Flow:**
```
Customer visits order.html
  ↓
Fetches: menu_items, menu_item_variants, categories, store_settings
  ↓
Customer builds cart and fills checkout form
  ↓
Calls RPC: create_website_order()  OR  create_reservation()
  ↓
Supabase (SECURITY DEFINER — server-side validation)
  ↓  • Calculates tax/service/deposit server-side
     • Saves order or reservation with deposit_amount
     • Sends Telegram alert to restaurant staff
  ↓
Customer uploads payment proof (optional)
  ↓
Calls RPC: attach_payment_proof()
  ↓  • Stores proof URL on the order/reservation
     • Sends second Telegram alert with proof link
  ↓
POS app receives the order in real-time (Supabase Realtime)
  ↓
Staff prepare order / confirm reservation
```

---

## 🔧 Backend Integration

`order.html` connects to the same Supabase project used by the POS cashier app. The anon (public) key is embedded in the file — it is safe to expose since all writes go through `SECURITY DEFINER` RPCs that validate and compute values server-side.

### Tables read by `order.html`

| Table | Access | Purpose |
|-------|--------|---------|
| `menu_items` | Read (anon) | Dish names, prices, photos, descriptions |
| `menu_item_variants` | Read (anon) | Size/style options per dish |
| `categories` | Read (anon) | Menu category names and sort order |
| `store_settings` | Read (anon) | Tax %, service %, bank details, deposit config |

### Tables written by `order.html` (via RPC only)

| Table | Access | Purpose |
|-------|--------|---------|
| `orders` | Write via RPC | Customer food orders |
| `reservations` | Write via RPC | Table booking requests |
| Storage `proofs` bucket | Write (anon) | Payment proof image uploads |

### RPCs called by `order.html`

| Function | Purpose |
|----------|---------|
| `create_website_order(cart, customer, …)` | Validates cart, computes tax/service/deposit, saves order, notifies Telegram |
| `create_reservation(name, party_size, …)` | Saves reservation with deposit amount, notifies Telegram |
| `attach_payment_proof(kind, id, url)` | Stores upload URL on the record, pings Telegram again |

---

## 🚀 Deployment

**GitHub Pages** serves the repo directly — no build step for `order.html`. `index.html` is rebuilt automatically when `_data/menu.json` changes (see `.github/workflows/build-menu.yml`).

```bash
# Push any file change to trigger GitHub Pages deploy
git add index.html order.html
git commit -m "Update website"
git push origin main
```

---

## 🎨 Customization

### Colors & Fonts
CSS variables are defined in the `<style>` block of each file:

```css
:root {
  --esp: #160A06;   /* near-black background */
  --brn: #2C1810;   /* dark brown panels */
  --gld: #C9934A;   /* gold accent */
  --crm: #FAF6EF;   /* cream text */
  --ff: 'Cormorant Garamond', Georgia, serif;  /* display headings */
  --fs: 'Montserrat', system-ui, sans-serif;   /* body copy */
}
```

### Menu Management
`order.html` reads the **live `menu_items` table** on every page load — no regeneration needed. To add or edit dishes:

1. Open the **Homeslice POS app**
2. Go to **Manage Products**
3. Add / edit items → changes appear on the website on the next page load

`index.html` bakes the menu at deploy time from `_data/menu.json`. To update:

1. Edit `_data/menu.json`
2. Push to `main` → GitHub Actions runs `scripts/build-menu.js` and commits the updated `index.html` automatically

### Tax, Service & Deposit Rates
Stored in the `store_settings` table in Supabase. Update via the POS app **Settings** screen, or directly in the Supabase SQL Editor.

---

## 🔐 Security Notes

- The Supabase anon key in `order.html` is intentionally public — it only grants read access to public tables and write access through server-validated RPCs
- Tax, service charge, and deposit amounts are **computed server-side** — the client cannot submit a manipulated total
- Payment proof uploads are image-only with a 5 MB size limit
- No user accounts or passwords are stored by the website

---

## 📞 Contact

- Phone / WhatsApp: +62 878-1944-2998
- Instagram: [@homeslice.id](https://www.instagram.com/homeslice.id/)
- TikTok: [@homeslice.id](https://www.tiktok.com/@homeslice.id)
- Address: Jl. Kutilang No.8, Kerten, Laweyan, Surakarta

---

**Last updated:** June 2026
