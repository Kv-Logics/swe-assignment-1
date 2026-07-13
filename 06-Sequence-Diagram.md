# Diagram 6: Sequence Diagram

## Explanation

This Sequence Diagram traces one complete scan from the Developer's initial request through to PDF download, showing every major object described in the spec: **Developer**, **Frontend**, **Backend**, **APIParser**, **SecurityTestEngine**, the external **Target API**, the **Database**, and the **ReportGenerator**. It uses activation bars to show when each object is actively processing, a `loop` frame to iterate the five security tests over every discovered endpoint, and an `alt`/`else` frame to branch on whether a vulnerability was detected for a given test (storing either a finding or a "Pass" result). Return messages (dashed arrows) are shown for every synchronous call that produces a response.

## Diagram

![Sequence Diagram](images/Sequence_Diagram.png)

## PlantUML Code

```plantuml
@startuml Sequence_Diagram
skinparam backgroundColor #FFFFFF
skinparam defaultFontName Arial
skinparam defaultFontSize 12
skinparam sequenceMessageAlign center
skinparam maxMessageSize 160

actor Developer as Dev
participant "Frontend\n(React)" as FE
participant "Backend\n(FastAPI)" as BE
participant "APIParser\n(prance)" as Parser
participant "SecurityTestEngine\n(httpx)" as TestEngine
participant "Target API" as API
database "Database\n(PostgreSQL)" as DB
participant "ReportGenerator\n(reportlab)" as Report

Dev -> FE : Submit Base URL / Postman /\nOpenAPI Spec + Start Scan
activate FE

FE -> BE : POST /api/scans\n(spec data)
activate BE

BE -> Parser : parse(specData)
activate Parser
Parser --> BE : endpointList[methods, headers, params]
deactivate Parser

BE -> DB : INSERT INTO endpoints(...)
activate DB
DB --> BE : ack
deactivate DB

BE --> FE : 202 Accepted\n{scanId, status: "Scanning"}
FE --> Dev : Show "Scan in progress"

BE -> TestEngine : runTests(endpointList)
activate TestEngine

loop for each endpoint in endpointList
  TestEngine -> API : Request without auth token
  activate API
  API --> TestEngine : HTTP Response
  deactivate API
  TestEngine -> TestEngine : evaluate(AuthBypass)

  TestEngine -> API : Request as User A\nfor User B resource (IDOR)
  activate API
  API --> TestEngine : HTTP Response
  deactivate API
  TestEngine -> TestEngine : evaluate(IDOR)

  TestEngine -> API : Request with payload\n' OR 1=1--
  activate API
  API --> TestEngine : HTTP Response
  deactivate API
  TestEngine -> TestEngine : evaluate(SQLInjection)

  TestEngine -> API : Burst requests (rate limit test)
  activate API
  API --> TestEngine : HTTP Responses
  deactivate API
  TestEngine -> TestEngine : evaluate(RateLimiting)

  TestEngine -> TestEngine : scanResponseBody(SensitiveDataScanner)

  alt vulnerability detected
    TestEngine -> BE : reportFinding(severity, evidence)
    BE -> DB : INSERT INTO scan_results(...)
    activate DB
    DB --> BE : ack
    deactivate DB
  else no vulnerability
    TestEngine -> BE : reportFinding(status: "Pass")
    BE -> DB : INSERT INTO scan_results(...)
    activate DB
    DB --> BE : ack
    deactivate DB
  end
end

TestEngine --> BE : allTestsComplete()
deactivate TestEngine

BE -> Report : generateReport(scanId)
activate Report
Report -> DB : SELECT * FROM scan_results\nWHERE scan_id = :scanId
activate DB
DB --> Report : resultRows[]
deactivate DB
Report -> Report : buildPDF(severity, evidence,\nsuggestedFix, summary)
Report --> BE : pdfFilePath
deactivate Report

BE -> DB : UPDATE scans SET status='Completed'
activate DB
DB --> BE : ack
deactivate DB

BE --> FE : 200 OK\n{reportUrl, status:"Completed"}
deactivate BE

FE --> Dev : Display report summary
Dev -> FE : Click "Download PDF"
FE -> BE : GET /api/reports/{scanId}/download
activate BE
BE --> FE : PDF binary stream
deactivate BE
FE --> Dev : Save PDF file
deactivate FE

@enduml
```

## Mermaid Code

```mermaid
sequenceDiagram
    actor Dev as Developer
    participant FE as Frontend (React)
    participant BE as Backend (FastAPI)
    participant Parser as APIParser (prance)
    participant TestEngine as SecurityTestEngine (httpx)
    participant API as Target API
    participant DB as Database (PostgreSQL)
    participant Report as ReportGenerator (reportlab)

    Dev ->> FE: Submit Base URL / Postman / OpenAPI Spec + Start Scan
    activate FE
    FE ->> BE: POST /api/scans (spec data)
    activate BE

    BE ->> Parser: parse(specData)
    activate Parser
    Parser -->> BE: endpointList [methods, headers, params]
    deactivate Parser

    BE ->> DB: INSERT INTO endpoints(...)
    activate DB
    DB -->> BE: ack
    deactivate DB

    BE -->> FE: 202 Accepted {scanId, status: Scanning}
    FE -->> Dev: Show "Scan in progress"

    BE ->> TestEngine: runTests(endpointList)
    activate TestEngine

    loop for each endpoint in endpointList
        TestEngine ->> API: Request without auth token
        activate API
        API -->> TestEngine: HTTP Response
        deactivate API
        TestEngine ->> TestEngine: evaluate(AuthBypass)

        TestEngine ->> API: Request as User A for User B resource (IDOR)
        activate API
        API -->> TestEngine: HTTP Response
        deactivate API
        TestEngine ->> TestEngine: evaluate(IDOR)

        TestEngine ->> API: Request with payload ' OR 1=1--
        activate API
        API -->> TestEngine: HTTP Response
        deactivate API
        TestEngine ->> TestEngine: evaluate(SQLInjection)

        TestEngine ->> API: Burst requests (rate limit test)
        activate API
        API -->> TestEngine: HTTP Responses
        deactivate API
        TestEngine ->> TestEngine: evaluate(RateLimiting)

        TestEngine ->> TestEngine: scanResponseBody(SensitiveDataScanner)

        alt vulnerability detected
            TestEngine ->> BE: reportFinding(severity, evidence)
            BE ->> DB: INSERT INTO scan_results(...)
            activate DB
            DB -->> BE: ack
            deactivate DB
        else no vulnerability
            TestEngine ->> BE: reportFinding(status: Pass)
            BE ->> DB: INSERT INTO scan_results(...)
            activate DB
            DB -->> BE: ack
            deactivate DB
        end
    end

    TestEngine -->> BE: allTestsComplete()
    deactivate TestEngine

    BE ->> Report: generateReport(scanId)
    activate Report
    Report ->> DB: SELECT * FROM scan_results WHERE scan_id = scanId
    activate DB
    DB -->> Report: resultRows[]
    deactivate DB
    Report ->> Report: buildPDF(severity, evidence, suggestedFix, summary)
    Report -->> BE: pdfFilePath
    deactivate Report

    BE ->> DB: UPDATE scans SET status='Completed'
    activate DB
    DB -->> BE: ack
    deactivate DB

    BE -->> FE: 200 OK {reportUrl, status: Completed}
    deactivate BE

    FE -->> Dev: Display report summary
    Dev ->> FE: Click "Download PDF"
    FE ->> BE: GET /api/reports/scanId/download
    activate BE
    BE -->> FE: PDF binary stream
    deactivate BE
    FE -->> Dev: Save PDF file
    deactivate FE
```

## Notes

- **Assumption:** All five security tests are shown sequentially *within* the `loop` over endpoints, rather than as five separate top-level loops, since they logically apply per-endpoint (this matches the Activity Diagram's structure and keeps the sequence readable).
- **Assumption:** The `alt`/`else` block is shown once (conceptually applying after each test's evaluation) rather than duplicated five times, to keep the diagram legible while still fully representing the conditional storage behavior described in the spec ("Checks unexpected behavior" / "Checks authorization failures").
- Self-messages (e.g., `TestEngine -> TestEngine : evaluate(...)`) represent internal computation/evaluation steps that do not involve another object, which is standard UML sequence notation for internal logic.
- Both code blocks are **syntax-validated**: PlantUML compiles cleanly; Mermaid passes `mermaid.parse()` with diagram type `sequence`.
