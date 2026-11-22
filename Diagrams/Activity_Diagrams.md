# Smart Environment Station --- Activity Diagrams

This README documents the **last eight activity-diagram ideas**
discussed for the Smart Environment Station system. Each diagram is
represented as a Mermaid flowchart so it can be viewed in GitHub and
other Markdown viewers that support Mermaid.

> **Scope note:** These are workflow proposals based on the functions
> discussed: ESP device discovery/connection, sensor readings, current
> and historical readings, threshold checking, alerts, and database
> storage. A few branches depend on implementation details that have not
> yet been confirmed. Those are called out under the relevant diagram
> and should only be included in the final project if they match the
> actual requirements and sequence diagrams.

------------------------------------------------------------------------

## 1. Multiple Threshold Violations

**Purpose:** Handle a monitoring cycle in which readings from multiple
sensors are evaluated, rather than checking only one reading.

``` mermaid
flowchart TD
    A([Start]) --> B[Receive multiple sensor readings]
    B --> C[Compare each reading with its threshold]
    C --> D{Any threshold violations?}
    D -- No --> E[Continue normal monitoring]
    D -- Yes --> F[Identify parameters that exceeded limits]
    F --> G{{Fork: perform both actions}}
    G --> H[Generate alert for violated parameters]
    G --> I[Store abnormal readings in database]
    H --> J{{Join}}
    I --> J
    J --> K[Display monitoring summary]
    E --> L([End])
    K --> L
```

**Key activity-diagram concept:** Parallel actions after identifying
threshold violations.

**Condition:** Use this workflow if the system evaluates readings from
more than one sensor in the same monitoring cycle.

------------------------------------------------------------------------

## 2. Device Selection Based on Scan Results

**Purpose:** Show different outcomes depending on how many ESP devices
are discovered during a scan.

``` mermaid
flowchart TD
    A([Start]) --> B[Scan for available devices]
    B --> C[Retrieve discovered devices]
    C --> D{How many devices were found?}
    D -- None --> E[Display no devices found]
    E --> F{Scan again?}
    F -- Yes --> B
    F -- No --> Z([End])
    D -- One --> G[Select the discovered device]
    D -- Multiple --> H[Display available devices]
    H --> I[User selects a device]
    G --> J[Send connection request]
    I --> J
    J --> K[Test connection]
    K --> L{Connection successful?}
    L -- Yes --> M[Set device as connected]
    L -- No --> N[Display connection failure]
    M --> Z
    N --> Z
```

**Key activity-diagram concept:** Multiple decision branches based on
the scan result.

**Condition:** The automatic selection branch assumes that the app can
connect directly when exactly one device is found. Keep it only if that
behavior is supported by the system design.

------------------------------------------------------------------------

## 3. Historical Readings Filtering

**Purpose:** Retrieve historical readings for a selected time period and
handle the case where no matching data exists.

``` mermaid
flowchart TD
    A([Start]) --> B[Request historical readings]
    B --> C[Select time period]
    C --> D[Send database query]
    D --> E[Retrieve matching readings]
    E --> F{Any data available?}
    F -- No --> G[Display empty history]
    F -- Yes --> H[Filter and organize readings]
    H --> I[Generate historical chart]
    I --> J[Display historical results]
    G --> K([End])
    J --> K
```

**Key activity-diagram concept:** Alternative outcomes for an empty
result versus a result containing readings.

------------------------------------------------------------------------

## 4. Sensor Failure Handling During Reading

**Purpose:** Represent what happens if a sensor does not provide a
usable response during a reading cycle.

``` mermaid
flowchart TD
    A([Start]) --> B[Request sensor readings from ESP]
    B --> C[Receive sensor response]
    C --> D{Is the response valid?}
    D -- Yes --> E[Process received readings]
    E --> F[Display available readings]
    D -- No --> G[Mark sensor as unavailable]
    G --> H[Check remaining sensor readings]
    H --> I[Display available readings and sensor status]
    F --> J([End])
    I --> J
```

**Key activity-diagram concept:** A failure branch that allows the
workflow to continue with available sensor data.

**Condition:** Use this only if the system can identify an unavailable
or invalid sensor response and continue with other sensors.

------------------------------------------------------------------------

## 5. Threshold Alert Decision and Alert Suppression

**Purpose:** Avoid generating a duplicate alert when an alert for the
same threshold violation already exists.

``` mermaid
flowchart TD
    A([Start]) --> B[Receive sensor reading]
    B --> C[Check reading against threshold]
    C --> D{Threshold exceeded?}
    D -- No --> E[Update normal status]
    D -- Yes --> F[Check for an existing alert]
    F --> G{Alert already exists?}
    G -- Yes --> H[Update existing alert]
    G -- No --> I[Generate new alert]
    H --> J[Store reading]
    I --> J
    E --> K([End])
    J --> K
```

**Key activity-diagram concept:** A decision based on whether an alert
already exists.

**Important:** Alert lookup, updating an existing alert, and suppressing
duplicates were not confirmed as existing system functions. Treat this
as an optional design idea unless those behaviors are present in the
requirements or implementation.

------------------------------------------------------------------------

## 6. Interrupted Reading Request

**Purpose:** Show how the app can handle a reading request when the ESP
does not respond.

``` mermaid
flowchart TD
    A([Start]) --> B[Send reading request to ESP]
    B --> C[Wait for ESP response]
    C --> D{Response received?}
    D -- Yes --> E[Validate received data]
    E --> F[Display readings]
    D -- No --> G{Retry limit reached?}
    G -- No --> H[Retry reading request]
    H --> B
    G -- Yes --> I[Display communication error]
    F --> J([End])
    I --> J
```

**Key activity-diagram concept:** A retry loop with a termination
condition.

**Condition:** Include the retry limit only if the application has a
defined retry policy. Otherwise, represent the failure handling
according to the actual sequence diagram.

------------------------------------------------------------------------

## 7. Environmental Monitoring Session Lifecycle

**Purpose:** Represent a complete monitoring session, from
initialization through repeated reading cycles to session termination.

``` mermaid
flowchart TD
    A([Start]) --> B[Start monitoring session]
    B --> C[Verify ESP connection]
    C --> D{ESP connected?}
    D -- No --> E[Display connection error]
    E --> Z([End])
    D -- Yes --> F[Initialize sensors]
    F --> G[Start periodic reading cycle]
    G --> H[Collect sensor readings]
    H --> I[Update current readings]
    I --> J[Store readings in database]
    J --> K{Continue monitoring?}
    K -- Yes --> G
    K -- No --> L[Stop reading process]
    L --> M[Close monitoring session]
    M --> Z
```

**Key activity-diagram concept:** A repeated monitoring loop controlled
by a continue/stop decision.

**Important:** A user-controlled start/stop session and explicit
session-closing behavior were not confirmed in the functions listed so
far. Keep this diagram only if the system actually supports those
actions.

------------------------------------------------------------------------

## 8. Environmental Data Synchronization

**Purpose:** Represent the flow of a reading from the application to the
database, including confirmation that storage succeeded.

``` mermaid
flowchart TD
    A([Start]) --> B[Receive new sensor readings]
    B --> C[Update current readings in application]
    C --> D[Prepare readings for storage]
    D --> E[Save readings to database]
    E --> F{Storage successful?}
    F -- Yes --> G[Mark data as synchronized]
    F -- No --> H[Keep data pending]
    H --> I[Retry database storage]
    I --> E
    G --> J([End])
```

**Key activity-diagram concept:** A success/failure decision and a retry
loop around database storage.

**Important:** Storage confirmation, pending-data status, and retrying
database writes were not confirmed as existing system behaviors. Include
them only if they are part of the implementation or approved
requirements.

------------------------------------------------------------------------

## Summary

  -----------------------------------------------------------------------
  \#                      Activity diagram        Main modeling idea
  ----------------------- ----------------------- -----------------------
  1                       Multiple Threshold      Parallel alert
                          Violations              generation and database
                                                  storage

  2                       Device Selection Based  Branching based on the
                          on Scan Results         number of discovered
                                                  devices

  3                       Historical Readings     Empty-result versus
                          Filtering               data-available paths

  4                       Sensor Failure Handling Continue with available
                          During Reading          sensor data after a
                                                  failure

  5                       Threshold Alert         Check for an existing
                          Decision and Alert      alert before generating
                          Suppression             another

  6                       Interrupted Reading     Retry loop and
                          Request                 communication-failure
                                                  path

  7                       Environmental           Repeated monitoring
                          Monitoring Session      cycle and session
                          Lifecycle               termination

  8                       Environmental Data      Storage confirmation
                          Synchronization         and retry handling
  -----------------------------------------------------------------------

## Before using these diagrams in the project

-   Keep each activity consistent with the system's actual sequence
    diagrams and requirements.
-   Do not present optional assumptions as implemented functionality.
-   Use UML fork and join nodes only for genuinely parallel actions. In
    Mermaid, the `Fork` and `Join` labels above are explanatory
    representations.
-   Rename actions to match the exact terminology used in the project.
