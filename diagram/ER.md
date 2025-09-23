erDiagram
  %% ================= RELATIONSHIPS =================
  USERS ||--o{ ADDRESSES : has
  USERS ||--|| CARTS : owns
  USERS ||--o{ ORDERS : places

  CATEGORIES ||--o{ PRODUCTS : contains
  PRODUCTS  ||--o{ PRODUCT_VARIANTS : has
  PRODUCT_VARIANTS ||--|| INVENTORY_ITEMS : stocks

  CARTS ||--o{ CART_ITEMS : contains
  PRODUCT_VARIANTS ||--o{ CART_ITEMS : item_of

  COUPONS ||--o{ CARTS : applies_to

  ORDERS ||--o{ ORDER_ITEMS : includes
  PRODUCT_VARIANTS ||--o{ ORDER_ITEMS : fulfills

  ORDERS ||--|| PAYMENTS : paid_by
  ORDERS ||--|| SHIPMENTS : fulfilled_by

  %% Order addresses (two roles)
  ORDERS ||--|| ADDRESSES : ships_to
  ORDERS ||--|| ADDRESSES : bills_to

  %% ================= ENTITIES + ATTRIBUTES =================
  USERS {
    UUID id PK
    string name
    string email UNIQUE
    string phone
    datetime created_at
  }

  ADDRESSES {
    UUID id PK
    UUID user_id FK
    string line1
    string line2
    string city
    string state
    string postal_code
    string country
    bool is_default
  }

  CATEGORIES {
    UUID id PK
    string name
    string slug UNIQUE
    string description
  }

  PRODUCTS {
    UUID id PK
    UUID category_id FK
    string name
    string slug UNIQUE
    string image
    string description
    decimal base_price
    bool is_customizable
  }

  PRODUCT_VARIANTS {
    UUID id PK
    UUID product_id FK
    string sku UNIQUE
    string size        %% e.g., 6", 8", 10"
    string flavor      %% vanilla, chocolate...
    decimal price
    int lead_time_days
    bool is_available
  }

  INVENTORY_ITEMS {
    UUID id PK
    string sku UNIQUE
    int quantity_on_hand
    int quantity_reserved
  }

  CARTS {
    UUID id PK
    UUID user_id FK
    decimal subtotal
    decimal discount
    decimal total
    datetime updated_at
  }

  CART_ITEMS {
    UUID id PK
    UUID cart_id FK
    UUID variant_id FK
    int quantity
    decimal unit_price
    decimal line_total
  }

  COUPONS {
    UUID id PK
    string code UNIQUE
    float percent_off
    decimal amount_off
    date valid_from
    date valid_to
    int usage_limit
  }

  ORDERS {
    UUID id PK
    UUID user_id FK
    string order_no UNIQUE
    string status           %% CREATED, CONFIRMED, ...
    decimal subtotal
    decimal discount
    decimal shipping
    decimal total
    datetime placed_at
    UUID ship_to_id FK
    UUID bill_to_id FK
  }

  ORDER_ITEMS {
    UUID id PK
    UUID order_id FK
    UUID variant_id FK
    int quantity
    decimal unit_price
    decimal line_total
    string sku
  }

  PAYMENTS {
    UUID id PK
    UUID order_id FK UNIQUE
    string txn_id
    string method         %% CARD, WALLET, COD
    string status         %% INITIATED, AUTHORIZED, CAPTURED, FAILED, REFUNDED
    decimal amount
    datetime paid_at
  }

  SHIPMENTS {
    UUID id PK
    UUID order_id FK UNIQUE
    string carrier
    string tracking_number
    datetime scheduled_for
    string status         %% SCHEDULED, OUT_FOR_DELIVERY, DELIVERED
  }
