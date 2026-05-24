# 03. TO-BE Event Storming

## Cel

Docelowy proces automatyzuje intake i triage reklamacji, ale utrzymuje człowieka w decyzjach niepewnych lub ryzykownych. Główny diagram TO-BE znajduje się w [diagrams/metabot-to-be-main-flow.drawio](../diagrams/metabot-to-be-main-flow.drawio). Obsługa wyjątków jest pokazana osobno w [diagrams/metabot-exception-handling.drawio](../diagrams/metabot-exception-handling.drawio).

## Aktorzy

- Customer: wysyła email reklamacyjny.
- Email Intake Service: obsługuje Microsoft Graph webhook i pobranie emaila.
- Complaint Orchestrator: koordynuje archiwizację, AI, walidacje, JIRA, statusy i routing.
- Attachment Storage Service: zapisuje zdjęcia w Azure Blob Storage.
- AI Extraction and Classification Service: interpretuje email i sugeruje kategorię.
- SAP Integration Adapter: pobiera dane order/batch z SAP.
- JIRA Integration Adapter: tworzy JIRA Complaint i Correction.
- Customer Database Adapter: sprawdza klienta w PostgreSQL.
- AI Service: interpretuje treść emaila i opcjonalnie zdjęcia.
- Service Specialist: obsługuje human review i zatwierdza odpowiedzi.
- Quality Team: pracuje na JIRA Correction.

## External Systems

- Exchange / Microsoft Graph.
- Azure Blob Storage.
- PostgreSQL customer database.
- SAP ERP PP/QM REST API.
- JIRA Cloud.
- Operational Database oraz Monitoring and Reporting Layer dla statusów, audytu i metryk procesu.

## Główne Commands

- Detect New Complaint Email.
- Fetch Complaint Email.
- Archive Attachments.
- Extract Complaint Data.
- Match Customer Data.
- Retrieve SAP Order Data.
- Retrieve SAP Batch Data.
- Apply Complaint Routing Rules.
- Create JIRA Complaint.
- Draft Customer Response.
- Route Complaint For Human Review.
- Create Correction Ticket.
- Update Complaint Metrics.

## Główne Domain Events

- New Complaint Email Detected.
- Complaint Email Fetched.
- Attachments Archived.
- Complaint Data Extracted.
- Defect Category Suggested.
- Customer Data Matched.
- SAP Order Data Retrieved.
- SAP Batch Data Retrieved.
- JIRA Complaint Created.
- Customer Response Drafted.
- Complaint Routed For Human Review.
- Correction Ticket Created.
- Complaint Metrics Updated.

## Policies / Business Rules

- Jeżeli AI confidence jest niskie, sprawa trafia do human review.
- Jeżeli brakuje numeru zamówienia, sprawa trafia do human review.
- Jeżeli klient nie został znaleziony w PostgreSQL, sprawa trafia do human review.
- Jeżeli SAP zwraca niepoprawne dane albo jest niedostępny, sprawa trafia do human review.
- Jeżeli jest podejrzenie duplikatu, sprawa trafia do human review.
- Jeżeli sprawa jest standardowa, kompletna i ma wysokie confidence, system przygotowuje draft odpowiedzi.
- Jeżeli defekt zostanie potwierdzony, tworzony jest JIRA Correction dla Quality Team.
- Jeżeli email zawiera wiele reklamacji, nieczytelne zdjęcia, niejednoznaczny język albo niejasną kategorię, sprawa trafia do human review.

## Separacja odpowiedzialności

- AI interpretation: język, ekstrakcja danych, kategoria, podsumowanie, draft odpowiedzi.
- Deterministic validation: kompletność, klient, SAP, reguły routingu, duplikaty.
- External system integration: Graph, Blob, PostgreSQL, SAP, JIRA.
- Human approval: przypadki niepewne i ryzykowne.
- Metrics/reporting: statusy, czasy, błędy, kategorie, confidence.
