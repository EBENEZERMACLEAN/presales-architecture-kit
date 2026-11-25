# Architecture document — Analytics platform (v0.1)

> Résumé (FR) : dossier d'architecture (gabarit arc42 allégé, diagrammes C4) d'une plateforme analytique pour un client fictif. Version 0.1 : objectifs, exigences de qualité, contraintes, contexte, principes et conteneurs. Données 100 % synthétiques, aucun élément tiré d'un mandat réel.

| Field | Value |
|---|---|
| Status | Draft |
| Version | 0.1 |
| Client | Client Démo (fictitious) |
| Author | Ebenezer MACLEAN |
| Last updated | 2026-11-25 |
| Decision makers | CIO, Head of Analytics |
| Templates | arc42 (lightweight) · C4 model · Azure Well-Architected Framework |

## 1. Introduction and goals

### 1.1 Requirements overview

- Provide store managers with a daily overview of their stores' performance
- Help the replenishment team plan replenishment more effectively and prevent stock-outs caused by operational inefficiencies
- Give directors a consolidated view of revenue, sales performance and replenishment efficiency
- Keep 3 years of sales and stock history so that directors and analysts can compare trends year over year

### 1.2 Quality goals

| Priority | Quality goal | Motivation | Target |
|---|---|---|---|
| 1 | Data freshness | Store managers review the previous day's results before opening | Previous day's data available before 8:00 a.m. on 95% of store opening days (Q3, Q4) |
| 2 | Report responsiveness | Store managers have only a few minutes to check their report before opening | • Page fully rendered within 5 s, 98% of loads (Q1)<br>• Visuals refreshed within 5 s after a filter change, 98% of interactions (Q2) |
| 3 | Personal data confidentiality | Loyalty program data is personal information protected under Québec's Law 25 | • 0 direct identifiers visible outside the authorized roles (Q7)<br>• 100% of loyalty data reads logged (Q8)<br>• 100% of exposure incidents handled and recorded (Q9) |
| 4 | Mobile usability | Most store managers check their report on their mobile phone | • 100% of KPIs visible on a mobile phone screen without horizontal scrolling (Q5)<br>• 100% of filters usable by touch (Q6) |
| 5 | Historical data completeness | Directors and analysts compare sales trends year over year | • 100% of days from the last 36 months available (Q10)<br>• 36-month reports rendered within 5 s, 98% of loads (Q1) |
| 6 | Cost efficiency | The platform runs on an annual Fabric capacity budget capped by the organization | • Monthly Fabric cost within the monthly share of the approved annual budget, 12 months out of 12 (Q11)<br>• Alert sent when cost reaches 80% of that monthly share (Q11) |

### 1.3 Stakeholders

| Stakeholder | Main concern | Answered by |
|---|---|---|
| Directors | Compare sales trends and replenishment efficiency year over year | 5 |
| Privacy officer | Keep loyalty data compliant with Law 25 and be alerted of any exposure | 3 |
| Analysts | Query sales history, including loyalty data, for analysis | 3, 5 |
| Store managers | See results from the previous day on their mobile phone before opening | 1, 2, 4 |
| Replenishment team | Review the previous day's stock levels and open purchase orders by store and product to prevent stock-outs | 1, 2 |
| Data team – implementation | Clear requirements and architecture decisions to build the pipelines, model and reports | 1, 2, 3, 4, 5 |
| Data team – operations | Run and monitor the nightly load, capacity usage and access logs, and be alerted when something fails | 1, 3, 6 |
| Budget owner | Keep the platform within the approved Fabric budget | 6 |

## 2. Constraints

| Constraint | Type | Consequence for the architecture |
|---|---|---|
| Loyalty program data is personal information under Québec's Law 25 | Regulatory | Direct identifiers restricted to authorized roles, all loyalty data reads logged, incident register and notification process in place (Q7 to Q9) |
| All data hosted in Canada | Organizational | Fabric capacity and storage deployed in a Canadian region, and no service that stores data outside Canada |
| Microsoft Fabric is the only data platform (existing Microsoft contract) | Technical | All components (ingestion, storage, transformation, reports) built with Fabric items |
| Annual Fabric capacity budget, capped by the organization (amount to be confirmed) | Organizational | Capacity size chosen to fit the budget, with usage and cost monitored (Q11) |
| Data team of 3 people, proficient in SQL and Power BI, with no Spark experience | Organizational | SQL-based Fabric items favored over Spark notebooks to keep the platform maintainable by the team (see ADR-001, v0.2) |
| ERP (SQL Server) hosted on premises, reachable only from the client's network | Technical | ERP data cannot be reached directly from Fabric: a gateway inside the client's network or a private network link is required. Needs to be hosted and maintained by the client (Section 5) |

## 3. Context and scope

~~~mermaid
---
title: "System context (C4 level 1) — Analytics platform — Client Démo (fictitious) — v0.1"
---
flowchart LR
  manager["<b>Store manager</b><br/>[Person]<br/>Checks the previous day's<br/>sales and stock-outs"]
  director["<b>Director</b><br/>[Person]<br/>Compares sales trends and<br/>replenishment efficiency<br/>year over year"]
  replenishment["<b>Replenishment team</b><br/>[Person]<br/>Reviews the previous day's<br/>stock levels and open purchase<br/>orders by store and product<br/>to prevent stock-outs"]
  analyst["<b>Analyst</b><br/>[Person]<br/>Queries curated sales and<br/>stock history, including<br/>loyalty data, for analysis"]
  dataops["<b>Data team – operations</b><br/>[Person]<br/>Runs and monitors<br/>the nightly load"]
  subgraph scope["Scope: analytics platform"]
    platform["<b>Analytics platform</b><br/>[Software system]<br/>Integrates sales, stock and<br/>loyalty data, serves reports"]
  end
  pos["<b>Point-of-sale system</b><br/>[External software system]<br/>Records every<br/>in-store checkout"]
  erp["<b>ERP (purchasing and stock)</b><br/>[External software system]<br/>Manages purchasing and<br/>stock levels, on premises"]
  ols["<b>Online store</b><br/>[External software system]<br/>Records every<br/>online checkout"]
  crm["<b>CRM (loyalty program)</b><br/>[External software system]<br/>Manages loyalty members and<br/>their personal information"]
  xlsx["<b>Marketing promotions file</b><br/>[External file]<br/>Lists every<br/>marketing promotion"]
  manager -- "Views daily<br/>sales report<br/>[HTTPS]" --> platform
  director -- "Views weekly<br/>sales report<br/>[HTTPS]" --> platform
  replenishment -- "Views daily stock and<br/>purchasing report<br/>[HTTPS]" --> platform
  analyst -- "Queries curated sales<br/>and stock history<br/>[SQL]" --> platform
  platform -- "Sends load failure<br/>and delay alerts<br/>[Email]" --> dataops
  platform -- "Pulls daily<br/>sales export<br/>[SFTP, CSV]" --> pos
  platform -- "Pulls daily purchasing<br/>and stock data<br/>[SQL]" --> erp
  platform -- "Pulls daily online<br/>sales export<br/>[REST API]" --> ols
  platform -- "Pulls loyalty members<br/>[Protocol to confirm]" --> crm
  platform -- "Pulls marketing promotions<br/>when the file is updated<br/>[XLSX on SharePoint]" --> xlsx
  classDef person fill:#08427b,color:#ffffff,stroke:#052e56
  classDef system fill:#1168bd,color:#ffffff,stroke:#0b4884
  classDef external fill:#999999,color:#ffffff,stroke:#6b6b6b
  class manager,director,replenishment,analyst,dataops person
  class platform system
  class pos,erp,ols,crm,xlsx external
~~~

Legend: dark blue = person · blue = system in scope · grey = external system · arrow = from the initiator to the dependency, label = intent [protocol].

Assumptions: loyalty member data comes from an existing CRM (access protocol to be confirmed); the promotions file is stored on SharePoint.

## 4. Solution strategy

### Architecture principles
#### P1 — One definition per KPI (Single source of truth)
**Statement:** Every business KPI has one business definition and one calculation, reused by every consumer.<br>
**Rationale:** Store reports, analysts and the future forecasting model must show the same numbers. A team of three can maintain each rule in one place only.<br>
**Implications:**
- A new KPI starts with its business definition in the glossary.
- Its calculation lives in a single shared layer, never duplicated in reports or notebooks.

#### P2 — Least privilege
**Statement:** Users and services can only access the data their tasks require.<br>
**Rationale:** Law 25 requires that personal information be accessible only to those who need it for their duties (Section 2). Limiting access also contains the damage if an account or service is compromised. For example, analysts see loyalty behaviour, not member identities (Q7).<br>
**Implications:**
- Access is granted by role, through groups managed by the client, never to individual accounts.
- Direct identifiers are hidden from roles that do not need them. Hidden identifiers do not make data anonymous.
- Services run under dedicated identities with only the permissions they need.

#### P3 — Decisions rely on certified data
**Statement:** Reports used for decision-making draw only on data that has been certified by its owner.<br>
**Rationale:** Decision-makers must trust the figures they use. Certification makes clear which data is reliable and who is accountable for it.<br>
**Implications:**
- Each certified dataset has a named data owner.
- Certification follows explicit criteria (quality checks, documentation, validated definitions).
- Uncertified data stays available for exploration but is clearly labelled as such.

## 5. Building block view

~~~mermaid
---
title: "Containers (C4 level 2) — Analytics platform — Client Démo (fictitious) — v0.1"
---
flowchart LR
  manager["Store manager<br/>[Person]"]
  director["Director<br/>[Person]"]
  replenishment["Replenishment team<br/>[Person]"]
  analyst["Analyst<br/>[Person]"]
  dataops["Data team – operations<br/>[Person]"]

  pos["Point-of-sale system<br/>[External software system]"]
  ols["Online store<br/>[External software system]"]
  crm["CRM (loyalty program)<br/>[External software system]"]
  xlsx["Marketing promotions file<br/>[External file on SharePoint]"]

  subgraph onprem["Client network (on-premises)"]
    gateway["On-premises data gateway<br/>[Infrastructure, operated by client IT]"]
    erp["ERP (purchasing and stock)<br/>[External software system, SQL Server]"]
  end

  subgraph platform["Analytics platform (Microsoft Fabric)"]
    pipeline["Ingestion pipelines<br/>[Container: Fabric Data Factory pipeline]<br/>Copies source data every night"]
    bronze["Raw data store<br/>[Container: Fabric Lakehouse, Delta tables]<br/>Keeps source data as received"]
    transform["Transformations<br/>[Container: T-SQL stored procedures]<br/>Cleans, conforms and models data"]
    silver["Cleaned data store<br/>[Container: Fabric Data Warehouse, silver schema]<br/>Deduplicated, standardized, validated data"]
    gold["Curated data store<br/>[Container: Fabric Data Warehouse, gold schema]<br/>Star schema for end-user consumption"]
    semantic["Semantic model<br/>[Container: Power BI semantic model, Direct Lake]<br/>Single definition of every KPI"]
    reports["Reports and app<br/>[Container: Power BI report and app]<br/>Daily store report, weekly executive view, stock-outs"]
  end

  pipeline -- "Reads daily export<br/>[SFTP, CSV]" --> pos
  pipeline -- "Reads purchasing and stock<br/>[Gateway connection]" --> gateway
  gateway -- "Queries<br/>[SQL]" --> erp
  pipeline -- "Reads online orders<br/>[REST API]" --> ols
  pipeline -- "Reads loyalty members<br/>[Protocol to confirm]" --> crm
  pipeline -- "Reads promotions<br/>[SharePoint, XLSX]" --> xlsx
  pipeline -- "Writes raw tables<br/>[OneLake, Delta]" --> bronze
  pipeline -- "Runs after ingestion" --> transform
  transform -- "Reads raw tables<br/>[SQL endpoint, T-SQL]" --> bronze
  transform -- "Writes cleaned tables<br/>[T-SQL]" --> silver
  transform -- "Writes curated tables<br/>[T-SQL]" --> gold
  semantic -- "Reads curated tables<br/>[Direct Lake]" --> gold
  reports -- "Reads KPIs<br/>[DAX]" --> semantic
  manager -- "Views daily report, often on mobile<br/>[HTTPS]" --> reports
  director -- "Views weekly review<br/>[HTTPS]" --> reports
  replenishment -- "Views stock-outs<br/>[HTTPS]" --> reports
  analyst -- "Explores data<br/>[SQL endpoint, TDS]" --> gold
  pipeline -- "Sends load failure<br/>and delay alerts<br/>[Email]" --> dataops

  classDef person fill:#08427b,color:#ffffff,stroke:#052e56
  classDef container fill:#438dd5,color:#ffffff,stroke:#2e6295
  classDef external fill:#999999,color:#ffffff,stroke:#6b6b6b
  classDef infra fill:#ffffff,color:#333333,stroke:#6b6b6b,stroke-dasharray:4 2
  class manager,director,replenishment,analyst,dataops person
  class pipeline,bronze,transform,silver,gold,semantic,reports container
  class pos,ols,crm,xlsx,erp external
  class gateway infra
~~~

Legend: dark blue = person · blue = container inside the platform · grey = external system · dashed white = infrastructure operated by client IT · arrow = from the initiator to the dependency, label = intent [protocol].

| Container | Technology | Responsibility | Data classification |
|---|---|---|---|
| Ingestion pipelines | Fabric Data Factory pipeline | Copies the five sources every night, runs the transformations, sends alerts | Restricted |
| Raw data store | Fabric Lakehouse, Delta tables | Keeps source data as received, with 3 years of history | Restricted |
| Transformations | T-SQL stored procedures | Deduplicates, standardizes and validates data, then builds the star schema | Restricted |
| Cleaned data store | Fabric Data Warehouse, silver schema | Holds validated, conformed data | Restricted |
| Curated data store | Fabric Data Warehouse, gold schema | Star schema; members' direct identifiers excluded (P2) | Confidential |
| Semantic model | Power BI semantic model, Direct Lake | Single definition of every KPI (P1) | Confidential |
| Reports and app | Power BI report and app | Daily store report before opening, weekly executive view, stock-outs | Confidential |

## 6. Runtime view

To be completed in a later version.

## 7. Deployment view

To be completed in a later version.

## 8. Crosscutting concepts

To be completed in a later version.

## 9. Architecture decisions

To be completed (ADR log, v0.2).


## 10. Quality requirements

| # | Pillar | Scenario | Measure |
|---|---|---|---|
| Q1 | Performance efficiency | A user opens a report page on a mobile phone or desktop, for any period up to 36 months, with 50 to 55 users connected at the same time | All visuals on the page rendered within 5 s for 98% of page loads over each month |
| Q2 | Performance efficiency | A user changes a filter on a report page, under the same conditions as Q1 | All visuals refreshed within 5 s for 98% of filter changes over each month |
| Q3 | Reliability | On a store opening day, the nightly load brings in the previous day's sales and stock data | Previous day's data available in store reports before 8:00 a.m. on 95% of store opening days over each quarter |
| Q4 | Operational excellence | The nightly load fails, or has not completed by 7:00 a.m. | Data team - operations alerted within 15 min for 100% of cases |
| Q5 | Usability | A store manager opens the daily report on a mobile phone (375 px wide screen) | 100% of KPIs visible without horizontal scrolling or zooming |
| Q6 | Usability | A store manager changes the store and date filters on the mobile report | • 100% of filters meet the minimum touch target size (44 × 44 px)<br>• Store and date changed in ≤ 3 taps |
| Q7 | Security | A store manager or director queries or exports sales that include loyalty purchases | 0 direct identifiers (name, email, phone, loyalty card number) returned outside the Admin and Analyst roles |
| Q8 | Security | Any user or process reads the loyalty tables | 100% of reads logged with user and timestamp. Logs kept 12 months |
| Q9 | Security | Direct identifiers from loyalty data are exposed outside the authorized roles (report, export or shared file) | • Privacy officer alerted within 1 h of detection<br>• Exposure removed within 4 h<br>• Security impact on the organization assessed within 24 h<br>• 100% of incidents recorded in the incident register |
| Q10 | Reliability | An analyst queries any day of the last 36 months | 100% of days present |
| Q11 | Cost optimization | The platform runs at its normal monthly volume | • Monthly Fabric cost within the monthly share of the approved annual budget, 12 months out of 12<br>• Alert sent when cost reaches 80% of that monthly share |

## 11. Risks and technical debt

To be completed (v0.2).

## 12. Glossary

| Term | Definition |
|---|---|
| CRM | Customer relationship management system; here, the existing system that holds loyalty members' data |
| Direct identifier | Data that identifies a loyalty member on its own: name, email, phone or loyalty card number |
| Store opening day | Any day the stores are open, weekends included (assumption to be confirmed) |

## 13. Cost estimate and business case

To be completed (v0.2).
