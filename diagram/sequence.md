sequenceDiagram
    autonumber
    actor Customer
    participant Web as Website (Frontend)
    participant Catalog as CatalogService
    participant Cart as CartService
    participant Auth as AuthService
    participant Inv as InventoryService
    participant Pay as PaymentGateway
    participant Order as OrderService
    participant Ship as ShippingService
    participant Msg as Email/SMS Service
        %% Browse & select
    Customer->>Web: Open KK&CAKES
    Web->>Catalog: GET /products?category=cakes
    Catalog-->>Web: Product list (name, price, stock)
    Customer->>Web: View product details
    Web->>Catalog: GET /products/{id}
    Catalog-->>Web: {name, images, options, price, stock}
    Customer->>Web: Choose options (size, flavor, qty) + Add to cart
    Web->>Cart: POST /cart/items
    Cart-->>Web: Cart updated (items, subtotal)
       %% Cart review
    Customer->>Web: Open cart
    Web->>Cart: GET /cart
    Cart-->>Web: Items, totals
    alt Has coupon
        Customer->>Web: Enter coupon
        Web->>Cart: POST /cart/apply-coupon
        Cart-->>Web: Coupon valid → totals updated
    else No coupon
        Note over Web: Proceed without discount
    end

    %% Checkout & auth
    Customer->>Web: Proceed to checkout
    alt Not logged in
        Web->>Auth: POST /login or /signup
        Auth-->>Web: Session/JWT
    else Logged in
        Note over Web: Session already active
    end

    %% Fulfillment choice
    Web->>Customer: Choose delivery or pickup
    alt Delivery
        Customer->>Web: Enter/select address + window
    else Pickup
        Customer->>Web: Choose pickup location + time
    end

    %% Stock check & reservation
    Web->>Inv: POST /reserve {cartId, window}
    Inv-->>Web: Reservation token (TTL) / OK
    alt Any item unavailable
        Web-->>Customer: Show OOS/lead time → edit cart
        Customer->>Web: Update cart
        Web->>Cart: PATCH /cart
        Cart-->>Web: Updated items/totals
        Web->>Inv: Re-reserve
        Inv-->>Web: OK
    else All items reserved
        Note over Web,Inv: Inventory temporarily held
    end
