# Diagram 4: Activity Diagram

## Explanation

This Activity Diagram models the complete end-to-end workflow of a single scan, from the developer's initial input through validation, parsing, a **fork** into the five parallel-capable security tests, a **loop** that iterates the full test suite over every discovered endpoint, result storage, PDF report generation, and final display. Two decision points (input validation, parsing success) allow early termination on bad input, and a third decision point governs the report-generation outcome. The fork/join pair represents that the five security tests are conceptually independent checks that could run concurrently against a given request context, while the surrounding loop drives them across the full endpoint list.

## Diagram

![Activity Diagram](images/Activity_Diagram.png)

## PlantUML Code

```plantuml
@startuml Activity_Diagram
skinparam backgroundColor #FFFFFF
skinparam defaultFontName Arial
skinparam defaultFontSize 12
skinparam activity {
  BackgroundColor #EBF5FB
  BorderColor #2874A6
  DiamondBackgroundColor #FDEBD0
  DiamondBorderColor #CA6F1E
}

start
:Developer provides input\n(Base URL / Postman Collection / OpenAPI Spec);
:Validate input format;

if (Input valid?) then (no)
  :Display validation error;
  stop
else (yes)
endif

:Parse API specification\n(extract endpoints, methods,\nheaders, parameters);

if (Parsing successful?) then (no)
  :Log parsing error;
  :Notify user of failure;
  stop
else (yes)
endif

:Initialize Scan Job\n(status = Scanning);

fork
  :Run Authentication\nBypass Test;
fork again
  :Run IDOR Test;
fork again
  :Run SQL Injection Test;
fork again
  :Run Rate Limiting Test;
fork again
  :Run Sensitive Data Scan;
end fork

repeat :Select next endpoint;
  :Execute security test suite\nagainst current endpoint;
  :Capture response, status code,\nheaders, and evidence;
  :Assign severity level\n(Critical / High / Medium / Low);
  :Store result in database;
repeat while (More endpoints remaining?) is (yes)
->no;

:Aggregate all scan results;
:Generate PDF report\n(summary, severity, evidence,\nsuggested fixes);

if (Report generation successful?) then (no)
  :Log report error;
  :Mark scan job as Failed;
  stop
else (yes)
endif

:Store report reference in database;
:Mark scan job as Completed;
:Display report to Developer;
:Developer downloads PDF report;

stop

@enduml
```

## Mermaid Code

```mermaid
flowchart TD
    Start([Start]) --> Input["Developer provides input:\nBase URL / Postman Collection / OpenAPI Spec"]
    Input --> Validate["Validate input format"]
    Validate --> D1{Input valid?}
    D1 -- No --> ErrEnd["Display validation error"] --> End1([End])
    D1 -- Yes --> Parse["Parse API specification\nextract endpoints, methods,\nheaders, parameters"]
    Parse --> D2{Parsing successful?}
    D2 -- No --> ErrLog["Log parsing error"] --> ErrNotify["Notify user of failure"] --> End2([End])
    D2 -- Yes --> Init["Initialize Scan Job\nstatus = Scanning"]

    Init --> ForkNode((( )))
    ForkNode --> T1["Run Authentication\nBypass Test"]
    ForkNode --> T2["Run IDOR Test"]
    ForkNode --> T3["Run SQL Injection Test"]
    ForkNode --> T4["Run Rate Limiting Test"]
    ForkNode --> T5["Run Sensitive Data Scan"]
    T1 --> JoinNode((( )))
    T2 --> JoinNode
    T3 --> JoinNode
    T4 --> JoinNode
    T5 --> JoinNode

    JoinNode --> LoopSelect["Select next endpoint"]
    LoopSelect --> Exec["Execute security test suite\nagainst current endpoint"]
    Exec --> Capture["Capture response, status code,\nheaders, and evidence"]
    Capture --> Severity["Assign severity level\nCritical / High / Medium / Low"]
    Severity --> StoreResult["Store result in database"]
    StoreResult --> D3{More endpoints\nremaining?}
    D3 -- Yes --> LoopSelect
    D3 -- No --> Aggregate["Aggregate all scan results"]

    Aggregate --> GenReport["Generate PDF report\nsummary, severity, evidence,\nsuggested fixes"]
    GenReport --> D4{Report generation\nsuccessful?}
    D4 -- No --> ErrReport["Log report error"] --> MarkFailed["Mark scan job as Failed"] --> End3([End])
    D4 -- Yes --> StoreRef["Store report reference in database"]
    StoreRef --> MarkComplete["Mark scan job as Completed"]
    MarkComplete --> Display["Display report to Developer"]
    Display --> Download["Developer downloads PDF report"]
    Download --> End4([End])

    classDef decision fill:#FDEBD0,stroke:#CA6F1E,stroke-width:2px;
    class D1,D2,D3,D4 decision;
```

## Notes

- **Assumption:** The fork/join block represents the five security tests as *conceptually parallel* activities (they don't depend on each other's output), even though the loop below re-sequences them per endpoint. This satisfies the requirement for "Forks, Joins" in true UML activity notation, which Mermaid approximates using small circular fork/join bar nodes.
- **Assumption:** Three separate terminal (`stop`/`End`) nodes are used for the three distinct failure exits (invalid input, parsing failure, report failure), which is standard UML activity practice — an activity diagram can have multiple final nodes.
- The loop uses PlantUML's native `repeat`/`repeat while` construct (a proper UML loop node), and the Mermaid equivalent uses a conditional back-edge to reproduce the same semantics, since Mermaid flowcharts have no dedicated loop node.
- Both code blocks are **syntax-validated**: PlantUML compiles cleanly; Mermaid passes `mermaid.parse()` with diagram type `flowchart-v2`.
