# Diagram 2: Data Flow Diagram (Level 1)

## Explanation

The Level 1 DFD decomposes process "0" (APIShield) from the Context Diagram into its five major internal processes: the **Frontend**, the **Backend API Server**, the **API Spec Parser**, the **Security Test Engine**, and the **Report Generator**. It introduces the single persistent data store, **D1: PostgreSQL**, which holds endpoints, scan results, and report metadata. The diagram traces the full data flow: a developer's input travels from the Frontend, through the Backend, into the Parser for endpoint extraction, then into the Test Engine which talks to the external Target API, with all results persisted to the database and ultimately assembled into a PDF by the Report Generator.

## Diagram

![DFD Level 1 Diagram](images/DFD_Level1.png)

## PlantUML Code

```plantuml
@startuml DFD_Level1
skinparam backgroundColor #FFFFFF
skinparam defaultFontName Arial
skinparam defaultFontSize 12
skinparam ArrowColor #333333
skinparam ArrowFontSize 10
skinparam rectangle {
  BackgroundColor #EBF5FB
  BorderColor #2874A6
}
skinparam database {
  BackgroundColor #FDEBD0
  BorderColor #CA6F1E
}

actor "Developer" as Dev
actor "Security Tester" as Tester
actor "Target API" as ExtAPI

rectangle "1.0\nFrontend\n(React + Tailwind)" as P1
rectangle "2.0\nBackend API Server\n(FastAPI)" as P2
rectangle "3.0\nAPI Spec Parser\n(prance)" as P3
rectangle "4.0\nSecurity Test Engine\n(httpx)" as P4
rectangle "5.0\nReport Generator\n(reportlab)" as P5
database "D1: PostgreSQL\n(Scans, Endpoints,\nResults, Reports)" as DB

Dev --> P1 : Base URL / Postman File /\nOpenAPI Spec / Start Scan
P1 --> P2 : Scan Request (HTTP/JSON)
P2 --> P1 : Scan Status / Results (JSON)

P2 --> P3 : Raw Spec / Base URL
P3 --> P2 : Parsed Endpoint List\n(methods, headers, params)
P2 --> DB : Store Endpoint List

P2 --> P4 : Endpoint List + Test Config
P4 --> ExtAPI : HTTP Test Requests\n(Auth Bypass, IDOR,\nSQLi, Rate Limit, Data Scan)
ExtAPI --> P4 : HTTP Responses
P4 --> P2 : Test Results\n(Severity, Evidence)
P2 --> DB : Store Scan Results

P2 --> P5 : Request Report Generation
P5 --> DB : Fetch Scan Results
DB --> P5 : Scan Result Records
P5 --> P2 : Generated PDF Report
P2 --> P1 : PDF Report / Download Link
P1 --> Dev : Display Report

Tester --> P1 : Manual Test Trigger
DB --> P2 : Scan History Records

@enduml
```

## Mermaid Code

```mermaid
flowchart TB
    Dev([Developer]):::actor
    Tester([Security Tester]):::actor
    ExtAPI([Target API]):::actor

    P1["1.0 Frontend\n(React + Tailwind)"]:::process
    P2["2.0 Backend API Server\n(FastAPI)"]:::process
    P3["3.0 API Spec Parser\n(prance)"]:::process
    P4["4.0 Security Test Engine\n(httpx)"]:::process
    P5["5.0 Report Generator\n(reportlab)"]:::process
    DB[("D1: PostgreSQL\nScans / Endpoints /\nResults / Reports")]:::store

    Dev -- "Base URL / Postman File / OpenAPI Spec / Start Scan" --> P1
    P1 -- "Scan Request JSON" --> P2
    P2 -- "Scan Status / Results" --> P1

    P2 -- "Raw Spec / Base URL" --> P3
    P3 -- "Parsed Endpoint List" --> P2
    P2 -- "Store Endpoint List" --> DB

    P2 -- "Endpoint List + Test Config" --> P4
    P4 -- "HTTP Test Requests" --> ExtAPI
    ExtAPI -- "HTTP Responses" --> P4
    P4 -- "Test Results (Severity, Evidence)" --> P2
    P2 -- "Store Scan Results" --> DB

    P2 -- "Request Report Generation" --> P5
    P5 -- "Fetch Scan Results" --> DB
    DB -- "Scan Result Records" --> P5
    P5 -- "Generated PDF Report" --> P2
    P2 -- "PDF Report / Download Link" --> P1
    P1 -- "Display Report" --> Dev

    Tester -- "Manual Test Trigger" --> P1
    DB -- "Scan History Records" --> P2

    classDef actor fill:#FDEBD0,stroke:#CA6F1E,stroke-width:2px,color:#000;
    classDef process fill:#D6EAF8,stroke:#2874A6,stroke-width:2px,color:#000;
    classDef store fill:#FCF3CF,stroke:#B7950B,stroke-width:2px,color:#000;
```

## Notes

- **Assumption:** Processes are numbered 1.0–5.0 in the order data primarily flows through them (Frontend → Backend → Parser → Test Engine → Report Generator), which is standard DFD leveling practice.
- **Assumption:** Only one data store (D1: PostgreSQL) is shown, since the spec describes a single relational database holding all scan-related entities (endpoints, results, reports) rather than separate physical stores.
- The Backend (2.0) acts as the central orchestrator — all inter-process communication and all database writes are routed through it, reflecting the FastAPI backend's role as the system's controller layer.
- Both code blocks are **syntax-validated**: PlantUML compiles cleanly; Mermaid passes `mermaid.parse()` with diagram type `flowchart-v2`.
