classDiagram
    direction LR

    class User {
      +UUID id
      +string name
      +string email
      +string phone
      +login()
      +requestPasswordReset()
    }

    class Address {
      +UUID id
      +string line1
      +string line2
      +string city
      +string state
      +string postalCode
      +string country
      +bool isDefault
    }

    class Product {
      +UUID id
      +string name
      +string slug
      +string description
      +money basePrice
      +bool isCustomizable
    }

    class ProductVariant {
      +UUID id
      +string sku
      +string size   // 6", 8", 10"
      +string flavor // vanilla, chocolate...
      +money price
      +int leadTimeDays
      +bool isAvailable
    }

    class InventoryItem {
      +UUID id
      +string sku
      +int quantityOnHand
      +int quantityReserved
      +reserve(qty)
      +commit(qty)
      +release(qty)
    }

    class Cart {
      +UUID id
      +money subtotal
      +money discount
      +money total
      +addItem()
      +removeItem()
      +applyCoupon(code)
    }

    class CartItem {
      +UUID id
      +int quantity
      +money unitPrice
      +money lineTotal
    }

    class Coupon {
      +UUID id
      +string code
      +float percentOff
      +money amountOff
      +date validFrom
      +date validTo
      +int usageLimit
      +isValid()
    }

    class Order {
      +UUID id
      +string orderNo
      +OrderStatus status
      +money subtotal
      +money discount
      +money shipping
      +money total
      +datetime placedAt
      +cancel()
      +markShipped()
      +markDelivered()
    }

    class OrderItem {
      +UUID id
      +int quantity
      +money unitPrice
      +money lineTotal
      +string sku
    }

    class Payment {
      +UUID id
      +string txnId
      +PaymentMethod method
      +PaymentStatus status
      +money amount
      +datetime paidAt
      +refund()
    }

    class Shipment {
      +UUID id
      +string carrier
      +string trackingNumber
      +datetime scheduledFor
      +ShipmentStatus status
    }

    %% (Optional) enums for clarity
    class OrderStatus {
      <<enumeration>>
      CREATED
      CONFIRMED
      FULFILLING
      SHIPPED
      DELIVERED
      CANCELLED
    }
    class PaymentStatus {
      <<enumeration>>
      INITIATED
      AUTHORIZED
      CAPTURED
      FAILED
      REFUNDED
    }
    class PaymentMethod {
      <<enumeration>>
      CARD
      WALLET
      CASH_ON_DELIVERY
    }
    class ShipmentStatus {
      <<enumeration>>
      SCHEDULED
      OUT_FOR_DELIVERY
      DELIVERED
    }

    %% Relationships (focus on order flow)
    User "1" -- "0..*" Address : has >
    Product "1" -- "0..*" ProductVariant : defines >
    ProductVariant "1" -- "1" InventoryItem : stockedBy >
    User "1" -- "1" Cart : owns >
    Cart "1" -- "0..*" CartItem : contains >
    ProductVariant "1" -- "0..*" CartItem : pricedFrom >
    Cart "0..1" -- "0..1" Coupon : uses >
    User "1" -- "0..*" Order : places >
    Order "1" -- "1..*" OrderItem : contains >
    ProductVariant "1" -- "0..*" OrderItem : fulfilledFrom >
    Order "1" -- "1" Payment : paidBy >
    Order "1" -- "0..1" Shipment : fulfilledBy >
    Order "1" -- "1" Address : shipsTo >
    Order "1" -- "1" Address : billsTo >
