# Diagram 5: State Diagram

## Explanation

This State Diagram models the lifecycle of a single **Scan Job** object, from creation through to its terminal outcome. A scan starts in `Idle`, moves to `WaitingForInput` once a user opens the scan form, then transitions through `Parsing` and `Scanning` before entering the composite state `RunningTests` — an internal sub-state machine representing the five security tests executing in sequence for the current endpoint. From there the job cycles through `SavingResults` (looping back to `RunningTests` if more endpoints remain), then `GeneratingReport`, finally reaching `Completed`. A `Failed` state is reachable from any processing stage on unrecoverable error, and a `Cancelled` state is reachable from any pre-completion state if the user aborts the scan.

## Diagram

![State Diagram](images/State_Diagram.png)

## PlantUML Code

```plantuml
@startuml State_Diagram
skinparam backgroundColor #FFFFFF
skinparam defaultFontName Arial
skinparam defaultFontSize 12
skinparam state {
  BackgroundColor #EBF5FB
  BorderColor #2874A6
}

[*] --> Idle

Idle --> WaitingForInput : User opens\nnew scan form

WaitingForInput --> Parsing : Input submitted\n(URL / Postman / OpenAPI)
WaitingForInput --> Idle : User cancels

Parsing --> Scanning : Endpoints extracted\nsuccessfully
Parsing --> Failed : Parsing error\n(invalid spec / unreachable URL)

Scanning --> RunningTests : Endpoint list ready

state RunningTests {
  [*] --> AuthTest
  AuthTest --> IDORTest
  IDORTest --> SQLiTest
  SQLiTest --> RateLimitTest
  RateLimitTest --> SensitiveDataTest
  SensitiveDataTest --> [*]
}

RunningTests --> SavingResults : All tests executed\nfor current endpoint
RunningTests --> RunningTests : More endpoints\nremaining
RunningTests --> Failed : Unrecoverable\ntest engine error

SavingResults --> RunningTests : More endpoints\nqueued
SavingResults --> GeneratingReport : All endpoints\nprocessed

GeneratingReport --> Completed : PDF generated\nsuccessfully
GeneratingReport --> Failed : Report generation\nerror

Completed --> [*]
Failed --> [*]

Idle --> Cancelled : User cancels\nbefore start
WaitingForInput --> Cancelled : User cancels
Parsing --> Cancelled : User cancels scan
Scanning --> Cancelled : User cancels scan
RunningTests --> Cancelled : User cancels scan
SavingResults --> Cancelled : User cancels scan
GeneratingReport --> Cancelled : User cancels scan
Cancelled --> [*]

@enduml
```

## Mermaid Code

```mermaid
stateDiagram-v2
    [*] --> Idle

    Idle --> WaitingForInput : User opens new scan form

    WaitingForInput --> Parsing : Input submitted (URL/Postman/OpenAPI)
    WaitingForInput --> Idle : User cancels

    Parsing --> Scanning : Endpoints extracted successfully
    Parsing --> Failed : Parsing error

    Scanning --> RunningTests : Endpoint list ready

    state RunningTests {
        [*] --> AuthTest
        AuthTest --> IDORTest
        IDORTest --> SQLiTest
        SQLiTest --> RateLimitTest
        RateLimitTest --> SensitiveDataTest
        SensitiveDataTest --> [*]
    }

    RunningTests --> SavingResults : All tests executed for endpoint
    RunningTests --> RunningTests : More endpoints remaining
    RunningTests --> Failed : Unrecoverable test engine error

    SavingResults --> RunningTests : More endpoints queued
    SavingResults --> GeneratingReport : All endpoints processed

    GeneratingReport --> Completed : PDF generated successfully
    GeneratingReport --> Failed : Report generation error

    Completed --> [*]
    Failed --> [*]

    Idle --> Cancelled : User cancels before start
    WaitingForInput --> Cancelled : User cancels
    Parsing --> Cancelled : User cancels scan
    Scanning --> Cancelled : User cancels scan
    RunningTests --> Cancelled : User cancels scan
    SavingResults --> Cancelled : User cancels scan
    GeneratingReport --> Cancelled : User cancels scan
    Cancelled --> [*]
```

## Notes

- **Assumption:** `RunningTests` is modeled as a **composite state** with an internal sub-state-machine (Auth → IDOR → SQLi → RateLimit → SensitiveData) to reflect that the five tests execute sequentially per endpoint within a single higher-level "testing" state, while the outer loop (`RunningTests` → `SavingResults` → `RunningTests`) handles moving between endpoints.
- **Assumption:** `Cancelled` is reachable from every pre-terminal state (`Idle` through `GeneratingReport`) since the spec lists it as a required state but does not specify exactly when cancellation is allowed; modeling it as available throughout the active lifecycle is the safest and most realistic interpretation for a user-facing scan tool.
- `Completed`, `Failed`, and `Cancelled` are all modeled as distinct terminal states feeding into the final state `[*]`, which is valid UML — a state machine may have multiple paths to termination.
- Both code blocks are **syntax-validated**: PlantUML compiles cleanly; Mermaid passes `mermaid.parse()` with diagram type `stateDiagram`.
