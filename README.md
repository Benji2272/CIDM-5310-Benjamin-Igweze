# CIDM-5310-Benjamin-Igweze
Business Intelligence 
# KK&CAKES — Order Flow Diagrams (Summary)

This repo documents **how a customer places an order** on the KK&CAKES website, from browsing to confirmation.  
It includes four diagram types—each shows the system from a different angle.

## Diagram Map (what each one is for)

| Diagram | Purpose | What to look for |
|---|---|---|
| **Flowchart** | High-level **user journey** and decisions | Browse → Cart → Login → Fulfillment (Delivery/Pickup) → Payment → Confirmation; branches for coupon, out-of-stock, payment failure |
| **Sequence** | **Interactions between components** over time | Frontend ↔ Services (Catalog, Cart, Auth, Inventory, Payment, Order, Shipping, Messaging); stock **reservation** during payment; **3-D Secure** step |
| **Class** | **Domain model** used by the app | User, Address, Product/Variant, Cart/CartItem, Coupon, Order/OrderItem, Payment, Shipment; responsibilities & relationships |
| **ER (Entity-Relationship)** | **Database schema** (tables & FKs) | Users/Addresses, Catalog (Categories/Products/Variants), Inventory, Cart/CartItems, Orders/OrderItems, Payments, Shipments, Coupons; ship-to/bill-to |

---

## One-paragraph summaries

### Flowchart (User Journey)
A customer **browses products**, views details, selects options (size/flavor), and **adds to cart**. They review the cart, optionally **apply a coupon**, then **checkout**. If not logged in, they **sign in or create an account**. They choose **Delivery** (address + time window) or **Pickup** (location + time). The system checks stock and **temporarily reserves** items. The customer **pays** (card/wallet; 3-D Secure if required). If payment fails, reservation is released and the user retries; if it succeeds, the order is **created and confirmed**, delivery/pickup is scheduled, and a **confirmation email/SMS** is sent.

### Sequence Diagram (System Interactions)
Shows calls among **Website (Frontend)** and backend services: **Catalog** (product data), **Cart**, **Auth**, **Inventory** (reserve → commit), **Payment** (intent + capture with 3DS), **Order**, **Shipping**, and **Messaging**. Key idea: **reserve inventory** before payment to stop oversells, then **commit** after payment succeeds; send notifications and show the **thank-you** page with order number.

### Class Diagram (Domain Model)
Defines the core objects the app manipulates: **User** (with **Address**es), **Product** and **ProductVariant** (what is sold), **InventoryItem** (stock per SKU), **Cart** with **CartItem**s (pre-purchase), **Coupon** (discounts), **Order** with **OrderItem**s (post-purchase snapshot), **Payment**, and **Shipment**. Relationships reflect the order flow (e.g., an Order has Items, is paid by a Payment, fulfilled by a Shipment).

### ER Diagram (Database)
Tables and foreign keys mirroring the domain: **USERS–ADDRESSES**, **CATEGORIES–PRODUCTS–PRODUCT_VARIANTS**, **INVENTORY_ITEMS** (by SKU), **CARTS–CART_ITEMS**, **ORDERS–ORDER_ITEMS**, **PAYMENTS**, **SHIPMENTS**, **COUPONS**. Each **ORDER** references **ship_to** and **bill_to** addresses. Prices are **snapshotted** in `ORDER_ITEMS` so history doesn’t change when catalog prices do.

---

## Key assumptions (can be changed)
- **One active cart per user** (simple UX; guest carts optional).
- **One payment** and **one shipment** per order (split flows can be modeled as 1..* later).
- **One inventory row per SKU** (single-warehouse; add locations for multi-warehouse).
- **One coupon per cart** (use a junction table if stacking is needed).

## Common tweaks
- **Guest checkout:** make `CARTS.user_id` nullable and capture `guest_email`.
- **Split payments/shipments:** change to `ORDERS 1..* PAYMENTS` and `ORDERS 1..* SHIPMENTS`.
- **Multi-warehouse:** add `INVENTORY_LOCATIONS` and `INVENTORY_BALANCES`.
- **Custom cakes:** add `CUSTOM_ORDERS` linked 1:1 to `ORDER_ITEMS` for messages/designs.

---

## How these fit together
1. **Flowchart** = the customer’s decision path.  
2. **Sequence** = the backend calls that make that path work.  
3. **Class** = the objects your code manipulates to implement those calls.  
4. **ER** = how those objects persist as tables and relationships.
