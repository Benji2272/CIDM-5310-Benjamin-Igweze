classDiagram
    direction LR

    class Customer {
      +int id
      +string name
      +string email
      +string phone
      +string preferredChannel  // email | sms | push
      +datetime createdAt
    }

    class Event {
      +int id
      +int customerId
      +string name             // Birthday, Anniversary, etc.
      +date eventDate
      +int remindDaysBefore    // e.g., 3~5
      +datetime createdAt
    }

    class Reminder {
      +int id
      +int eventId
      +string channel          // email | sms | push
      +datetime scheduledAt
      +string status           // pending | sent | failed
      +string messageTemplate  // optional
    }

    class Order {
      +int id
      +int customerId
      +int eventId             // nullable
      +string status           // created | paid | shipped | cancelled
      +decimal totalAmount
      +datetime createdAt
    }

    class OrderItem {
      +int id
      +int orderId
      +int productId
      +int qty
      +decimal unitPrice
      +decimal lineTotal
    }

    class Product {
      +int id
      +string name
      +string category
      +decimal price
      +bool isActive
    }

    %% Relationships
    Customer "1" --> "0..*" Event : has
    Event "1" --> "0..*" Reminder : schedules
    Customer "1" --> "0..*" Order : places
    Event "0..1" --> "0..*" Order : may_trigger
    Order "1" --> "1..*" OrderItem : contains
    Product "1" --> "0..*" OrderItem : referenced_by
