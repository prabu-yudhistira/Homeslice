# Homeslice — Website

A static website for **Homeslice Eatery & Bistro**, a Western restaurant in Surakarta, Indonesia.

- **Live site:** [homeslice-solo.com](https://homeslice-solo.com)
- **Deploy:** GitHub Pages (automatic on push to `main`)
- **Backend:** Shared Supabase project with the [POS cashier app](https://github.com/prabu-yudhistira/homeslice-pos)

---

## 📄 Site Structure

### `index.html` — Main Website
The landing page showcasing the restaurant.

**Sections:**
- **Hero** — Full-screen video background with call-to-action
- **Story** — Restaurant background and philosophy
- **Menu teaser** — Featured dishes with photos
- **Contact & Hours** — Location, phone, Instagram, WhatsApp links
- **Order CTAs** — 27 buttons linking to `order.html` (nav, inline, menu items)

**Features:**
- Responsive design (mobile-first)
- Smooth scroll sections with animations
- Light box for dish photos (click to zoom)
- Sticky navigation (hides/shows on scroll)
- Brown + gold color scheme matching restaurant branding

**Technology:**
- Vanilla HTML/CSS (no JS framework)
- Google Fonts: Cormorant Garamond (serif) + Montserrat (sans)
- CSS variables for theming

---

### `order.html` — Ordering & Reservations
A dedicated page for online orders and table reservations. Connects to the restaurant's Supabase database.

**Tabs:**
1. **Order** — Browse menu by category, pick items + variants, pay 50% deposit upfront
2. **Reserve a Table** — Book a table with party size, name, phone, preferred date/time + 50k IDR/person deposit

**Key Features:**

#### Menu Browsing
- Categories (Appetizers, Beef, Chicken, Pide & Pizza, etc.) — sourced live from `categories` table
- Menu items with photos from Supabase Storage (uploaded by POS staff)
- Variants (e.g., burger sizes, pizza sizes) — fetched live from `menu_item_variants`
- Add-ons (cheese, sauce, etc.) per dish
- Search/filter by category chips
- Sticky cart bar showing item count + total

#### Order Checkout
- **Cart sheet** — review items, adjust quantities, modify add-ons
- **Customer details** — name, phone, delivery address
- **Deposit calculation** — 50% of order total
- **Payment proof** — customer uploads transfer receipt to WhatsApp or via image upload
- **Order confirmation** — generates an order code, receipt PDF download
- **Tax/Service** — 10% service + tax applied server-side (per store_settings)

#### Table Reservation
- **Guest details** — name, party size, phone
- **Date & time picker** — select preferred slot
- **Deposit** — Rp 50,000 × party size
- **Payment proof** — same as orders (WhatsApp or upload)
- **Confirmation** — booking reference code

#### Payment Flow
- Shows bank details configured in store_settings
- Two payment proof methods:
  - **WhatsApp button** — pre-fills message with order code + amount
  - **Image upload** → saved to Supabase Storage (`proofs` bucket) → triggers server notification
- No payment gateway integration (manual transfer via bank)

**Technology:**
- Vanilla HTML/CSS/JavaScript (single-file app)
- Supabase SDK (anon key, no auth)
- Live menu + category syncing (reads `menu_items` table on every page load)
- Real-time order notifications to Telegram (via Supabase function)
- Responsive mobile-first design

**Data Flow:**
```
order.html
  ↓
Fetches: menu_items, menu_item_variants, categories, store_settings
  ↓
Customer fills cart + checkout
  ↓
calls: create_website_order() or create_website_reservation()
  ↓
Supabase RPC (SECURITY DEFINER)
  ↓
• Saves order/reservation with deposit_amount
• Sends Telegram alert to restaurant
• Stores payment_proof_url (if uploaded)
  ↓
POS app receives in real-time (Realtime subscriptions)
  ↓
Staff prepare order / confirm reservation
```

---

## 🔧 Backend Integration

Both pages read from the shared **Supabase project** (configured in order.html).

### Tables Used by `order.html`
| Table | Read/Write | Purpose |
|-------|-----------|---------|
| `menu_items` | Read (anon) | Dish names, prices, photos, descriptions |
| `menu_item_variants` | Read (anon) | Size/style options per dish |
| `categories` | Read (anon) | Menu category names & sort order |
| `store_settings` | Read (anon) | Tax %, service %, bank details, deposit amounts |
| `orders` | Write (anon, via RPC) | Customer orders |
| `reservations` | Write (anon, via RPC) | Table bookings |
| `payment_proofs` | Write (anon) | Upload receipts (optional) |

### Key RPCs (Supabase Functions)
- **`create_website_order()`** — Validates cart, calculates tax/service/deposit, saves order, notifies Telegram
- **`create_website_reservation()`** — Saves booking, notifies Telegram
- **`attach_payment_proof()`** — Stores upload URL, pings Telegram

---

## 🚀 Deployment

**GitHub Pages** — automatic on every push to `main`.

```bash
# Push changes to trigger deploy
git add index.html order.html
git commit -m "Update website"
git push origin main
```

**No build step** — all files are static HTML/CSS/JS.

---

## 🎨 Customization

### Colors & Fonts
Edit CSS variables in both files (`<style>` block, `:root`):

```css
:root {
  --esp: #160A06;      /* dark background */
  --brn: #2C1810;      /* brown panels */
  --gld: #C9934A;      /* gold accent */
  --crm: #FAF6EF;      /* cream text */
  --ff: 'Cormorant Garamond', Georgia, serif;  /* headings */
  --fs: 'Montserrat', system-ui, sans-serif;   /* body */
}
```

### Menu Management
`order.html` reads live from the `menu_items` table in Supabase. To add/edit dishes:
1. Open the POS app
2. Go to **Manage Products** screen
3. Add/edit items — changes appear on the website immediately (next page load)

### Bank Details, Tax, Service
Stored in `store_settings` table. Edit in Supabase SQL Editor:

```sql
UPDATE store_settings SET
  tax_percent = 10,
  service_percent = 5,
  bank_account = 'BRI 215301888333567',
  bank_name = 'Yosef Aria Yudo',
  deposit_order_percent = 50,
  deposit_reservation_amount = 50000
WHERE id = (SELECT id FROM store_settings LIMIT 1);
```

---

## 🔐 Security Notes

- **`order.html` uses Supabase anon key** — reads public menu, writes via SECURITY DEFINER RPCs
- **All writes are server-side validated** (tax, deposit calculation done on server, not client)
- **Payment proofs** — image-only, 5 MB limit, scanned for spam
- **Phone numbers** — stored unencrypted (acceptable for a small restaurant, but could be improved)

---

## 📞 Support / Issues

- **Website design** → See [HANDOFF.md](./HANDOFF.md)
- **Supabase backend** → Shared with [homeslice-pos](https://github.com/prabu-yudhistira/homeslice-pos)
- **POS sync** — Orders placed on the website appear in the POS app in real-time

---

**Last updated:** June 2026
