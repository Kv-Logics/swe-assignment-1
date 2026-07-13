# Diagram 1: Data Flow Diagram (Level 0 — Context Diagram)

## Explanation

The Level 0 DFD (Context Diagram) shows **APIShield** as a single process bubble at the center, with all external entities that interact with it, and the high-level data flows entering and leaving the system. This is the highest level of abstraction — no internal processes and no data stores are broken out individually (the database is represented only implicitly inside the single process, per standard Gane–Sarson/Yourdon convention that Level 0 shows exactly one process). The external entities are the **Developer**, the **Security Tester**, the **Professor (Demo Viewer)**, and the **Target API** (the external REST API being scanned).

## Diagram

![DFD Level 0 Context Diagram](images/DFD_Level0_Context.png)

## PlantUML Code

```plantuml
@startuml DFD_Level0_Context
skinparam backgroundColor #FFFFFF
skinparam defaultFontName Arial
skinparam defaultFontSize 13
skinparam ArrowColor #333333
skinparam ArrowFontSize 11

skinparam rectangle {
  BackgroundColor #FDF6E3
  BorderColor #B58900
  FontStyle bold
}

left to right direction

actor "Developer" as Dev
actor "Security Tester" as Tester
actor "Professor\n(Demo Viewer)" as Prof
actor "Target API\n(External System)" as API

rectangle "0\nAPIShield\n(Automated API Security\nTesting System)" as Sys #FDF6E3

Dev --> Sys : Base URL /\nPostman Collection /\nOpenAPI Spec
Dev --> Sys : Start Scan Command
Dev <-- Sys : PDF Security Report
Dev <-- Sys : Scan Status / Progress

Tester --> Sys : Manual Test Trigger\n(Auth / IDOR / SQLi / Rate Limit)
Tester <-- Sys : Test Results & Evidence

Prof --> Sys : View Scan History Request
Prof <-- Sys : Scan History / Demo Report

Sys --> API : HTTP Test Requests\n(with / without auth,\nSQLi payloads, burst requests)
Sys <-- API : HTTP Responses\n(status codes, headers, body)

@enduml
```

## Mermaid Code

```mermaid
flowchart LR
    Dev([Developer]):::actor
    Tester([Security Tester]):::actor
    Prof([Professor / Demo Viewer]):::actor
    API([Target API - External System]):::actor

    Sys(("0\nAPIShield\nAutomated API Security\nTesting System")):::process

    Dev -- "Base URL / Postman Collection / OpenAPI Spec" --> Sys
    Dev -- "Start Scan Command" --> Sys
    Sys -- "PDF Security Report" --> Dev
    Sys -- "Scan Status / Progress" --> Dev

    Tester -- "Manual Test Trigger (Auth/IDOR/SQLi/Rate Limit)" --> Sys
    Sys -- "Test Results & Evidence" --> Tester

    Prof -- "View Scan History Request" --> Sys
    Sys -- "Scan History / Demo Report" --> Prof

    Sys -- "HTTP Test Requests (with/without auth, SQLi payloads, burst requests)" --> API
    API -- "HTTP Responses (status, headers, body)" --> Sys

    classDef actor fill:#FDEBD0,stroke:#CA6F1E,stroke-width:2px,color:#000;
    classDef process fill:#D6EAF8,stroke:#2874A6,stroke-width:3px,color:#000,font-weight:bold;
```

## Notes

- **Assumption:** The Database is intentionally *not* shown as a separate entity at Level 0, per standard DFD convention — internal data stores only appear starting at Level 1. It is represented implicitly inside process "0".
- **Assumption:** "Target API" is modeled as an external entity because it is a system *external* to APIShield's boundary, even though APIShield initiates the calls to it (this is standard for systems being tested/monitored).
- The Professor is modeled as a read-only actor (views scan history / demo reports) since the spec describes them as a "Demo Viewer."
- Both PlantUML and Mermaid versions render as a single-process context diagram consistent with Gane-Sarson notation adapted for Mermaid's flowchart engine (Mermaid has no native DFD shape library, so a circle/rounded node is used for the process and stadium shapes for external entities).
- Both code blocks above have been **syntax-validated**: the PlantUML compiles cleanly with PlantUML v1.2024.7, and the Mermaid code passes `mermaid.parse()` with no errors.
