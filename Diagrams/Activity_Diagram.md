# Smart Environment Station


## Activity Diagram 1: ESP32 Initialization & Sensor Check
> Eng. Omar Ismail

Based on Sequence Diagram 5 (Sensor Connection Testing), Sequence Diagram 3 (Read DHT11 Sensor Data) and Sequence Diagram 2 (Read TEMT6000 Sensor Data), at a higher level of abstraction (GPIO details omitted). The DHT11 and TEMT6000 tests run in parallel (fork/join). If both sensors are connected, the ESP32 reads both sensors in parallel (fork/join) and prepares the readings to be sent. If at least one sensor is not connected, the ESP32 collects the status of both sensors in parallel (fork/join) and reports the sensor fault. The two outcome branches mirror each other.

```mermaid
stateDiagram-v2
    direction TB

    state "Power on ESP32" as PowerOn
    state "Initialize ESP32" as Init

    state fork_test <<fork>>

    state "Test DHT11 connection" as TestDHT
    state dht_result <<choice>>
    state "DHT11 connected" as DHTOk
    state "DHT11 not connected" as DHTFail

    state "Test TEMT6000 connection" as TestTEMT
    state temt_result <<choice>>
    state "TEMT6000 connected" as TEMTOk
    state "TEMT6000 not connected" as TEMTFail

    state join_test <<join>>

    state "Evaluate sensor test results" as Evaluate
    state all_ok <<choice>>

    state fork_read <<fork>>
    state "Read DHT11 sensor data" as ReadDHT
    state "Read TEMT6000 sensor data" as ReadTEMT
    state join_read <<join>>
    state "Prepare readings to be sent" as Prepare

    state fork_fault <<fork>>
    state "Collect DHT11 status" as StatusDHT
    state "Collect TEMT6000 status" as StatusTEMT
    state join_fault <<join>>
    state "Report sensor fault" as Fault

    [*] --> PowerOn
    PowerOn --> Init
    Init --> fork_test

    fork_test --> TestDHT
    fork_test --> TestTEMT

    TestDHT --> dht_result
    dht_result --> DHTOk: Connected
    dht_result --> DHTFail: Not connected

    TestTEMT --> temt_result
    temt_result --> TEMTOk: Connected
    temt_result --> TEMTFail: Not connected

    DHTOk --> join_test
    DHTFail --> join_test
    TEMTOk --> join_test
    TEMTFail --> join_test

    join_test --> Evaluate
    Evaluate --> all_ok
    all_ok --> fork_read: Both sensors connected
    all_ok --> fork_fault: At least one sensor not connected

    fork_read --> ReadDHT
    fork_read --> ReadTEMT
    ReadDHT --> join_read
    ReadTEMT --> join_read
    join_read --> Prepare
    Prepare --> [*]

    fork_fault --> StatusDHT
    fork_fault --> StatusTEMT
    StatusDHT --> join_fault
    StatusTEMT --> join_fault
    join_fault --> Fault
    Fault --> [*]
```

---
## Diagram 2: Connection Request & Device Scan

Based on Sequence Diagram 1 (Request Connection Flow) and Sequence Diagram 4 (Device Scan Flow). After the user opens the app, a decision selects which action the user takes: request a connection (left branch) or request a device scan (right branch). The two paths merge again before the end.

```mermaid
stateDiagram-v2
    direction TB

    state "User opens app" as OpenApp
    state action_choice <<choice>>

    %% Branch A: Connection request
    state "User requests connection" as C1
    state "App forwards request to API" as C2
    state "API forwards request to ESP32" as C3
    state conn_result <<choice>>
    state "Connection success" as C4
    state "Connection failure" as C5
    state "App displays connection result to user" as C6

    %% Branch B: Device scan
    state "User requests device scan" as S1
    state "App forwards request to API" as S2
    state "API scans for devices" as S3
    state scan_result <<choice>>
    state "Device list returned" as S4
    state "No devices found" as S5
    state "App displays scan result to user" as S6

    state merge_end <<choice>>
    state "Result shown to user" as Done

    [*] --> OpenApp
    OpenApp --> action_choice

    action_choice --> C1: User selects connect
    C1 --> C2
    C2 --> C3
    C3 --> conn_result
    conn_result --> C4: ESP32 responds with success
    conn_result --> C5: ESP32 responds with failure
    C4 --> C6
    C5 --> C6
    C6 --> merge_end

    action_choice --> S1: User selects device scan
    S1 --> S2
    S2 --> S3
    S3 --> scan_result
    scan_result --> S4: Devices found
    scan_result --> S5: No devices found
    S4 --> S6
    S5 --> S6
    S6 --> merge_end

    merge_end --> Done
    Done --> [*]
```
---
## Diagram 5 - Threshold Alert Decision and Alert Suppression
> Eng. Abdullah Mahmoud
```mermaid
stateDiagram-v2
    direction TB

    state "Receive Sensor Reading" as Receive
    state "Check Threshold" as CheckThresh
    state threshold_choice <<choice>>
    state "Check Existing Alert" as CheckExisting
    state "No Alert Required" as NoAlert
    state existing_choice <<choice>>
    state "Skip Duplicate Alert" as SkipAlert
    state "Generate Alert" as GenAlert
    state "Store Alert in Database" as StoreAlert
    state merge_inner <<choice>>
    state merge_outer <<choice>>

    [*] --> Receive
    Receive --> CheckThresh
    CheckThresh --> threshold_choice : Reading exceeds threshold?
    
    threshold_choice --> CheckExisting : Yes
    threshold_choice --> NoAlert : No
    
    CheckExisting --> existing_choice : Alert already exists?
    existing_choice --> SkipAlert : Yes
    existing_choice --> GenAlert : No
    
    GenAlert --> StoreAlert
    
    SkipAlert --> merge_inner
    StoreAlert --> merge_inner
    
    merge_inner --> merge_outer
    NoAlert --> merge_outer
    
    merge_outer --> [*]
```

## Diagram 8 - Environmental Data Synchronization
```mermaid
stateDiagram-v2
    direction TB

    state "Receive Sensor Readings" as Receive
    state "Update Current Readings" as Update
    state "Prepare Data for Storage" as Prepare
    state "Save Readings to Database" as Save
    state save_choice <<choice>>
    state "Confirm Data Saved" as ConfirmLeft
    state "Mark Data as Pending" as MarkPending
    state retry_choice <<choice>>
    state "Retry Saving" as Retry
    state "Check Saving Result" as CheckResult
    state "Confirm Data Saved" as ConfirmRight
    state merge_final <<choice>>

    [*] --> Receive
    Receive --> Update
    Update --> Prepare
    Prepare --> Save
    Save --> save_choice : Was saving successful?

    save_choice --> ConfirmLeft : Yes
    save_choice --> MarkPending : No

    MarkPending --> retry_choice : Saving unsuccessful?
    retry_choice --> Retry : Yes
    retry_choice --> ConfirmRight : No

    Retry --> CheckResult
    CheckResult --> retry_choice

    ConfirmLeft --> merge_final
    ConfirmRight --> merge_final

    merge_final --> [*]

```
