# 05. AI Automation Flow

## Zasada główna

AI jest komponentem interpretującym dane nieustrukturyzowane. Nie jest orkiestratorem procesu, nie wywołuje API i nie decyduje samodzielnie o uznaniu albo odrzuceniu reklamacji.

Za proces odpowiada Complaint Orchestrator. AI Extraction and Classification Service zwraca tylko ustrukturyzowaną sugestię, która potem przechodzi walidację schematu, walidację deterministyczną i reguły routingu.

## Wejście do AI

Do AI trafiają dane przygotowane przez Email Intake Service i Complaint Orchestrator:

- treść emaila,
- metadane emaila, np. nadawca, temat i data otrzymania,
- lista załączników oraz informacja, które zostały zarchiwizowane,
- opcjonalnie obrazy lub ich bezpieczne referencje, jeżeli format jest obsługiwany.

Sekrety, tokeny OAuth2, SAS tokens i dane techniczne integracji nie są przekazywane do AI.

## Oczekiwane wyjście AI

AI zwraca wynik zgodny ze schematem JSON podobnym do [examples/ai-extraction-result.example.json](../examples/ai-extraction-result.example.json):

- `language`,
- `order_id`,
- `customer_name`,
- `customer_identifier`,
- `defect_description`,
- `suggested_category`,
- `confidence`,
- `requires_human_review`,
- `summary`,
- `missing_information`,
- `safety_notes`,
- `validation_warnings`,
- `response_draft`,
- `image_hints`.

Nazwy pól są celowo techniczne i stabilne. Dzięki temu wynik AI można walidować schematem przed użyciem w dalszym procesie.

## Progi confidence

Progi są decyzją procesową, nie prawdą modelu. W MVP powinny być konfigurowalne i monitorowane na podstawie realnych wyników.

| Poziom | Przykładowy próg | Zachowanie |
| --- | --- | --- |
| High confidence | `confidence >= 0.85` | JIRA Complaint może zostać utworzony automatycznie, jeśli dane SAP i klienta są poprawne, a sprawa nie spełnia warunków mandatory review. System przygotowuje draft odpowiedzi. |
| Medium confidence | `0.60 <= confidence < 0.85` | JIRA Complaint może zostać utworzony ze statusem wymagającym weryfikacji. Kategoria i odpowiedź wymagają akceptacji specjalisty. |
| Low confidence | `confidence < 0.60` | Sprawa trafia do human review przed dalszą automatyzacją komunikacji. |

## Mandatory human review

Sprawa zawsze trafia do human review, niezależnie od confidence, jeśli wystąpi co najmniej jeden warunek:

- email nie zawiera numeru zamówienia,
- numer zamówienia jest niepoprawny albo SAP go nie potwierdza,
- klient nie został znaleziony w PostgreSQL,
- wykryto możliwy duplikat,
- SAP, JIRA lub Graph są niedostępne w krytycznym kroku,
- załącznik jest za duży, nieobsługiwany albo nieczytelny,
- email zawiera wiele reklamacji,
- kategoria defektu jest niejasna,
- język wiadomości jest niejednoznaczny,
- draft odpowiedzi mógłby sugerować finalną decyzję reklamacyjną,
- wynik AI nie przechodzi walidacji schematu.

## Guardrails

- Schema validation: wynik AI musi przejść walidację wymaganych pól i typów.
- Deterministic checks: po AI następuje walidacja klienta, zamówienia, partii, duplikatu i kompletności.
- Prompt/version tracking: każdy wynik AI zapisuje `model_version`, `prompt_version` i czas przetwarzania.
- Retry strategy: retry dotyczy błędów technicznych, nie wymusza ponownego losowania decyzji AI bez zapisu poprzedniej próby.
- Audit log: zapisywane są wejścia referencyjne, wynik AI, reguły routingu, błędy integracji i decyzje człowieka.
- No final rejection/approval by AI: AI nie uznaje i nie odrzuca reklamacji.
- Human approval: sprawy niepewne, konfliktowe i ryzykowne wymagają akceptacji specjalisty.

## Decyzja routingowa

Routing nie jest decyzją AI. Complaint Orchestrator stosuje deterministyczne reguły na podstawie:

- wyniku AI,
- walidacji klienta z Customer Database Adapter,
- walidacji order/batch z SAP Integration Adapter,
- statusu archiwizacji załączników,
- detekcji duplikatów,
- dostępności JIRA i Graph,
- polityk human-in-the-loop.

## Przykładowy podział odpowiedzialności

| Obszar | Odpowiedzialny komponent |
| --- | --- |
| Wykrycie nowego emaila | Email Intake Service / Microsoft Graph |
| Archiwizacja zdjęć | Attachment Storage Service / Azure Blob Storage |
| Język emaila | AI Extraction and Classification Service |
| Numer zamówienia z treści | AI Extraction and Classification Service |
| Czy numer zamówienia istnieje | SAP Integration Adapter |
| Czy klient istnieje | Customer Database Adapter |
| Kategoria defektu jako sugestia | AI Extraction and Classification Service |
| Ostateczny routing | Complaint Orchestrator + deterministic rules |
| Utworzenie JIRA | JIRA Integration Adapter |
| Finalna reakcja w sprawie ryzykownej | Service Specialist |
| Audyt i metryki | Operational Database / Monitoring and Reporting Layer |

## Human-in-the-loop

Człowiek przejmuje sprawę, gdy automatyzacja mogłaby udzielić niepełnej, błędnej lub zbyt ryzykownej odpowiedzi. Human review nie jest błędem procesu; jest kontrolowanym mechanizmem bezpieczeństwa.

Specjalista powinien widzieć:

- oryginalny email,
- zarchiwizowane załączniki,
- wynik AI z confidence,
- wynik walidacji SAP i PostgreSQL,
- powód routingu do review,
- draft odpowiedzi,
- historię audytu.

## Draft odpowiedzi

Draft odpowiedzi jest pomocą dla specjalisty albo automatyzacją niskiego ryzyka w standardowych przypadkach. Powinien:

- używać języka wykrytego w emailu,
- potwierdzać przyjęcie reklamacji,
- zawierać numer JIRA Complaint, jeśli jest dostępny,
- prosić o brakujące informacje, jeśli routing wymaga uzupełnienia,
- nie obiecywać uznania reklamacji przed potwierdzeniem defektu.
