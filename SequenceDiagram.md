
# Smart Environment station  
## Sequence Diagrams:
> First 5 for student: Omar Tamer
### 1: Request Connection Flow
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
### 2: Read TMT6000 Sensor Data

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

### 3: Read DHT11 Sensor Data

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
### 4: Device Scan Flow

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

### 5: Sensor Connection Testing
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

> 6, 7 Student: Omar Ismail
### 6: Alert Notification Loop
```mermaid
sequenceDiagram
    actor User
    participant Application
    participant API
    participant ESP32
    participant Database

    activate ESP32
    loop Loop
        ESP32->>ESP32: Checking Thresholds
        alt If Threshold exceeded
            ESP32->>ESP32: Thresholds exceeded
            ESP32->>API: Send Alert signals
            activate API
            API->>Database: Store Alert date & values
            activate Database
            API->>Application: Display Alert notification
            activate Application
            Application->>User: Display Alert notification
            activate User
            deactivate User
            deactivate Application
            deactivate Database
            deactivate API
        else else
            ESP32->>ESP32: Checking Thresholds
        end
    end
    deactivate ESP32
```
### 7: Send sensor data
```mermaid
sequenceDiagram
    participant ESP32
    participant API
    participant Database

    activate ESP32
    activate API
    activate Database
    loop Loop
        ESP32->>ESP32: Reading sensors
        ESP32->>ESP32: Validating Data
        ESP32->>API: Send Sensor Data
        API->>Database: Store data
        Database-->>API: Storing Confirmation
    end
    deactivate Database
    deactivate API
    deactivate ESP32
```

> 8, 9 student: [Abdullah Mahmoud](https://github.com/AbdullahMahmoud206)
### 8: Configure Alert Thresholds
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


### 9: View Alerts
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




