# Smart Environment Station - Activity Diagrams
## This document contains the activity diagrams for the Smart Environment Station project. Each diagram represents a specific workflow or process within the system, providing a visual representation of the activities and their interactions.
- [Diagram 1](#diagram-1---dashboard-readings-and-alert-flow), [Diagram 4](#diagram-4---handling-multiple-threshold-violations) Eng. Omar Emad
- [Diagram 2](#diagram-2-esp32-initialization--sensor-check), [Diagram 3](#diagram-3-connection-request--device-scan) Eng. Omar Ismail
- [Diagram 5](#diagram-5---threshold-alert-decision-and-alert-suppression), [Diagram 6](#diagram-6---environmental-data-synchronization) Eng. Abdullah Mahmoud
- [Diagram 7](#diagram-7---environmental-monitoring-session-life-cycle), [Diagram 8](#Diagram-8---Threshold-checking) Eng. Omar Tamer

## Diagram 2: ESP32 Initialization & Sensor Check
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
## Diagram 3: Connection Request & Device Scan

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

## Diagram 6 - Environmental Data Synchronization
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
---
## Diagram 1 - Dashboard Readings and Alert Flow
> Eng. Omar Emad

```mermaid
stateDiagram-v2
    direction TB

    state "User Opens Dashboard" as OpenDash
    state "Users select data type" as SelectType
    state data_type_choice <<choice>>

    %% Left Branch (Current Readings)
    state "Request Current Data from API" as ReqCurrent
    state "Receive Current Readings" as RecCurrent
    state "Display Current Readings" as DispCurrent
    state "Compare Readings with Thresholds" as CompThresh
    state thresh_choice <<choice>>
    state "don't perform any thing" as DoNothing
    state fork_alert <<fork>>
    state "store this readings to database" as StoreReadings
    state "Generate alert to user" as GenAlert
    state join_alert <<join>>

    %% Right Branch (Historical Readings)
    state "User select the selected period" as SelectPeriod
    state "Check the selected period contain readings or not" as CheckPeriod
    state data_avail_choice <<choice>>
    state "Print no readings available in this period" as PrintNoData
    state "display historical readings" as DispHist
    state merge_hist <<choice>>

    [*] --> OpenDash
    OpenDash --> SelectType
    SelectType --> data_type_choice

    %% Current Readings Path
    data_type_choice --> ReqCurrent : [User select to show current readings]
    ReqCurrent --> RecCurrent
    RecCurrent --> DispCurrent
    DispCurrent --> CompThresh
    CompThresh --> thresh_choice

    thresh_choice --> DoNothing : [If readings don't exceed thresholds]
    thresh_choice --> fork_alert : [If readings exceed thresholds]

    fork_alert --> StoreReadings
    fork_alert --> GenAlert
    StoreReadings --> join_alert
    GenAlert --> join_alert

    DoNothing --> [*]
    join_alert --> [*]

    %% Historical Readings Path
    data_type_choice --> SelectPeriod : [User select to show historical readings]
    SelectPeriod --> CheckPeriod
    CheckPeriod --> data_avail_choice

    data_avail_choice --> PrintNoData : no available data
    data_avail_choice --> DispHist : Data is available

    PrintNoData --> merge_hist
    DispHist --> merge_hist
    merge_hist --> [*]
```
## Diagram 4 - Handling multiple threshold violations 

```mermaid
stateDiagram-v2
    direction TB

    state "Read current readings" as ReadReadings
    state "Load alert thresholds" as LoadThresh
    state "Compare readings with thresholds" as CompThresh
    state violation_choice <<choice>>

    state "Create violations list" as CreateList
    state loop_merge <<choice>>
    state more_violations_choice <<choice>>
    state "Get next violations" as GetNext
    state "Classify severity(low,medium,high)" as ClassifySev

    state "sort violations by severity" as SortViolations
    state "Combine violation in one alert" as CombineAlert
    state fork_process <<fork>>
    state "store violations in database" as StoreViolations
    state "Send combined alert to user" as SendAlert
    state join_process <<join>>

    [*] --> ReadReadings
    ReadReadings --> LoadThresh
    LoadThresh --> CompThresh
    CompThresh --> violation_choice

    violation_choice --> [*] : No violation found
    violation_choice --> CreateList : [Violations found]

    CreateList --> loop_merge
    loop_merge --> more_violations_choice

    more_violations_choice --> GetNext : Yes (More unprocessed violations)
    GetNext --> ClassifySev
    ClassifySev --> loop_merge

    more_violations_choice --> SortViolations : NO
    SortViolations --> CombineAlert
    CombineAlert --> fork_process

    fork_process --> StoreViolations
    fork_process --> SendAlert
    StoreViolations --> join_process
    SendAlert --> join_process

    join_process --> [*]
```
---
## Diagram 7 - Environmental Monitoring session life cycle
> Eng. Omar Tamer

```mermaid
stateDiagram-v2
    direction TB

    state "Initialization" as Init
    state init_choice <<choice>>
    state "Terminate" as Term
    state "Start" as StartNode

    state fork_main <<fork>>
    
    %% Left Parallel Branch
    state "Collect data" as Collect
    state "Process data" as Process

    %% Right Parallel Branch
    state "Await connection" as AwaitConn
    state conn_choice <<choice>>

    state join_main <<join>>
    state "Send data" as SendData

    [*] --> Init
    Init --> init_choice : Initialization success

    init_choice --> Term : [No]
    Term --> [*]

    init_choice --> StartNode : [Yes]
    StartNode --> fork_main

    %% Left Branch Flow
    fork_main --> Collect
    Collect --> Process
    Process --> join_main

    %% Right Branch Flow
    fork_main --> AwaitConn
    AwaitConn --> conn_choice : Connection established
    conn_choice --> AwaitConn : [No]
    conn_choice --> join_main : [Yes]

    %% Join and Loop Back
    join_main --> SendData
    SendData --> StartNode
```
## Diagram 8 - Threshold checking
```mermaid
stateDiagram-v2
    direction TB

    state "Read sensors" as ReadSensors
    state "Process/Validate data" as ProcessData
    state threshold_choice <<choice>>
    state "Send alert" as SendAlert
    state "Send data" as SendData

    [*] --> ReadSensors
    ReadSensors --> ProcessData
    ProcessData --> threshold_choice

    threshold_choice --> SendAlert : [Threshold exceeded]
    threshold_choice --> SendData : [Within limits]

    SendAlert --> SendData
    SendData --> ReadSensors
```


