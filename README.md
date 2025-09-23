# KK&CAKES – Detailed Diagram Walkthrough (Order Flow)

This guide explains, step-by-step, how a customer places an order on the **KK&CAKES** website.  
It ties together the four diagram types—**Flowchart**, **Sequence**, **Class**, and **ER**—so readers see *what* happens, *who* does it, and *where* it’s stored.

---

## Diagram Map (what each one is for)

| Diagram | Purpose | What to look for |
|---|---|---|
| **Flowchart** | High-level **user journey** and decisions | Browse → Cart → Login → Delivery/Pickup → Payment → Confirmation; branches for coupon, out-of-stock, payment failure |
| **Sequence** | **Service interactions** over time | Frontend ↔ Catalog/Cart/Auth/Inventory/Payment/Order/Shipping/Messaging; **reserve → commit** stock; **3-D Secure** |
| **Class** | **Domain model** (objects used by code) | User, Address, Product/Variant, Cart/CartItem, Coupon, Order/OrderItem, Payment, Shipment |
| **ER** | **Database schema** (tables & FKs) | Users/Addresses, Catalog (Categories/Products/Variants), Inventory, Cart/CartItems, Orders/OrderItems, Payments, Shipments, Coupons; ship-to/bill-to |

---

## 1) Flowchart — User Journey (Step-by-Step)

**Goal:** Show the **decisions** and **happy/error** paths end-to-end.

1. **Open Website** – Home/categories/search appear (entry point).
2. **Browse & View Product** – User opens details to see options, price, images, reviews.
3. **Choose Options & Add to Cart** – Size/flavor/qty selected; cart totals update.
4. **Open Cart & Review** – See items, quantities, subtotal, delivery estimate.
5. **Edit Cart?**  
   - **Yes:** Adjust qty/remove → totals refresh → back to cart.  
   - **No:** Continue.
6. **Coupon?**  
   - **Yes:** Validate code (date/usage/applicability).  
     - **Valid:** Discount applied → continue.  
     - **Invalid:** Stay in cart to fix.  
   - **No:** Continue.
7. **Proceed to Checkout** – System rechecks price/availability for freshness.
8. **Logged In?**  
   - **No:** Login or create account; retry on failure.  
   - **Yes:** Continue.
9. **Fulfillment: Delivery or Pickup**  
   - **Delivery:** Enter/select address + delivery window.  
   - **Pickup:** Choose store + time window.
10. **Stock Check & Reservation** – System verifies items and **temporarily reserves** stock (TTL).  
    - **OOS/lead time:** Show message → return to cart to adjust.
11. **Payment Method** – Choose card/wallet/(optional) cash on delivery.
12. **Payment Intent & 3-D Secure** – Create intent; bank challenge if required.
13. **Payment Outcome**  
    - **Failed:** Release reservation; user retries/changes method.  
    - **Succeeded:** **Commit reservation**, **create order**.
14. **Post-Purchase** – Schedule delivery/pickup, send **email/SMS**, show **Thank-You** page with order # and tracking.

> **Key idea:** *Reserve then commit* prevents overselling during payment.

---

## 2) Sequence — Service-to-Service Calls (Step-by-Step)

**Actors:** `Customer`, `Website(Frontend)`, `Catalog`, `Cart`, `Auth`, `Inventory`, `Payment`, `Order`, `Shipping`, `Email/SMS`.

1. **Browse Products** – `Website → Catalog` list products (price/stock).
2. **View Details** – `Website → Catalog` fetch product (options/images/stock).
3. **Add to Cart** – `Website → Cart` add item; get updated totals.
4. **Open Cart / Apply Coupon** – `Website ↔ Cart` get cart/apply coupon.
5. **Authenticate if Needed** – `Website → Auth` login/signup; receive session/JWT.
6. **Fulfillment Choice** – Website collects delivery address+window or pickup store+time.
7. **Reserve Inventory** – `Website → Inventory` reserve; receive **reservation token (TTL)**.
8. **Payment** – `Website → Payment` create intent; confirm/capture (3DS if needed).
9. **Handle Result**  
   - **Fail:** `Website → Inventory` release reservation; prompt retry.  
   - **Success:**  
     - `Website → Order` create order (cart, address/pickup, paymentId, reservationToken)  
     - `Order → Inventory` **commit** reservation  
     - `Order → Shipping` create shipment/pickup task (tracking/slot)  
     - `Order → Email/SMS` send confirmation
10. **Thank-You** – `Website → Customer` show order #, ETA, tracking link.

---

## 3) Class — Domain Objects Used in Code (Step-by-Step)

**Core classes:** `User`, `Address`, `Product`, `ProductVariant`, `InventoryItem`, `Cart`, `CartItem`, `Coupon`, `Order`, `OrderItem`, `Payment`, `Shipment`.

1. **Browsing** – `Product` (+ `ProductVariant`) populate UI with options & price.
2. **Carting** – `Cart` aggregates `CartItem`s; `applyCoupon(code)` adjusts totals.
3. **Auth** – `User` session established; default `Address` may be selected.
4. **Fulfillment** – `Address` used for shipping (or pickup selection held).
5. **Stock Hold** – `InventoryItem.reserve(qty)` during payment; `commit(qty)` on success; `release(qty)` on failure.
6. **Payment** – `Payment` captures method/status/amount and timestamps.
7. **Order Creation** – `Order` snapshots totals & addresses, links `OrderItem`s (unit price at purchase time).
8. **Shipment** – `Shipment` stores carrier/tracking/schedule & status.

> **Why snapshot prices in `OrderItem`?** Historical orders stay accurate even if catalog prices change later.

---

## 4) ER — Database Changes Through One Purchase (Step-by-Step)

**Tables touched:** `USERS`, `ADDRESSES`, `CATEGORIES`, `PRODUCTS`, `PRODUCT_VARIANTS`, `INVENTORY_ITEMS`, `CARTS`, `CART_ITEMS`, `COUPONS`, `ORDERS`, `ORDER_ITEMS`, `PAYMENTS`, `SHIPMENTS`.

1. **Cart Build-Up** – `CARTS` (one active per user) and multiple `CART_ITEMS` (variant, qty, unit_price, line_total).  
   Optional: `CARTS.coupon_id` set after coupon validation.
2. **Pre-Checkout Validation** – Totals recalculated; price/options rechecked.
3. **Reservation** – `INVENTORY_ITEMS.quantity_reserved` increments per SKU.
4. **Payment Attempt** – Gateway state only; no DB rows yet; reservation held.
5. **Payment Fail (branch)** – Release reservation (`quantity_reserved` decremented); user retries.
6. **Payment Success (happy path)** – Create:  
   - `ORDERS` (order_no, status=CONFIRMED, money totals, `ship_to_id`, `bill_to_id`)  
   - `ORDER_ITEMS` (snapshot of cart: variant_id, qty, unit_price, line_total, sku)  
   - `PAYMENTS` (txn_id, method, status, amount, paid_at; one per order in simple model)  
   - `SHIPMENTS` (carrier, tracking_number, scheduled_for, status)  
   Inventory: reserved qty moved to committed (reserved--, on_hand-- as per your policy).
7. **Notifications** – Not modeled here; add `NOTIFICATIONS`/`AUDIT_LOGS` if needed.

---

## 5) Error & Edge Cases

- **Invalid coupon** – Flowchart: back to Cart; Sequence: `Cart.applyCoupon` invalid; DB unchanged.  
- **Out of stock** – Flowchart: adjust cart; Sequence: `Inventory.reserve` fails; DB unchanged except transient reservation attempts.  
- **Payment failure** – Flowchart: retry; Sequence: `Payment` fails; DB: reservation released, no order/payment rows created.  
- **Address/pickup change** – Flowchart: reselect; Sequence: choice repeated; DB: only final addresses saved on `ORDERS`.

---

## 6) Traceability (Action → Services → Classes → Tables)

| User Action | Services (Sequence) | Classes (Code) | Tables (ER) |
|---|---|---|---|
| Add to cart | Website → Cart | `Cart`, `CartItem`, `ProductVariant` | `CARTS`, `CART_ITEMS` |
| Apply coupon | Website → Cart | `Cart.applyCoupon`, `Coupon` | `CARTS` (`coupon_id`) |
| Login | Website → Auth | `User` | — |
| Choose delivery | — | `Address` | `ADDRESSES` (existing rows) |
| Reserve stock | Website → Inventory | `InventoryItem.reserve` | `INVENTORY_ITEMS` (reserved++) |
| Pay | Website → Payment | `Payment` (domain-side) | — (gateway state) |
| Payment success | Website → Order | `Order`, `OrderItem`, `Payment`, `Shipment` | `ORDERS`, `ORDER_ITEMS`, `PAYMENTS`, `SHIPMENTS` |
| Commit stock | Order → Inventory | `InventoryItem.commit` | `INVENTORY_ITEMS` (reserved--, on_hand--) |

---

## Assumptions (tweak as needed)

- **One active cart per user** (guest carts optional).
- **One payment** and **one shipment** per order (split flows can be modeled as 1..* later).
- **One inventory row per SKU** (single-warehouse; add locations for multi-warehouse).
- **One coupon per cart** (use a junction for stacking).

**Common tweaks:**  
- *Guest checkout:* make `CARTS.user_id` nullable; capture `guest_email`.  
- *Split payments/shipments:* change to `ORDERS 1..* PAYMENTS` and `ORDERS 1..* SHIPMENTS`.  
- *Multi-warehouse:* add `INVENTORY_LOCATIONS` + `INVENTORY_BALANCES`.  
- *Custom cakes:* add `CUSTOM_ORDERS` linked `1:1` to `ORDER_ITEMS` for messages/designs.

---

## How to Render the Diagrams (optional)

- **Mermaid Live Editor**: paste each `.mmd` file to render.  
- **Lucid**: *Insert → Code/Diagram → Mermaid* then paste.  
- **CLI export** (PNG/PDF):
  ```bash
  npm i -g @mermaid-js/mermaid-cli
  mmdc -i diagrams/flowchart.mmd -o exports/flowchart.png
  mmdc -i diagrams/sequence.mmd  -o exports/sequence.png
  mmdc -i diagrams/class.mmd     -o exports/class.png
  mmdc -i diagrams/er.mmd        -o exports/er.png
