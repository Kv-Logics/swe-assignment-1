# Diagram 3: Use Case Diagram

## Explanation

This Use Case Diagram captures how the three human actors — **Developer**, **Security Tester**, and **Professor (Demo Viewer)** — interact with APIShield, plus two supporting actors representing external systems the use cases depend on: the **External API** (the target being scanned) and the **Database**. The central "Start Scan" use case `<<include>>`s endpoint parsing and all five security tests, since a scan always requires them. Input-method use cases (Upload Postman, Enter API URL, Upload OpenAPI Spec) are modeled as `<<extend>>` points of "Parse Endpoints" because only one of the three is used per scan, depending on the developer's choice. Viewing a report `<<extend>>`s into report generation only if a PDF hasn't already been generated for that scan.

## Diagram

![Use Case Diagram](images/UseCase_Diagram.png)

## PlantUML Code

```plantuml
@startuml UseCase_Diagram
skinparam backgroundColor #FFFFFF
skinparam defaultFontName Arial
skinparam defaultFontSize 12
skinparam usecase {
  BackgroundColor #EBF5FB
  BorderColor #2874A6
}
left to right direction

actor "Developer" as Dev
actor "Security Tester" as Tester
actor "Professor\n(Demo Viewer)" as Prof
actor "External API" as ExtAPI
actor "Database" as DB

rectangle "APIShield System" {

  usecase "Upload Postman\nCollection" as UC1
  usecase "Enter API URL" as UC2
  usecase "Upload OpenAPI\nSpecification" as UC2b
  usecase "Start Scan" as UC3
  usecase "Parse Endpoints" as UC3b
  usecase "Run Authentication\nBypass Test" as UC4
  usecase "Run IDOR Test" as UC5
  usecase "Run SQL Injection\nTest" as UC6
  usecase "Run Rate Limit\nTest" as UC7
  usecase "Run Sensitive Data\nScan" as UC8
  usecase "Send HTTP Request\nto Target API" as UC8b
  usecase "Store Scan Results" as UC9
  usecase "View Report" as UC10
  usecase "Download PDF" as UC11
  usecase "Generate PDF\nReport" as UC11b
  usecase "View Scan History" as UC12
  usecase "Login / Provide\nCredentials" as UC13
}

Dev --> UC1
Dev --> UC2
Dev --> UC2b
Dev --> UC3
Dev --> UC10
Dev --> UC11
Dev --> UC12

Tester --> UC4
Tester --> UC5
Tester --> UC6
Tester --> UC7
Tester --> UC8
Tester --> UC12

Prof --> UC10
Prof --> UC12

UC3 ..> UC3b : <<include>>
UC3b ..> UC1 : <<extend>>
UC3b ..> UC2 : <<extend>>
UC3b ..> UC2b : <<extend>>

UC3 ..> UC4 : <<include>>
UC3 ..> UC5 : <<include>>
UC3 ..> UC6 : <<include>>
UC3 ..> UC7 : <<include>>
UC3 ..> UC8 : <<include>>

UC5 ..> UC13 : <<include>>

UC4 ..> UC8b : <<include>>
UC5 ..> UC8b : <<include>>
UC6 ..> UC8b : <<include>>
UC7 ..> UC8b : <<include>>
UC8 ..> UC8b : <<include>>

UC8b --> ExtAPI

UC3 ..> UC9 : <<include>>
UC9 --> DB

UC10 ..> UC11b : <<extend>>
UC11 ..> UC11b : <<include>>
UC10 --> DB
UC12 --> DB

@enduml
```

## Mermaid Code

```mermaid
graph LR
    Dev([Developer]):::actor
    Tester([Security Tester]):::actor
    Prof([Professor / Demo Viewer]):::actor
    ExtAPI([External API]):::actor
    DB([Database]):::actor

    subgraph System["APIShield System"]
        UC1(("Upload Postman\nCollection"))
        UC2(("Enter API URL"))
        UC2b(("Upload OpenAPI\nSpecification"))
        UC3(("Start Scan"))
        UC3b(("Parse Endpoints"))
        UC4(("Run Authentication\nBypass Test"))
        UC5(("Run IDOR Test"))
        UC6(("Run SQL Injection\nTest"))
        UC7(("Run Rate Limit\nTest"))
        UC8(("Run Sensitive Data\nScan"))
        UC8b(("Send HTTP Request\nto Target API"))
        UC9(("Store Scan Results"))
        UC10(("View Report"))
        UC11(("Download PDF"))
        UC11b(("Generate PDF\nReport"))
        UC12(("View Scan History"))
        UC13(("Login / Provide\nCredentials"))
    end

    Dev --> UC1
    Dev --> UC2
    Dev --> UC2b
    Dev --> UC3
    Dev --> UC10
    Dev --> UC11
    Dev --> UC12

    Tester --> UC4
    Tester --> UC5
    Tester --> UC6
    Tester --> UC7
    Tester --> UC8
    Tester --> UC12

    Prof --> UC10
    Prof --> UC12

    UC3 -. "include" .-> UC3b
    UC3b -. "extend" .-> UC1
    UC3b -. "extend" .-> UC2
    UC3b -. "extend" .-> UC2b

    UC3 -. "include" .-> UC4
    UC3 -. "include" .-> UC5
    UC3 -. "include" .-> UC6
    UC3 -. "include" .-> UC7
    UC3 -. "include" .-> UC8

    UC5 -. "include" .-> UC13

    UC4 -. "include" .-> UC8b
    UC5 -. "include" .-> UC8b
    UC6 -. "include" .-> UC8b
    UC7 -. "include" .-> UC8b
    UC8 -. "include" .-> UC8b

    UC8b --> ExtAPI

    UC3 -. "include" .-> UC9
    UC9 --> DB

    UC10 -. "extend" .-> UC11b
    UC11 -. "include" .-> UC11b
    UC10 --> DB
    UC12 --> DB

    classDef actor fill:#FDEBD0,stroke:#CA6F1E,stroke-width:2px,color:#000;
```

## Notes

- **Assumption:** "Database" and "External API" are modeled as secondary (system) actors on the use case diagram, per UML convention that any external system a use case interacts with can be shown as an actor, not just human roles.
- **Assumption:** IDOR testing is the only test that explicitly `<<include>>`s a login/credential step, since the spec describes it as requiring "logs in as User A" — the other tests do not require authenticated session setup.
- **Assumption:** "View Report" `<<extend>>`s "Generate PDF Report" rather than always including it, modeling the case where a report may already exist and simply needs to be displayed/fetched.
- Mermaid has **no native use-case-diagram type**; the standard workaround (used above) is a `graph`/`flowchart` with a `subgraph` boundary and circular/stadium nodes to approximate use-case ovals, with dotted edges labeled "include"/"extend" standing in for the `<<include>>`/`<<extend>>` stereotypes. PlantUML supports true use-case notation natively.
- Both code blocks are **syntax-validated**: PlantUML compiles cleanly; Mermaid passes `mermaid.parse()` with diagram type `flowchart-v2`.
