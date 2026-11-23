# Smart Environment - Readings lifecycle - State Diagram
This model represents the lifecycle of a readings in a smart environment system. It illustrates the various states a readings can be in, as well as the transitions between these states.
---
## Visual Representation
```mermaid
stateDiagram-v2

    [*] --> NotMeasured : SensorsOff() 

    NotMeasured --> Reading : SensorsOn()

    Reading --> Valid : ValidateReadings()
    Reading --> Invalid : ValidateReadings()
    Invalid --> Reading : Read()

    Valid --> NotThresholdExceeded : CheckThreshold()
    Valid --> ThresholdExceeded : CheckThreshold()

    NotThresholdExceeded --> Stored_In_DB : StoreReadings()
    Stored_In_DB --> Showed : DisplayReadings()

    ThresholdExceeded --> Notified : AlertNotification()
    ThresholdExceeded --> Stored_In_DB : StoreReadings()
    Notified --> Showed : DisplayAlerts()
    Showed --> [*]
```

## State Difinitions
| State | Description | Entry / Do / Exit Behaviors |
| :--- | :--- | :--- |
| **NotMeasured** | The Readings have not been taken yet. | `Entry: SensorsOff()` |
| **Reading** | The Readings are currently being taken. | `Entry: SensorsOn()` |
| **Valid** | The Readings argumnets are valid. | `Entry: ValidateReadings()` |
| **Invalid** | The Reading arguments are invalid. | `Entry: ValidateReadings()` |
| **Stored_In_DB** | The measurements are stored in the database. | `Entry: StoreReadings()` |
| **Showed** | The Readings are displayed through Mobile App . | `Entry: DisplayReadings()` |
| **ThresholdExceeded** | The measurements have exceeded the threshold. | `Entry: CheckThreshold()` |
| **NotThresholdExceeded** | The measurements have not exceeded the threshold. | `Entry: CheckThreshold()` |
| **Notified** | An alert has been notified. | `Entry: AlertNotification()` |
