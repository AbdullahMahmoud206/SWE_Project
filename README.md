

## SequenceDiagram
```mermaid
   
    actor U as User
    participant App as Mobile App
    participant API as ASP.NET Core API
    participant DB as SQL Server

    U->>App: Open Alert Settings
    App->>API: GET /api/thresholds
    API->>DB: Retrieve saved thresholds
    DB-->>API: Return current thresholds
    API-->>App: Return thresholds as JSON
    App-->>U: Display threshold settings

    U->>App: Enter new temperature limit
    U->>App: Press Save
    App->>API: PUT /api/thresholds
    API->>API: Validate threshold values

    alt Invalid values
        API-->>App: 400 Bad Request
        App-->>U: Display validation error
    else Valid values
        API->>DB: Update threshold settings
        DB-->>API: Confirm update
        API-->>App: 200 OK
        App-->>U: Display success message
    end
```
