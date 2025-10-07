flowchart TD
    A[Start] --> B[Customer Registers or Logs In]
    B --> C[Customer Adds Event Dates<br>(Birthday, Anniversary, etc.)]
    C --> D[Save Event Details to Database]
    D --> E[System Scheduler Runs Daily<br>(Cron Job)]
    E --> F{Any Upcoming Events<br>in 3-5 Days?}
    F -->|Yes| G[Fetch Customer Details<br>& Event Info]
    F -->|No| O[No Action Needed]
    G --> H[Generate Personalized<br>Reminder Message]
    H --> I[Send Notification<br>(Email, SMS, or App)]
    I --> J[Include Order Link<br>or Discount Code]
    J --> K[Customer Clicks & Places Order]
    K --> L[Order Confirmation Sent]
    L --> M[Record Customer Response Analytics]
    M --> N[End]
    O --> N

    %% Simple color styling for readability
    style A fill:#fde8e8,stroke:#e57373,stroke-width:2px
    style N fill:#fde8e8,stroke:#e57373,stroke-width:2px
    style B,C fill:#fff9c4,stroke:#fbc02d,stroke-width:1.5px
    style D,E fill:#e1f5fe,stroke:#42a5f5,stroke-width:1.5px
    style F,G,H,I,J fill:#f3e5f5,stroke:#ab47bc,stroke-width:1.5px
    style K,L,M fill:#e8f5e9,stroke:#66bb6a,stroke-width:1.5px
    style O fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1.5px
