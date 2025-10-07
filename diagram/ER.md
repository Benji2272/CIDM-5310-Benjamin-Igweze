erDiagram
    %% ================= ENTITIES =================
    CUSTOMER {
      int id
      string name
      string email
      string phone
      string preferred_channel   /* email | sms | push */
      datetime created_at
    }

    EVENT {
      int id
      int customer_id
      string name                /* Birthday, Anniversary, etc. */
      date event_date
      int remind_days_before     /* e.g., 3–5 */
      datetime created_at
    }

    REMINDER {
      int id
      int event_id
      string channel             /* email | sms | push */
      datetime scheduled_at
      string status              /* pending | sent | failed */
      string message_template
    }

    "ORDER" {
      int id
      int customer_id
      int event_id               /* nullable */
      string status              /* created | paid | shipped | cancelled */
      decimal total_amount
      datetime created_at
    }

    ORDER_ITEM {
      int id
      int order_id
      int product_id
      int qty
      decimal unit_price
      decimal line_total
    }

    PRODUCT {
      int id
      string name
      string category
      decimal price
      bool is_active
    }

    %% ================ RELATIONSHIPS ================
    CUSTOMER ||--o{ EVENT : has
    EVENT ||--o{ REMINDER : schedules
    CUSTOMER ||--o{ "ORDER" : places
    EVENT |o--o{ "ORDER" : may_trigger
    "ORDER" ||--o{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : listed_in
