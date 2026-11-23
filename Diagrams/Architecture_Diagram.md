## Smart Environment Station - Architecture Diagrams
# This document contains the architecture diagrams for the Smart Environment Station project. Each diagram represents a specific component or aspect of the system, providing a visual representation of the architecture and its interactions.

```mermaid
flowchart TB
    %% Nodes with Stereotypes and Descriptions
    ESP32["«device»<br/><b>ESP32 Device</b><br/><br/>(Sensor Station)"]
    DataReceiver["«component»<br/><b>Data Receiver</b><br/><br/>validates sensor data"]
    AlertEngine["«component»<br/><b>Alert Engine</b><br/><br/>checks thresholds,<br/>generates alerts"]
    NotifService["«service»<br/><b>Notification Service</b><br/><br/>(Optional)"]
    
    QueryService["«component»<br/><b>Query Service</b><br/><br/>current readings,<br/>historical data, alerts"]
    Database[("«database»<br/><b>Database</b><br/><br/>readings, devices,<br/>thresholds, alerts")]
    ConfigManager["«component»<br/><b>Configuration Manager</b><br/><br/>devices (add / edit / remove)<br/>alert thresholds"]
    
    UI["«UI»<br/><b>User Interface</b><br/><br/>(Dashboard / App)"]
    User(["<b>User</b>"])

    %% Top Layer Connections
    ESP32 -- "sensor readings" --> DataReceiver
    DataReceiver -- "ack / error" --> ESP32
    DataReceiver -- "valid readings" --> AlertEngine
    AlertEngine -- "notification request" --> NotifService

    %% Data Receiver <-> Database
    DataReceiver -- "store valid readings" --> Database
    Database -- "registered devices" --> DataReceiver

    %% Alert Engine <-> Database
    Database -- "load thresholds" --> AlertEngine
    AlertEngine -- "save alerts" --> Database

    %% Middle Layer Connections (Database <-> Services)
    QueryService -- "query by period" --> Database
    Database -- "readings / no data" --> QueryService

    ConfigManager -- "save devices, thresholds" --> Database
    Database -- "current settings" --> ConfigManager

    %% Bottom Layer Connections (UI <-> Services)
    UI -- "request readings,<br/>history & alerts" --> QueryService
    QueryService -- "readings, history,<br/>alerts" --> UI

    UI -- "manage devices,<br/>configure thresholds" --> ConfigManager
    ConfigManager -- "result /<br/>current settings" --> UI

    %% User Interactions
    User -- "uses" --> UI
    NotifService -- "push notification" --> User
```
---
