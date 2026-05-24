# 08. Q&A projektowe

## Q&A

### What is Metabot’s business model?

Metabot jest producentem metalowych komponentów dla automotive. Proces reklamacyjny chroni relację z klientem, jakość produkcji i koszty korekt. Reklamacja musi być szybko przyjęta, powiązana z zamówieniem lub partią i przekazana do właściwej obsługi w JIRA.

### Who are the users of the process?

Główni użytkownicy to Service Specialists, którzy obsługują reklamacje, oraz Quality Team, który pracuje nad Correction po potwierdzeniu defektu. Klient jest zewnętrznym uczestnikiem procesu, a SAP, JIRA, Exchange, PostgreSQL i Azure Blob Storage są systemami wspierającymi.

### What breaks in the AS-IS process?

Email może trafić do spamu lub zostać przeczytany za późno. Dane są przepisywane do Excela, klasyfikacja jest ręczna i niespójna, SAP i JIRA nie są zintegrowane, a backlog rośnie w peakach. Brakuje też metryk: czasu reakcji, kategorii, źródeł defektów i awarii integracji.

### Why Event Storming here?

Event Storming dobrze pokazuje fakty, decyzje, aktorów, systemy i pain pointy. W tym projekcie pomaga oddzielić proces biznesowy od architektury integracyjnej.

### What is the difference between event, command and UI action?

Domain Event to fakt, który już zaszedł, np. `JIRA Complaint Created`. Command to intencja wykonania akcji, np. `Create JIRA Complaint`. UI action to techniczny gest użytkownika, np. kliknięcie przycisku; sam w sobie nie jest zdarzeniem domenowym.

### Where exactly is AI used?

AI wykrywa język, ekstrahuje `order_id`, klienta i opis defektu, sugeruje kategorię, liczy confidence, streszcza reklamację, przygotowuje draft odpowiedzi i opcjonalnie daje ostrożne wskazówki ze zdjęć. Wynik AI przechodzi schema validation i jest zapisywany z `model_version` oraz `prompt_version`.

### Where exactly is deterministic logic used?

Deterministyczna logika waliduje klienta w PostgreSQL, order/batch w SAP, kompletność danych, duplikaty, rate limit SAP, retry integracji, routing do human review i tworzenie JIRA. AI nie wywołuje API i nie jest źródłem prawdy.

### Why not automate final complaint decisions?

Finalna decyzja reklamacyjna może mieć skutki finansowe, jakościowe i relacyjne. AI może źle zinterpretować email albo zdjęcie, więc w MVP może tylko wspierać intake i triage. Uznanie lub odrzucenie reklamacji zostaje po stronie procesu i człowieka.

### What happens if SAP is unavailable?

SAP Integration Adapter zapisuje integration failure, stosuje retry/backoff i kolejkuje zapytanie z limitem 100 req/min. Sprawa trafia do human review, jeśli brak danych SAP blokuje bezpieczną odpowiedź lub utworzenie kompletnego JIRA.

### What happens if JIRA is unavailable?

Complaint Orchestrator zapisuje sprawę w Operational Database jako `pending_jira_creation`, zapisuje błąd i ponawia próbę przez JIRA Integration Adapter. Sprawa wymaga review operacyjnego, bo bez JIRA zespół może nie widzieć jej w standardowym narzędziu pracy.

### What happens if Microsoft Graph webhook fails?

System używa idempotencji po `source_email_id` i okresowego reconciliation scan skrzynki. Webhook jest głównym triggerem, ale scan zmniejsza ryzyko zgubienia reklamacji.

### What happens if AI confidence is low?

Low confidence zawsze kieruje sprawę do human review. System może nadal zapisać wynik AI jako sugestię, ale nie powinien wysyłać automatycznej odpowiedzi rozstrzygającej ani finalnie klasyfikować reklamacji bez człowieka.

### What happens if the email has no order number?

Sprawa trafia do human review. AI może przygotować draft prośby o uzupełnienie numeru zamówienia, ale SAP validation nie może zostać wykonana bez `order_id`.

### What is the MVP?

MVP obejmuje Exchange intake, archiwizację załączników w Azure Blob Storage, AI extraction and categorization, lookup klienta w PostgreSQL, SAP order/batch lookup, JIRA Complaint creation, human review queue, response draft i podstawowe metryki.

### What would you postpone?

Odłożyłbym custom-trained computer vision model, portal klienta, pełną autonomiczną akceptację/rejekcję, zaawansowane BI, rozbudowane wykrywanie duplikatów i dedykowany UI review. Najpierw trzeba ustabilizować intake, walidację i JIRA.

### What would you simplify with limited budget?

Zostawiłbym email i JIRA, ograniczył AI do ekstrakcji oraz draftu odpowiedzi, użył prostych reguł human review i minimalnej Operational Database dla statusów, idempotencji i metryk. Excel mógłby istnieć krótko jako narzędzie porównawcze, ale nie jako docelowy most procesu.

### How would you measure success?

Mierzyłbym intake delay, email-to-JIRA time, SLA / response time, backlog, manual review rate, AI confidence distribution, category distribution, duplicate rate i integration failure rate. Jeśli SAP zwraca dane produkcyjne, mierzyłbym też reklamacje per batch, operator lub linia.

### What are the biggest risks?

Największe ryzyka to błędna interpretacja AI, niekompletne emaile, awarie SAP/JIRA/Graph, duplikaty, nieczytelne zdjęcia, dane osobowe w załącznikach i zbyt szybka próba automatyzacji decyzji reklamacyjnych.

### What would you monitor in production?

Monitorowałbym backlog, email-to-JIRA time, Graph webhook failures, reconciliation scan findings, SAP queue delay, SAP 429/rate limit events, JIRA creation failures, attachment upload failures, AI schema validation failures, confidence distribution, manual review reasons i SLA breaches.

### Is the MVP realistic for 600 complaints/month and peaks up to 2000/month?

Tak. Średni wolumen 2000 miesięcznie jest niewysoki dla tych integracji, ale trzeba projektować pod spiętrzenia, nie średnią. Dlatego wystarczą idempotencja, kolejka w Operational Database, retry/backoff, throttling SAP i monitoring backlogu zamiast ciężkiej architektury high-scale.

### How can you defend the architecture diagram?

Diagram pokazuje separację odpowiedzialności: intake, storage, AI, adaptery integracyjne, orchestrator, operational database, monitoring i human review. To jest ważniejsze niż liczba pudełek, bo każdy komponent ma jasny powód i nie zastępuje dostępnych systemów SAP, JIRA ani Exchange.
