# Smart Environment Station — UML Activity Diagrams

This file models the **eight activity-diagram workflows** for the Smart Environment Station using **Mermaid (`flowchart TD`) with strict UML 2.x Activity Diagram design constraints**.

Because Mermaid does not have a dedicated `activityDiagram` keyword, standard UML 2.x semantics are enforced through the following structural conventions:
* **Initial (`●`) and Activity-Final (`◉`) Nodes:** Rendered as a solid filled circle `Start((" "))` and a double-circle bullseye `Stop(((" ")))`.
* **Action Nodes (`([Action])`):** Stadium-shaped nodes labeled with active verb phrases.
* **Separate Decision and Merge Diamonds (`{ }`):** Decision nodes accept **one incoming** flow and emit **two or more guarded** outgoing flows (`"[Guard]"`). Empty Merge diamonds (`{" "}`) reunite alternative branches or loop re-entries so flows never merge directly into an action or decision node.
* **Fork and Join Synchronization Bars (`▬`):** Parallel branches split from a solid black Fork bar and synchronize at a matching Join bar before continuing.

> **Scope note:** These diagrams are based on the Smart Environment Station functions discussed: ESP discovery and connection, sensor readings, current and historical readings, threshold checking, alerts, and database storage. Some branches are optional assumptions, identified below. Keep them only if they match the actual requirements and sequence diagrams.

---

## 1. Multiple Threshold Violations

**Purpose:** Evaluate readings from multiple sensors and handle more than one threshold violation in a monitoring cycle.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Receive multiple sensor readings"])
    A1 --> A2(["Compare each reading with its threshold"])
    A2 --> D1{"Any threshold violations?"}

    D1 -->|"[Yes]"| A3(["Identify violated parameters"])
    A3 --> Fork1["──────────────"]
    Fork1 --> A4(["Generate alert for violated parameters"])
    Fork1 --> A5(["Store abnormal readings in database"])
    A4 --> Join1["──────────────"]
    A5 --> Join1
    Join1 --> A6(["Display monitoring summary"])
    A6 --> M1{" "}

    D1 -->|"[No]"| A7(["Continue normal monitoring"])
    A7 --> M1

    M1 --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    classDef syncBar fill:#111,stroke:#111,color:#111;
    class Start,Stop termNode;
    class Fork1,Join1 syncBar;
```

**UML elements:** Initial node, decision/merge nodes, fork and join synchronization bars, activity-final node.

**Condition:** Use this workflow if the system evaluates readings from more than one sensor in the same monitoring cycle.

---

## 2. Device Selection Based on Scan Results

**Purpose:** Show different outcomes depending on the number of ESP devices found during scanning.

```mermaid
flowchart TD
    Start((" ")) --> ScanMerge{" "}
    ScanMerge --> A1(["Scan for available devices"])
    A1 --> A2(["Retrieve discovered devices"])
    A2 --> D1{"Devices found?"}

    D1 -->|"[No]"| A3(["Display 'No devices found'"])
    A3 --> D2{"Scan again?"}
    D2 -->|"[Yes]"| A4(["Restart scan"])
    A4 --> ScanMerge
    D2 -->|"[No]"| EndMerge{" "}

    D1 -->|"[Yes]"| D3{"Exactly one device?"}
    D3 -->|"[Yes]"| A5(["Select the discovered device"])
    A5 --> DeviceMerge{" "}
    D3 -->|"[No / Multiple devices]"| A6(["Display available devices"])
    A6 --> A7(["User selects a device"])
    A7 --> DeviceMerge

    DeviceMerge --> A8(["Send connection request"])
    A8 --> A9(["Test connection"])
    A9 --> D4{"Connection successful?"}
    D4 -->|"[Yes]"| A10(["Set device status to connected"])
    A10 --> ConnMerge{" "}
    D4 -->|"[No]"| A11(["Display connection failure"])
    A11 --> ConnMerge

    ConnMerge --> EndMerge
    EndMerge --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Nested decisions, dedicated merge nodes, alternative paths, and a scan retry loop.

**Condition:** Automatic selection when exactly one device is found is an assumption. Keep it only if that behavior is supported by the system design.

---

## 3. Historical Readings Filtering

**Purpose:** Retrieve readings for a selected time period and handle an empty database result.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Request historical readings"])
    A1 --> A2(["Select time period"])
    A2 --> A3(["Send database query"])
    A3 --> A4(["Retrieve matching readings"])
    A4 --> D1{"Any data available?"}

    D1 -->|"[Yes]"| A5(["Filter and organize readings"])
    A5 --> A6(["Generate historical chart"])
    A6 --> A7(["Display historical results"])
    A7 --> M1{" "}

    D1 -->|"[No]"| A8(["Display empty history"])
    A8 --> M1

    M1 --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Decision and merge of the data-available and empty-result paths.

---

## 4. Sensor Failure Handling During Reading

**Purpose:** Show how the app handles a sensor that does not provide a usable response.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Request sensor readings from ESP"])
    A1 --> A2(["Receive sensor response"])
    A2 --> D1{"Is the response valid?"}

    D1 -->|"[Yes]"| A3(["Process received readings"])
    A3 --> A4(["Display available readings"])
    A4 --> M1{" "}

    D1 -->|"[No]"| A5(["Mark sensor as unavailable"])
    A5 --> A6(["Check remaining sensor readings"])
    A6 --> A7(["Display available readings and sensor status"])
    A7 --> M1

    M1 --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Decision and merge with success and failure paths.

**Condition:** Use this only if the system can identify an unavailable or invalid sensor response and continue with other sensors.

---

## 5. Threshold Alert Decision and Alert Suppression

**Purpose:** Avoid generating a duplicate alert when an alert for the same threshold violation already exists.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Receive sensor reading"])
    A1 --> A2(["Check reading against threshold"])
    A2 --> D1{"Threshold exceeded?"}

    D1 -->|"[Yes]"| A3(["Check for an existing alert"])
    A3 --> D2{"Alert already exists?"}
    D2 -->|"[Yes]"| A4(["Update existing alert"])
    A4 --> AlertMerge{" "}
    D2 -->|"[No]"| A5(["Generate new alert"])
    A5 --> AlertMerge

    AlertMerge --> A6(["Store reading"])
    A6 --> OuterMerge{" "}

    D1 -->|"[No]"| A7(["Update normal status"])
    A7 --> OuterMerge

    OuterMerge --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Nested decisions, dedicated merge nodes, and alternative alert-handling paths.

**Important:** Checking for an existing alert, updating it, and suppressing duplicates were not confirmed as existing system functions. Treat this as an optional design idea unless those behaviors are present in the requirements or implementation.

---

## 6. Interrupted Reading Request

**Purpose:** Handle a reading request when the ESP does not respond.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Send reading request to ESP"])
    A1 --> LoopMerge{" "}
    LoopMerge --> A2(["Wait for ESP response"])
    A2 --> D1{"Response received?"}

    D1 -->|"[Yes]"| A3(["Validate received data"])
    A3 --> A4(["Display readings"])
    A4 --> EndMerge{" "}

    D1 -->|"[No]"| D2{"Retry limit reached?"}
    D2 -->|"[No]"| A5(["Retry reading request"])
    A5 --> LoopMerge
    D2 -->|"[Yes]"| A6(["Display communication error"])
    A6 --> EndMerge

    EndMerge --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Loop merge node, decisions, and alternative success/failure paths.

**Condition:** Include a retry limit only if the application has a defined retry policy. Otherwise, model failure handling according to the actual sequence diagram.

---

## 7. Environmental Monitoring Session Lifecycle

**Purpose:** Represent a monitoring session from initialization through repeated reading cycles to termination.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Start monitoring session"])
    A1 --> A2(["Verify ESP connection"])
    A2 --> D1{"ESP connected?"}

    D1 -->|"[Yes]"| A3(["Initialize sensors"])
    A3 --> CycleMerge{" "}
    CycleMerge --> A4(["Collect sensor readings"])
    A4 --> A5(["Update current readings"])
    A5 --> A6(["Store readings in database"])
    A6 --> D2{"Continue monitoring?"}
    D2 -->|"[Yes]"| CycleMerge
    D2 -->|"[No]"| A7(["Stop reading process"])
    A7 --> A8(["Close monitoring session"])
    A8 --> EndMerge{" "}

    D1 -->|"[No]"| A9(["Display connection error"])
    A9 --> EndMerge

    EndMerge --> Stop(((" ")))

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Decisions, repeated activity loop with merge node, and activity-final node.

**Important:** A user-controlled start/stop session and explicit session-closing behavior were not confirmed in the functions listed so far. Keep this diagram only if the system actually supports those actions.

---

## 8. Environmental Data Synchronization

**Purpose:** Represent sending readings from the application to the database and checking whether storage succeeded.

```mermaid
flowchart TD
    Start((" ")) --> A1(["Receive new sensor readings"])
    A1 --> A2(["Update current readings in application"])
    A2 --> A3(["Prepare readings for storage"])
    A3 --> SyncMerge{" "}

    SyncMerge --> A4(["Save readings to database"])
    A4 --> D1{"Storage successful?"}

    D1 -->|"[Yes]"| A5(["Mark data as synchronized"])
    A5 --> Stop(((" ")))

    D1 -->|"[No]"| A6(["Keep data pending"])
    A6 --> A7(["Retry database storage"])
    A7 --> SyncMerge

    classDef termNode fill:#111,stroke:#111,color:#fff;
    class Start,Stop termNode;
```

**UML elements:** Decision, merge node, and database write retry loop.

**Important:** Storage confirmation, pending-data status, and retrying database writes were not confirmed as existing system behaviors. Include them only if they are part of the implementation or approved requirements.

---

## Summary

| # | Activity diagram | Main UML modeling idea |
|---|---|---|
| 1 | Multiple Threshold Violations | Parallel alert generation and database storage (`fork` / `join`) |
| 2 | Device Selection Based on Scan Results | Branching by number of discovered devices and scan retry loop |
| 3 | Historical Readings Filtering | Data available versus empty result |
| 4 | Sensor Failure Handling During Reading | Failure path while handling sensor data |
| 5 | Threshold Alert Decision and Alert Suppression | Nested decisions for duplicate alerts |
| 6 | Interrupted Reading Request | Retry loop and communication failure |
| 7 | Environmental Monitoring Session Lifecycle | Repeated monitoring cycle |
| 8 | Environmental Data Synchronization | Database write confirmation and retry |

## Rendering the diagrams

The code blocks use **Mermaid** (`mermaid`), which renders natively in most modern Markdown environments:
* **GitHub & GitLab:** Automatically render `mermaid` blocks directly inside the README preview.
* **VS Code:** Supported natively in the built-in Markdown preview (`Ctrl+Shift+V` / `Cmd+Shift+V`) or via the **Markdown Preview Mermaid Support** extension.
* **Online Export:** Paste any block into **Mermaid Live Editor** (`mermaid.live`) to export high-resolution PNG or SVG images.

## Before using these diagrams in the project

- Keep each activity consistent with the system's actual sequence diagrams and requirements.
- Do not present optional assumptions as implemented functionality.
- Use fork/join synchronization bars only for genuinely parallel actions.
- Keep decision diamonds (1 incoming, 2+ guarded outgoing) separate from merge diamonds (2+ incoming, 1 outgoing).
- Rename actions to match the exact terminology used in the project.