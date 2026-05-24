# 04. Specyfikacja rozwiązania

## 1. Executive Summary

Proponowane rozwiązanie to AI-assisted complaint intake and triage dla reklamacji przychodzących emailem. Automatyzacja wykrywa nowe wiadomości w Exchange, pobiera treść i załączniki, archiwizuje zdjęcia w Azure Blob Storage, używa AI do ekstrakcji danych i sugestii kategorii, waliduje klienta w PostgreSQL oraz zamówienie lub partię w SAP, a następnie tworzy JIRA Complaint.

System nie podejmuje finalnych decyzji wysokiego ryzyka. Przypadki niepewne, niekompletne i awaryjne są kierowane do human review.

## 2. Business Context

Obecny proces jest emailowy i ręczny. Specjaliści kopiują dane do Excela, klasyfikują defekty, sprawdzają SAP i zakładają JIRA. Powoduje to opóźnienia, backlog, niespójną klasyfikację i brak metryk.

## 3. Goals

- Skrócić czas od przyjścia emaila do utworzenia JIRA Complaint.
- Ograniczyć ręczne przepisywanie danych.
- Ujednolicić klasyfikację defektów przez sugestię AI i reguły walidacyjne.
- Zapewnić archiwum zdjęć defektów.
- Wprowadzić podstawowe metryki operacyjne.
- Zachować human-in-the-loop dla przypadków ryzykownych.

## 3.1 Wolumen i skalowanie MVP

MVP jest realistyczne dla około 600 reklamacji miesięcznie i sezonowych wzrostów do około 2000 reklamacji miesięcznie. Przy 2000 spraw miesięcznie średni dzienny wolumen pozostaje niski dla Microsoft Graph, JIRA i SAP, ale problemem mogą być krótkie spiętrzenia po weekendzie, awarii integracji albo kampanii jakościowej klienta.

Z tego powodu rozwiązanie nie wymaga rozbudowanej architektury wysokiej skali, ale wymaga:

- idempotencji po `source_email_id`,
- kolejki pracy w Operational Database,
- retry/backoff dla integracji,
- throttlingu SAP do limitu 100 req/min,
- monitoringu backlogu i email-to-JIRA time,
- human review queue dla spraw, których nie można bezpiecznie obsłużyć automatycznie.

## 4. Non-Goals

- Pełna autonomiczna akceptacja lub odrzucanie reklamacji.
- Custom-trained computer vision model.
- Zastąpienie SAP.
- Zastąpienie JIRA.
- Budowa złożonego portalu klienta.
- Zaawansowana platforma BI.

## 5. Proposed Architecture

Architektura jest podzielona na jawne komponenty, które łatwo obronić technicznie:

- Email Intake Service: Microsoft Graph webhook i pobieranie emaili.
- Attachment Storage Service: zapis zdjęć i metadanych w Azure Blob Storage.
- AI Extraction and Classification Service: ekstrakcja danych, język, kategoria, confidence, missing information i draft odpowiedzi.
- Customer Database Adapter: read-only lookup klienta w PostgreSQL.
- SAP Integration Adapter: walidacja order/batch z kontrolą rate limit 100 req/min.
- JIRA Integration Adapter: tworzenie i aktualizacja JIRA Complaint oraz Correction.
- Complaint Orchestrator: koordynacja procesu, idempotencja, retry, routing i statusy.
- Operational Database: statusy procesu, wynik AI, referencje integracji, audyt techniczny i metryki.
- Monitoring and Reporting Layer: praktyczne metryki operacyjne.
- Human Review Workflow: obsługa spraw niepewnych, niekompletnych i awaryjnych.

Diagram architektury znajduje się w [diagrams/metabot-solution-architecture.drawio](../diagrams/metabot-solution-architecture.drawio). Osobny diagram struktury JIRA i metryk znajduje się w [diagrams/metabot-jira-ticket-and-metrics.drawio](../diagrams/metabot-jira-ticket-and-metrics.drawio).

## 6. AI Usage

AI jest używane do:

- wykrycia języka wiadomości,
- ekstrakcji numeru zamówienia, klienta, opisu defektu i brakujących informacji,
- sugestii kategorii defektu,
- policzenia confidence,
- podsumowania reklamacji,
- przygotowania draftu odpowiedzi,
- opcjonalnych wskazówek z analizy zdjęć defektu.

AI nie jest używane do:

- wywoływania API,
- walidacji źródeł prawdy,
- obsługi dostępności SAP/JIRA/Graph,
- obsługi rate limit,
- finalnych decyzji wysokiego ryzyka,
- audytu,
- decyzji bezpieczeństwa.

Wynik AI musi przejść schema validation. Zapisywane są `model_version`, `prompt_version`, confidence, powody human review i ostrzeżenia walidacyjne.

## 7. Integrations

- Microsoft Graph: webhook dla nowych emaili, pobranie treści i załączników.
- Azure Blob Storage: archiwum zdjęć z kontrolowanymi SAS tokens.
- PostgreSQL customer database: read-only customer lookup przez Customer Database Adapter.
- SAP ERP PP/QM REST API: order i batch lookup przez SAP Integration Adapter, limit 100 req/min.
- JIRA Cloud: tworzenie `Complaint` i `Correction` w projekcie `REK` przez JIRA Integration Adapter.

## 8. Main Flow

1. Nowy email przychodzi do Exchange.
2. Microsoft Graph webhook uruchamia automatyzację.
3. Email Intake Service pobiera body i załączniki.
4. Attachment Storage Service zapisuje zdjęcia defektów w Azure Blob Storage.
5. AI Extraction and Classification Service zwraca strukturalny wynik ekstrakcji.
6. Customer Database Adapter sprawdza klienta w PostgreSQL.
7. SAP Integration Adapter sprawdza zamówienie i partię w SAP.
8. Complaint Orchestrator stosuje deterministyczne reguły łączące wynik AI, dane klienta i SAP.
9. JIRA Integration Adapter tworzy JIRA Complaint.
10. Dla standardowych spraw wysokiej pewności powstaje draft odpowiedzi do klienta.
11. Sprawy niskiej pewności, niekompletne lub awaryjne trafiają do human review.
12. Po potwierdzeniu defektu tworzony jest JIRA Correction.
13. Statusy i metryki są zapisywane do raportowania.

## 9. Exception Flows

- Email bez numeru zamówienia: human review i draft prośby o uzupełnienie.
- Niepoprawny numer zamówienia: human review po walidacji SAP.
- Duplicate complaint: human review przed połączeniem lub zamknięciem sprawy.
- SAP unavailable: retry/backoff, kolejka zapytań i human review, jeśli brak danych blokuje obsługę.
- JIRA unavailable: zapis sprawy jako pending JIRA creation, retry i review operacyjny.
- Graph webhook failure: idempotencja po `source_email_id` i okresowy reconciliation scan skrzynki.
- Attachment too large or unsupported: archiwizacja statusu błędu i human review.
- Low AI confidence: human review.
- Multiple complaints in one email: human review.
- Customer not found: human review, bez automatycznej odpowiedzi rozstrzygającej.
- Unclear defect category: human review.
- Image unreadable: human review.
- Language ambiguity: human review albo draft neutralnej prośby o doprecyzowanie.

## 10. Data Model

### Complaint

- `id`
- `source_email_id`
- `received_at`
- `status`
- `language`
- `customer_id`
- `order_reference_id`
- `batch_reference_id`
- `defect_description`
- `summary`
- `created_jira_issue_id`

### Customer

- `id`
- `name`
- `email`
- `matched_from_postgresql`
- `match_status`

### OrderReference

- `order_id`
- `sap_status`
- `product`
- `production_line`
- `operator`
- `retrieved_at`

### BatchReference

- `batch_id`
- `sap_status`
- `production_date`
- `retrieved_at`

### Attachment

- `id`
- `complaint_id`
- `file_name`
- `content_type`
- `blob_uri`
- `archived_at`
- `supported`

### AIExtractionResult

- `complaint_id`
- `language`
- `schema_version`
- `model_version`
- `prompt_version`
- `order_id`
- `customer_name`
- `customer_identifier`
- `defect_description`
- `suggested_category`
- `confidence`
- `requires_human_review`
- `human_review_reasons`
- `missing_information`
- `summary`
- `safety_notes`
- `validation_warnings`
- `response_draft`

### ClassificationDecision

- `complaint_id`
- `final_category`
- `decision_source`
- `routing`
- `reason`
- `requires_human_review`

### JiraIssueReference

- `complaint_id`
- `project_key`
- `issue_key`
- `issue_type`
- `created_at`

### AuditLogEntry

- `id`
- `complaint_id`
- `timestamp`
- `actor`
- `action`
- `details`

## 11. Security and Compliance

- OAuth2 dla Microsoft Graph i JIRA.
- Least privilege dla dostępu do skrzynki, JIRA i Blob Storage.
- Brak sekretów w repozytorium.
- SAS tokens generowane krótkożyjące i używane tylko do wymaganych operacji.
- Audit logs dla decyzji, walidacji, integracji i zmian statusu.
- GDPR/data minimization: przechowywać tylko dane potrzebne do obsługi reklamacji.
- Retention policy dla emaili, załączników, logów i wyników AI.
- Human review dla spraw wrażliwych, niepewnych lub ryzykownych.

## 12. Monitoring and Metrics

Metryki odpowiadają bezpośrednio na problemy zgłoszone w briefie:

| Problem biznesowy | Metryka |
| --- | --- |
| Email może trafić do spamu lub zostać przeczytany za późno | Intake delay, liczba wiadomości wykrytych przez reconciliation scan |
| Backlog rośnie w peakach | Backlog, liczba spraw per dzień/tydzień, SLA / response time |
| Klasyfikacja jest niespójna | Category distribution, manual category override rate |
| SAP i JIRA nie są zintegrowane | Email-to-JIRA time, integration failure rate |
| Excel jest ręcznym mostem | Liczba spraw obsłużonych poza automatyzacją, pending JIRA creation |
| Brakuje metryk jakościowych | Reklamacje per linia, batch lub operator, jeżeli SAP zwraca te dane |
| AI może być niepewne | AI confidence distribution, manual review rate |
| Duplikaty obciążają zespół | Duplicate rate |

## 13. MVP Scope

MVP obejmuje:

- automated intake z Exchange,
- archiwizację załączników,
- AI extraction and categorization,
- lookup klienta w PostgreSQL,
- SAP order/batch lookup,
- tworzenie JIRA Complaint,
- human review queue,
- draft odpowiedzi,
- podstawowe metryki.

MVP nie obejmuje:

- pełnej autonomicznej akceptacji/rejekcji,
- custom-trained computer vision model,
- zastąpienia SAP,
- zastąpienia JIRA,
- złożonego portalu klienta,
- zaawansowanej platformy BI.

## 14. Future Improvements

- Lepsze wykrywanie duplikatów na podstawie historii reklamacji.
- Rozbudowane dashboardy, jeśli podstawowe metryki okażą się wartościowe.
- Usprawnienia klasyfikacji kategorii po zebraniu danych z produkcji.
- Szablony odpowiedzi per język i kategoria defektu.
- Ostrożne rozszerzenie analizy zdjęć, bez zastępowania decyzji jakościowych.

## 15. Trade-offs

- LLM vs rules: LLM obsługuje język naturalny i zdjęcia jako wskazówki, a reguły deterministyczne obsługują walidację i routing. To ogranicza ryzyko halucynacji.
- Human-in-the-loop vs full automation: człowiek spowalnia część spraw, ale chroni przed błędami finansowymi, jakościowymi i komunikacyjnymi.
- JIRA review queue vs custom UI: JIRA jest dostępna i szybka do użycia, ale może być mniej wygodna niż dedykowany panel review.
- Image analysis in MVP vs later: MVP może używać tylko ostrożnych image hints; pełna analiza jakościowa zdjęć zostaje poza zakresem.
- Excel coexistence during transition vs immediate removal: przez krótki okres Excel może służyć do porównania metryk, ale docelowo nie powinien być mostem procesowym.
- Operational Database vs relying only on JIRA: baza operacyjna daje idempotencję, audyt i metryki techniczne, ale dodaje komponent utrzymaniowy.
- Speed of delivery vs auditability: szybkie MVP jest możliwe, ale nie kosztem śledzenia decyzji AI, walidacji i integracji.
