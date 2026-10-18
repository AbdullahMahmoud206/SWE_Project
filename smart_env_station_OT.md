## Sequence 1: Device scanning

```mermaid
sequenceDiagram
    actor Homeowner
    participant App
    participant API

    Homeowner->>App: Request device scan
    App->>API: Request device scan
    API->>API: Scan for devices

    alt Devices found
        API-->>App: Device list
        App-->>Homeowner: Device list
    else No devices found
        API-->>App: No devices found
        App-->>Homeowner: No devices found
    end
```

## Sequence 2: ESP32 Connection

```mermaid

sequenceDiagram
    actor Homeowner
    participant App
    participant API
    participant ESP32

    Homeowner->>App: Request connection
    App->>API: Request connection
    API->>ESP32: Request connection

    alt Connection successful
        ESP32-->>API: Connection success
        API-->>App: Connection success
        App-->>Homeowner: Connection success
    else Connection failed
        ESP32-->>API: Connection failure
        API-->>App: Connection failure
        App-->>Homeowner: Connection failure
    end

```
