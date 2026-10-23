

# Sequence Diagrams:
> First 5 for student: Omar Tamer
### Diagram 1: Request Connection Flow
```mermaid
sequenceDiagram
    participant User as User:
    participant App as App:
    participant API as API:
    participant ESP32 as ESP32:

    activate User
    User->>App: Request connection
    activate App
    App->>API: Request connection
    activate API
    API->>ESP32: Request connection
    activate ESP32

    alt Connection Success
        ESP32-->>API: Connection success
        API-->>App: Connection success
        App-->>User: Connection success
    else Connection Failure
        ESP32-->>API: Connection failure
        deactivate ESP32
        API-->>App: Connection failure
        deactivate API
        App-->>User: Connection failure
        deactivate App
        deactivate User
    end
```
### Diagram 2: Read TMT6000 Sensor Data

```mermaid
sequenceDiagram
    participant ESP32 as ESP32:
    participant GPIO as GPIO:
    participant TMT6000 as TMT6000 Sensor:

    activate ESP32
    ESP32->>GPIO: Read Sensor
    activate GPIO
    GPIO->>TMT6000: 
    activate TMT6000
    TMT6000-->>GPIO: Readings
    deactivate TMT6000
    GPIO-->>ESP32: Readings
    deactivate GPIO
    ESP32->>ESP32: Prepare Readings To Be Sent
    deactivate ESP32
```

### Diagram 3: Read DHT11 Sensor Data

```mermaid
sequenceDiagram
    participant ESP32 as ESP32:
    participant GPIO as GPIO:
    participant DHT11 as DHT11 Sensor:

    activate ESP32
    ESP32->>GPIO: Read Sensor
    activate GPIO
    GPIO->>DHT11: 
    activate DHT11
    DHT11-->>GPIO: Readings
    deactivate DHT11
    GPIO-->>ESP32: Readings
    deactivate GPIO
    ESP32->>ESP32: Prepare Readings To Be Sent
    deactivate ESP32
```
### Diagram 4: Device Scan Flow

```mermaid
sequenceDiagram
    participant User as User:
    participant App as App:
    participant API as API:

    activate User
    User->>App: Request device scan
    activate App
    App->>API: Request device scan
    activate API
    API->>API: Scan for devices

    alt Devices Found
        API-->>App: Device list
        App-->>User: Device list
    else No Devices Found
        API-->>App: No devices found
        deactivate API
        App-->>User: No devices found
        deactivate App
        deactivate User
    end
```

### Diagram 5: Sensor Connection Testing
```mermaid
sequenceDiagram
    participant ESP32 as ESP32:
    participant GPIO as GPIO:
    participant DH11 as DH11 Sensor:
    participant TEMT6000 as TEMT6000 Sensor:

    activate ESP32
    ESP32->>GPIO: Initialize GPIO
    activate GPIO
    deactivate GPIO

    ESP32->>GPIO: Test Sensor Connection
    activate GPIO
    GPIO->>DH11: 
    activate DH11

    alt DH11 Connected
        DH11-->>GPIO: Connected
        GPIO-->>ESP32: Connected
    else DH11 Not Connected
        DH11-->>GPIO: Not Connected
        deactivate DH11
        GPIO-->>ESP32: Not Connected
        deactivate GPIO
    end

    ESP32->>GPIO: Test Sensor Connection
    activate GPIO
    GPIO->>TEMT6000: Test Sensor Connection
    activate TEMT6000

    alt TEMT6000 Connected
        TEMT6000-->>GPIO: Connected
        GPIO-->>ESP32: Connected
    else TEMT6000 Not Connected
        TEMT6000-->>GPIO: Not Connected
        deactivate TEMT6000
        GPIO-->>ESP32: Not Connected
        deactivate GPIO
    end
    deactivate ESP32
```



## Diagram 5: Configure Alert Thresholds
### Participants:
- **User:** The user who defines the alert boundaries.

- **Mobile App:** The mobile application that displays and sends the settings.

- **ASP.NET Core API:** Receives the request and verifies the values.

- **SQL Server:** Saves the alert boundaries in the database.
```mermaid


   
sequenceDiagram
    autonumber
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


## Sequence 7: View Alerts
### Participants:
- **User:** The user who wants to view the alerts.

- **Mobile App:** The mobile application that displays the list of alerts.

- **ASP.NET Core API:** Receives the application request and retrieves the alerts.

- **SQL Server:** The database that stores the recorded alerts.
```mermaid
   
sequenceDiagram
    autonumber
    actor U as User
    participant App as Mobile App
    participant API as ASP.NET Core API
    participant DB as SQL Server

    U->>App: Open Alerts Page
    App->>API: GET /api/alerts
    API->>DB: Retrieve saved alerts
    DB-->>API: Return alerts list
    API-->>App: Return alerts as JSON
    App->>App: Process alerts data

    alt Alerts exist
        App-->>U: Display alerts list
    else No alerts found
        App-->>U: Display No Alerts message
    end

    U->>App: Select an alert
    App-->>U: Display alert details

```




