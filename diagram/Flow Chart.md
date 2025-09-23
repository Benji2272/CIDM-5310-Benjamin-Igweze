flowchart TD
  %% ===== PHASE 1: BROWSING =====
  A[Open KK&CAKES Website] --> B[Browse Categories / Search]
  B --> C[View Product Details]
  C --> D{Choose Options: Size, Flavor, Icing, Qty}
  D -->|Add to Cart| E[Cart Updated]

  %% ===== PHASE 2: CART & CHECKOUT =====
  E --> F[Open Cart]
  F --> G{Edit Cart?}
  G -->|Yes| H[Update qty/remove items] --> F
  G -->|No| I{Have Coupon Code?}
  I -->|Yes| J[Apply Coupon / Validate] --> K{Coupon Valid?}
  I -->|No| L[Proceed to Checkout]
  K -->|No| F
  K -->|Yes| L[Proceed to Checkout]

  L --> M{Logged In?}
  M -->|No| N[Login / Create Account]
  N --> O{Success?}
  O -->|No| N
  O -->|Yes| P[Continue Checkout]
  M -->|Yes| P[Continue Checkout]

  %% ===== SHIPPING / DELIVERY =====
  P --> Q{Fulfillment Method?}
  Q -->|Delivery| R[Enter/Select Address]
  Q -->|Pickup| S[Choose Pickup Location]
  R --> T[Select Delivery Window]
  S --> T[Select Pickup Time Window]

  T --> U{All Items In Stock?}
  U -->|No| V[Show OOS/Lead Time / Split or Edit Cart] --> F
  U -->|Yes| W[Reserve Stock TTL]

  %% ===== PAYMENT =====
  W --> X[Select Payment Method: Card, Wallet, CoD]
  X --> Y[Create Payment Intent with amount and currency]
  Y --> Z{3-D Secure / Auth Required?}
  Z -->|Yes| ZA[Customer Completes Auth]
  ZA --> ZB{Auth Success?}
  ZB -->|No| X
  ZB -->|Yes| ZC[Capture Payment]
  Z -->|No| ZC[Capture Payment]

  ZC --> ZD{Payment Succeeded?}
  ZD -->|No| ZE[Release Reservation & Show Error] --> X
  ZD -->|Yes| ZF[Commit Reservation & Create Order]

  %% ===== POST-PURCHASE =====
  ZF --> ZG[Generate Order No. & Confirmation]
  ZG --> ZH[Schedule Delivery/Pickup]
  ZH --> ZI[Send Confirmation Email/SMS]
  ZI --> ZJ["Show Thank-You Page (Order No., ETA, Tracking)"]

  %% Notes
  classDef phase fill:#f7f7f7,stroke:#bbb,color:#111,stroke-width:1;
  class A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,ZA,ZB,ZC,ZD,ZE,ZF,ZG,ZH,ZI,ZJ phase;

  %% Optional footnote
  %% *CoD = Cash on Delivery (enable/disable per store policy)
