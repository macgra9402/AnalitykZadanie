# Specyfikacja rozwiązania — Metabot AI Complaint Automation

## Zakres dokumentu

Dokument przedstawia koncepcję procesowo-architektoniczną rozwiązania na poziomie analitycznym. Celem jest pokazanie sposobu uporządkowania procesu, roli automatyzacji AI, integracji systemów oraz kluczowych decyzji projektowych. Nie jest to kompletna specyfikacja wdrożeniowa ani pełny backlog implementacyjny. Szczegóły takie jak produkcyjny model danych, mapowanie integracji, szczegółowe reguły walidacji, backlog prac, konfiguracja środowisk oraz finalne parametry techniczne powinny zostać doprecyzowane w osobnym etapie discovery/warsztatowym.

Koncepcja skupia się nie tylko na wskazaniu miejsc użycia modelu AI, ale przede wszystkim na uporządkowaniu procesu, odpowiedzialności, danych, wyjątków i punktów kontroli człowieka.

Zakres dokumentu obejmuje analizę procesu **AS-IS**, koncepcję procesu **TO-BE**, wysokopoziomową architekturę, role integracji, granice użycia AI, decyzje projektowe, ryzyka, metryki, zakres **MVP** i trade-offy. Dokument celowo nie zawiera kompletnego backlogu wdrożeniowego, pełnego produkcyjnego modelu danych, szczegółowych mapowań pól, finalnych instrukcji dla modelu AI, produkcyjnej konfiguracji ani kodu implementacyjnego.

## 1. Podsumowanie

Metabot / Metalpol ma już narzędzia do obsługi reklamacji: **Microsoft 365 / Exchange**, Excel, **JIRA** i **SAP ERP PP/QM**. Problemem nie jest więc sam brak systemów, lecz brak spójnej wizji procesu, jasnego podziału odpowiedzialności i mierzalnej **odpowiedzialności za proces**. W modelu **AS-IS** prawie cały przepływ zależy od specjalisty serwisu, który czyta wiadomość, kopiuje dane, klasyfikuje defekt, tworzy JIRA i sprawdza SAP. Taka zależność od jednej roli tworzy niską **powtarzalność procesu**, niespójną klasyfikację i operacyjne wąskie gardła.

Największy obszar usprawnienia to konsolidacja danych, automatyzacja powtarzalnych czynności i przeniesienie specjalisty z monotonnej pracy operacyjnej do oceny, decyzji oraz kontroli jakości. Celem nie jest zastąpienie specjalistów, lecz odciążenie ich z ręcznego przepisywania i szukania informacji. System przygotowuje sprawę, porządkuje dane i wskazuje ryzyka, a specjalista może skupić się na pracy wymagającej doświadczenia.

Rozwiązanie opiera się na bezpiecznym podziale odpowiedzialności: **AI porządkuje i sugeruje**, walidacja techniczna sprawdza format danych, reguły deterministyczne wybierają ścieżkę procesu, a człowiek podejmuje decyzje w sprawach ryzykownych. Dokument pokazuje rekomendowany kierunek rozwoju automatyzacji i infrastruktury. Szczegółowy budżet, zakres **MVP** i kolejność wdrożenia powinny zostać doprecyzowane podczas trzech konsultacji warsztatowych.

## 2. Kontekst biznesowy

Metabot / Metalpol to producent metalowych komponentów dla branży automotive. Firma ma około 180 pracowników, 3 hale produkcyjne i dział obsługi posprzedażowej odpowiedzialny za reklamacje klientów.

Typowy wolumen to około 600 reklamacji miesięcznie, z sezonowymi wzrostami do około 2000 reklamacji miesięcznie. Wiadomości reklamacyjne przychodzą przez skrzynkę e-mail, zwykle w języku polskim albo angielskim. Wiadomość zawiera opis problemu, numer zamówienia i 1-3 zdjęcia defektu.

Obecny proces **AS-IS**:

1. Klient wysyła wiadomość reklamacyjną.
2. Specjalista serwisu czyta wiadomość ręcznie.
3. Dane są kopiowane do pliku Excel.
4. Specjalista przypisuje kategorię defektu.
5. Specjalista tworzy **JIRA Complaint** w projekcie `REK`.
6. Specjalista sprawdza zamówienie i partię w **SAP ERP PP/QM**.
7. Specjalista odpowiada klientowi.
8. Jeżeli defekt zostanie potwierdzony, tworzony jest **JIRA Correction** dla działu jakości.

## 3. Problemy procesu AS-IS

Obecny proces ma kilka praktycznych problemów operacyjnych:

- około 40% wiadomości trafia do spamu albo jest czytane z opóźnieniem,
- backlog wynosi 2-3 dni,
- jeden specjalista obsługuje około 30 reklamacji dziennie, a w okresach szczytu do 80 dziennie,
- dane są kopiowane ręcznie do Excela,
- klasyfikacja defektów jest niespójna między osobami,
- brakuje wiarygodnych **metryk** procesu,
- brakuje trendów kategorii reklamacji,
- brakuje widoczności problemów po linii produkcyjnej,
- SAP i JIRA nie są zintegrowane,
- specjalista ręcznie łączy Exchange, Excel, JIRA i SAP.

**Człowiek jest dziś ręcznym integratorem między systemami.**

Skutkiem jest opóźniona reakcja do klienta, brak jednego źródła statusu i niska powtarzalność klasyfikacji.

## 4. Docelowa koncepcja rozwiązania

Docelowy proces **TO-BE** porządkuje odpowiedzialności między systemami. Każdy komponent ma konkretną rolę: jedne pobierają dane, inne je przechowują, AI interpretuje treść, systemy źródłowe walidują fakty, a **Complaint Orchestrator** kontroluje ścieżkę obsługi.

Docelowy przepływ:

```text
Wiadomość klienta
→ Microsoft 365 / Exchange
→ Microsoft Graph Webhook
→ Usługa przyjęcia wiadomości
→ PostgreSQL Complaint Registry
→ Azure Blob Storage
→ AI Extraction & Classification Service
→ Pydantic / walidacja schematu
→ Customer DB Adapter
→ SAP Integration Service
→ Complaint Orchestrator
→ JIRA Complaint
→ ręczna weryfikacja
→ Customer Response / JIRA Correction
→ Metrics Dashboard
```

Rola komponentów:

| Komponent | Co robi | Wejście | Wynik | Charakter logiki | Co zdejmuje ze specjalisty |
| --- | --- | --- | --- | --- | --- |
| **Microsoft 365 / Exchange** | Przyjmuje wiadomości reklamacyjne od klientów. | Wiadomość klienta z opisem, numerem zamówienia i zdjęciami. | Wiadomość dostępna w skrzynce reklamacyjnej. | System zewnętrzny. | Pozostawia klientowi znany kanał kontaktu. |
| **Microsoft Graph Webhook** | Informuje system, że pojawiła się nowa wiadomość. | Zdarzenie o nowej wiadomości. | Sygnał uruchamiający pobranie wiadomości. | Integracja deterministyczna. | Zmniejsza ryzyko, że wiadomość zostanie zauważona z opóźnieniem. |
| **Microsoft Graph API** | Pobiera właściwą treść wiadomości i załączniki. | Identyfikator wiadomości z webhooka. | Treść wiadomości, metadane i pliki. | Integracja deterministyczna. | Eliminuje ręczne otwieranie wiadomości jako pierwszy krok procesu. |
| **Usługa przyjęcia wiadomości** | Obsługuje pobranie i wstępną rejestrację wiadomości. | Dane z Microsoft Graph API. | Rekord zgłoszenia i status przetwarzania. | Usługa integracyjna. | Zdejmuje ręczne przenoszenie danych z poczty do rejestru. |
| **PostgreSQL Complaint Registry** | Jest operacyjnym rejestrem reklamacji i zastępuje Excel. | Metadane wiadomości, statusy, wyniki walidacji, referencje JIRA i Blob. | Centralny status procesu. | Przechowywanie danych i audyt. | Zastępuje ręczne śledzenie reklamacji w Excelu. |
| **Azure Blob Storage** | Przechowuje zdjęcia i załączniki. | Pliki pobrane z wiadomości. | Bezpieczne linki i metadane plików. | Przechowywanie plików. | Zdejmuje ręczne archiwizowanie zdjęć i szukanie załączników w poczcie. |
| **AI Extraction & Classification Service** | Porządkuje treść wiadomości: wykrywa język, wyciąga dane, streszcza opis i sugeruje kategorię. | Treść wiadomości, metadane i informacja o załącznikach. | Strukturalny wynik, **poziom pewności AI**, braki danych i robocza odpowiedź. | AI, ale bez decyzji finalnej. | Zdejmuje ręczne czytanie od zera i pierwsze porządkowanie treści. |
| **Pydantic / walidacja schematu** | Sprawdza, czy wynik AI ma poprawną strukturę. | Wynik AI w formacie JSON. | Poprawny wynik albo błąd walidacji. | Walidacja techniczna. | Zapobiega pracy na niekompletnych lub źle sformatowanych danych. |
| **Customer DB Adapter** | Sprawdza klienta w PostgreSQL Customer DB tylko do odczytu. | Dane nadawcy, domena, nazwa firmy lub identyfikator klienta. | Wynik dopasowania klienta albo powód braku dopasowania. | Integracja deterministyczna. | Ogranicza ręczne sprawdzanie klienta i ryzyko pomyłki. |
| **SAP Integration Service** | Waliduje zamówienie i partię w **SAP ERP PP/QM**. | Identyfikator zamówienia, identyfikator partii, dane klienta. | Status zamówienia, status partii i kontekst produkcyjny, jeśli jest dostępny. | Integracja deterministyczna, tylko do odczytu w **MVP**. | Zmniejsza ręczne przełączanie się do SAP i kopiowanie wyników. |
| **Complaint Orchestrator** | Kontroluje ścieżkę obsługi i zapisuje status procesu. | Wynik wiadomości, AI, walidacji schematu, Customer DB, SAP, Blob Storage i historia reklamacji. | Decyzja procesowa: ścieżka standardowa, ręczna weryfikacja, kolejka ponowień albo kolejka błędów trwałych. | Deterministyczna, audytowalna logika procesu. | Pilnuje kompletności minimum danych i kieruje sprawę do właściwej ścieżki. |
| **JIRA Integration Service** | Tworzy przygotowany **JIRA Complaint** w projekcie `REK`. | Dane z rejestru, wynik AI, walidacja SAP, linki do załączników. | JIRA Complaint gotowy do pracy specjalisty. | Integracja deterministyczna. | Eliminuje ręczne tworzenie zgłoszenia od zera. |
| **JIRA Cloud / REK** | Pozostaje miejscem pracy zespołu serwisowego i jakości. | Przygotowany JIRA Complaint i ewentualny JIRA Correction. | Widoczna sprawa operacyjna i zadania dla działu jakości. | System pracy człowieka. | Pozwala pracować w znanym narzędziu zamiast w nowym panelu. |
| **Metrics Dashboard** | Pokazuje wolumen, backlog, czas obsługi, jakość AI i błędy integracji. | Statusy i zdarzenia z Complaint Registry, JIRA i integracji. | **Metryki** dla zarządzania i jakości. | Raportowanie. | Zdejmuje ręczne liczenie reklamacji i przygotowywanie raportów. |

**Complaint Orchestrator** nie jest "magią AI" ani autonomicznym mechanizmem decyzyjnym. To kontrolowany komponent wyboru ścieżki procesu. Może używać wyników AI, ale jego logika powinna być deterministyczna, audytowalna i możliwa do wyjaśnienia.

Logika **Complaint Orchestrator** obejmuje:

- zebranie wyników z wiadomości, AI, walidacji schematu, Customer DB, SAP, Blob Storage i historii reklamacji,
- sprawdzenie kompletności danych,
- sprawdzenie **poziomu pewności AI**,
- sprawdzenie wyniku walidacji SAP,
- sprawdzenie zgodności klienta i zamówienia,
- wykrycie możliwego duplikatu,
- wybór ścieżki: standardowa obsługa, ręczna weryfikacja, kolejka ponowień albo kolejka błędów trwałych,
- zapis statusu procesu w **PostgreSQL Complaint Registry**,
- zabezpieczenie przed podwójnym przetwarzaniem tej samej wiadomości,
- utworzenie JIRA dopiero wtedy, gdy dostępne są minimalne dane wymagane dla poprawnego zgłoszenia.

W skrócie: **AI wspiera interpretację**, **Pydantic sprawdza strukturę danych**, **Complaint Orchestrator stosuje jawne reguły**, a **człowiek kontroluje decyzje ryzykowne**.

## 5. Proponowana automatyzacja AI

AI jest używane tam, gdzie dane są nieustrukturyzowane i językowe. Nie zastępuje systemów źródłowych ani decyzji jakościowych.

**AI nie jest decydentem procesu. AI jest warstwą interpretacji i przygotowania danych.**

AI powinno:

- wykryć język PL/EN,
- wyciągnąć numer zamówienia,
- wyciągnąć tożsamość klienta, jeżeli jest dostępna,
- wyciągnąć opis defektu,
- streścić reklamację,
- zasugerować kategorię defektu:
  - visual,
  - dimension,
  - material,
  - logistics,
- policzyć **poziom pewności AI**,
- wykryć brakujące informacje,
- oznaczyć sprawę do ręcznej weryfikacji,
- przygotować roboczą odpowiedź do klienta,
- opcjonalnie wspierać interpretację zdjęć w kolejnych etapach.

AI nie powinno:

- podejmować finalnych decyzji reklamacyjnych,
- samodzielnie uznawać albo odrzucać reklamacji,
- zapisywać danych do SAP,
- zastępować walidacji SAP,
- wykonywać walidacji deterministycznej,
- tworzyć niekontrolowanych akcji w JIRA,
- omijać ręcznej weryfikacji w przypadkach ryzykownych.

**AI porządkuje i sugeruje. Walidacja techniczna sprawdza format danych. Reguły deterministyczne wybierają ścieżkę procesu. Człowiek podejmuje decyzje w sprawach ryzykownych.**

### 5.1 Pydantic i walidacja schematu

AI może zwrócić wynik niepełny, niespójny albo źle sformatowany. Dlatego wynik AI powinien przejść walidację schematu przed użyciem w dalszym procesie.

**Pydantic działa jak kontroler jakości danych po AI.** Nie ocenia, czy reklamacja jest zasadna. Sprawdza tylko, czy wynik AI ma strukturę, z którą system może bezpiecznie pracować.

Przykładowy wynik AI powinien zawierać pola:

- `order_id`,
- `defect_description`,
- `suggested_category`,
- `confidence`,
- `missing_information`.

Pydantic powinien sprawdzić:

- czy wymagane pola są obecne,
- czy nazwy pól są zgodne ze schematem,
- czy `suggested_category` jest jedną z wartości: `visual`, `dimension`, `material`, `logistics`,
- czy `confidence` jest liczbą od 0 do 1,
- czy struktura list i obiektów jest poprawna,
- czy wynik nadaje się technicznie do dalszego przetwarzania.

Jeżeli wynik AI nie przechodzi walidacji schematu, sprawa nie powinna iść ścieżką standardową. Powinna trafić do ręcznej weryfikacji albo do obsługi błędu, zależnie od przyczyny.

### 5.2 Reguły deterministyczne i wybór ścieżki procesu

Wybór ścieżki procesu nie jest swobodną decyzją AI. To zestaw jawnych reguł typu "jeżeli X, to Y". Te same dane wejściowe powinny zawsze dawać ten sam wynik decyzji procesowej.

Przykładowe reguły:

- jeżeli **poziom pewności AI** jest poniżej progu, sprawa trafia do ręcznej weryfikacji,
- jeżeli brakuje `order_id`, sprawa trafia do ręcznej weryfikacji,
- jeżeli SAP nie znajduje zamówienia, sprawa trafia do ręcznej weryfikacji,
- jeżeli SAP jest tymczasowo niedostępny, sprawa trafia do kolejki ponowień,
- jeżeli limit ponowień zostanie przekroczony, sprawa trafia do kolejki błędów trwałych,
- jeżeli dane są kompletne, walidacja SAP jest pozytywna, a poziom pewności AI jest wysoki, system tworzy przygotowany **JIRA Complaint** do weryfikacji.

AI może dostarczyć sygnał, ale ścieżkę procesu wybierają jawne reguły.

### 5.3 Doskonalenie modelu AI i monitoring poziomu pewności

Komponent AI nie powinien być traktowany jako jednorazowa, statyczna funkcja. Powinien być monitorowany i ulepszany w kontrolowany sposób.

**Poziom pewności AI nie jest decyzją. To sygnał ryzyka.** Wysoki wynik oznacza, że system może mieć większe zaufanie do sugestii AI, ale nie zastępuje walidacji SAP ani decyzji człowieka. Niski wynik oznacza, że sprawa powinna trafić do ręcznej weryfikacji. W implementacji technicznej ta miara może być zapisywana jako `confidence`.

System powinien zapisywać:

- kategorię zasugerowaną przez AI,
- poziom pewności AI,
- finalną kategorię zatwierdzoną przez człowieka,
- powód korekty człowieka,
- flagi brakujących danych,
- przypadki błędnej sugestii lub niepewności,
- sprawy skierowane do ręcznej weryfikacji,
- przypadki, w których walidacja Pydantic nie przeszła.

Bezpieczna pętla doskonalenia:

1. AI sugeruje kategorię i poziom pewności.
2. Człowiek potwierdza albo koryguje wynik.
3. Korekta jest zapisywana w rejestrze.
4. Metryki pokazują, gdzie AI działa dobrze lub słabo.
5. Zespół poprawia instrukcje dla modelu, schemat danych, przykłady, progi pewności albo wersję modelu.
6. Zmiany są testowane na danych historycznych lub ewaluacyjnych przed użyciem produkcyjnym.

Przykłady użycia:

- wysoki poziom pewności AI + pozytywna walidacja SAP → standardowa ścieżka weryfikacji,
- niski poziom pewności AI → ręczna weryfikacja,
- częsty niski poziom pewności AI dla jednej kategorii → poprawa instrukcji dla modelu, przykładów lub definicji kategorii,
- częste korekty z `visual` na `material` → przegląd zachowania modelu i definicji kategorii.

Celem nie jest ślepe zaufanie modelowi, lecz jego **kontrolowane doskonalenie**. Każda korekta człowieka jest informacją zwrotną, która pozwala ulepszać instrukcje dla modelu, schemat danych, progi pewności albo docelowo wersję modelu. AI nie uczy się samodzielnie w produkcji bez kontroli; zmiany wymagają ewaluacji, wersjonowania i zatwierdzenia.

## 6. TO-BE Process Flow

### 6.1 Przyjęcie wiadomości

Exchange odbiera wiadomość reklamacyjną. **Microsoft Graph Webhook** wykrywa nową wiadomość i uruchamia usługę przyjęcia wiadomości. Webhook jest tylko sygnałem startowym; właściwa treść wiadomości i załączniki są pobierane przez **Microsoft Graph API**. Dostęp do Microsoft 365 jest zabezpieczony przez **OAuth2** i ograniczony zasadą minimalnych uprawnień.

### 6.2 Rejestracja zgłoszenia i zapis załączników

Nowa reklamacja jest rejestrowana w **PostgreSQL Complaint Registry**. Rejestr zapisuje `message_id`, status, daty, metadane, identyfikatory i wyniki przetwarzania. Zdjęcia i załączniki są przechowywane w **Azure Blob Storage**, a PostgreSQL przechowuje tylko metadane i linki. Excel zostaje zastąpiony jako operacyjny rejestr procesu, ale może pozostać jako format eksportu raportów.

### 6.3 Ekstrakcja, klasyfikacja i walidacja schematu

**AI Extraction & Classification Service** analizuje treść wiadomości i zwraca strukturalny wynik: język, numer zamówienia, dane klienta, opis defektu, podsumowanie, sugerowaną kategorię, **poziom pewności AI** i brakujące informacje. Następnie Pydantic sprawdza, czy wynik AI ma poprawny format i wymagane pola.

Niski poziom pewności AI, niejednoznaczny opis, brak kluczowych danych albo błąd walidacji schematu kierują sprawę do ręcznej weryfikacji.

### 6.4 Dopasowanie klienta

**Customer DB Adapter** sprawdza klienta w PostgreSQL Customer DB tylko do odczytu. Dopasowanie może używać adresu e-mail, domeny, nazwy firmy albo identyfikatora klienta, jeśli występuje w wiadomości. Wynik dopasowania jest zapisywany w Complaint Registry. Customer DB nie jest modyfikowana.

### 6.5 Walidacja SAP

**SAP ERP PP/QM** jest używany tylko przez dostępne endpointy:

- `GET /api/v1/orders/{id}`
- `GET /api/v1/batches/{id}`

SAP jest używany tylko do odczytu w **MVP**. Rozwiązanie nie zapisuje decyzji ani danych do SAP.

SAP służy do sprawdzenia:

- czy zamówienie istnieje,
- czy zamówienie jest powiązane z klientem,
- jaka partia jest powiązana ze sprawą,
- jaki kontekst produkcyjny lub jakościowy jest dostępny,
- czy dla partii występują powtarzalne problemy, jeżeli SAP zwraca takie dane.

Limit SAP wynosi 100 req/min, co przy zakładanym wolumenie 600-2000 reklamacji miesięcznie jest wystarczającą przepustowością dla tego typu rozwiązania. Integracja powinna jednak respektować limity SAP i obsługiwać ponowienia w kontrolowany sposób.

### 6.6 Orkiestracja reklamacji

**Complaint Orchestrator** łączy:

- dane z wiadomości,
- wynik AI,
- wynik walidacji schematu,
- wynik dopasowania klienta,
- walidację SAP,
- linki do załączników,
- historię reklamacji,
- flagi ryzyka.

Na tej podstawie wybiera ścieżkę obsługi:

- ścieżka standardowa,
- ręczna weryfikacja,
- kolejka ponowień,
- kolejka błędów trwałych.

Wybór ścieżki procesu nie jest decyzją AI. Jest wynikiem reguł deterministycznych używających danych z AI, walidacji schematu, SAP, Customer DB, Blob Storage i JIRA.

### 6.7 JIRA i ręczna weryfikacja

**JIRA Integration Service** tworzy **JIRA Complaint** w projekcie `REK`. Ticket zawiera:

- dane klienta,
- order ID,
- identyfikator partii,
- oryginalny opis,
- podsumowanie AI,
- sugerowaną kategorię,
- poziom pewności AI,
- wynik walidacji SAP,
- linki do załączników,
- `human_review_required`,
- `human_review_reason`,
- sugerowany następny krok,
- roboczą odpowiedź.

JIRA pozostaje miejscem pracy człowieka. Serwisant nie traci czasu na szukanie i przepisywanie danych. Otrzymuje przygotowaną sprawę i może skupić się na ocenie, decyzji oraz kontakcie z klientem.

### 6.8 Korekcja, odpowiedź i metryki

Jeżeli defekt zostanie potwierdzony, tworzony jest **JIRA Correction** dla działu jakości. Odpowiedź do klienta jest przygotowana jako robocza wersja i wysyłana po weryfikacji, jeśli sprawa wymaga kontroli człowieka. Po każdym istotnym kroku aktualizowane są statusy i **metryki** w **PostgreSQL Complaint Registry**.

## 7. Integracje

| System / Integration | Rola | Dane | Uwagi / ograniczenia |
| --- | --- | --- | --- |
| **Microsoft 365 / Exchange** | Skrzynka reklamacyjna | Wiadomość, nadawca, temat, treść, załączniki | Kanał wejściowy, bez zmiany po stronie klienta. |
| **Microsoft Graph Webhook** | Wykrycie nowej wiadomości | Zdarzenie o nowej wiadomości | Webhook tylko uruchamia proces; nie przetwarza treści. |
| **Microsoft Graph API** | Pobranie treści | Treść wiadomości, metadane, załączniki | Treść pobierana po webhooku; OAuth2. |
| **OAuth2** | Bezpieczeństwo dostępu | Tokeny dostępu | Minimalne uprawnienia, brak sekretów w repozytorium. |
| **PostgreSQL Complaint Registry** | Rejestr operacyjny | Statusy, metadane, wyniki AI, referencje JIRA/Blob | Zastępuje Excel jako źródło operacyjne. |
| **Azure Blob Storage** | Archiwum załączników | Zdjęcia, pliki, linki, suma kontrolna | PostgreSQL przechowuje metadane, nie binaria. |
| **AI Extraction & Classification Service** | Interpretacja treści | Strukturalny JSON, poziom pewności AI, kategoria, podsumowanie | AI sugeruje, nie decyduje finalnie. |
| **Pydantic / walidacja schematu** | Kontrola jakości danych po AI | Wynik AI | Sprawdza format, wymagane pola i dozwolone wartości. |
| **PostgreSQL Customer DB** | Dopasowanie klienta | Klient, adres e-mail, domena, identyfikator | Tylko do odczytu; brak zapisu do bazy klientów. |
| **SAP ERP PP/QM** | Walidacja zamówienia i partii | Dane zamówienia, dane partii, kontekst produkcji | Tylko do odczytu w **MVP**; `GET /api/v1/orders/{id}`, `GET /api/v1/batches/{id}`, limit 100 req/min. |
| **JIRA Cloud / REK** | Miejsce pracy zespołu | Complaint, Correction, status weryfikacji | JIRA pozostaje miejscem pracy człowieka. |
| **Metrics Dashboard** | Raportowanie | Wolumen, czasy, kategorie, błędy, jakość AI, backlog | Dane z Complaint Registry, JIRA i integracji. |

## 8. Kierunek technologiczny

Proponowany stack powinien być praktyczny, nowoczesny i bezpieczny operacyjnie. Celem nie jest stworzenie narzędzia do swobodnej rozmowy ani zbioru prostych skryptów, tylko kontrolowanego procesu, w którym każdy krok ma wejście, wynik, walidację, status i ścieżkę błędu.

| Obszar | Propozycja |
| --- | --- |
| Warstwa API i integracji | **Python FastAPI** jako lekka warstwa API, integracji i endpointów technicznych. |
| Kontrolowany przepływ AI | **LangGraph** albo podobny framework grafowy / state-machine do modelowania procesu jako sekwencji stanów. |
| Walidacja danych AI | **Pydantic** do ścisłej walidacji strukturalnych wyników AI. |
| Baza procesu | **PostgreSQL Complaint Registry** jako rejestr procesu, statusów, audytu i metryk. |
| Istniejąca baza klientów | PostgreSQL Customer DB tylko do odczytu. |
| Przechowywanie załączników | **Azure Blob Storage**. |
| Przetwarzanie asynchroniczne w MVP | Procesy działające w tle, kolejka zadań oparta o PostgreSQL i zadanie harmonogramowe dla ponowień oraz kontrolnego skanowania skrzynki. |
| Kolejka / ponowienia później | Azure Service Bus albo RabbitMQ, jeśli wzrośnie złożoność. |
| AI | Model językowy do ekstrakcji tekstu, klasyfikacji, podsumowania i roboczej odpowiedzi. |
| Analiza zdjęć | Opcjonalnie jako V2 albo selektywne rozszerzenie MVP. |
| Integracje | Microsoft Graph API, SAP REST API, JIRA REST API, PostgreSQL, Azure Blob SDK. |
| Monitoring | Logi aplikacyjne, logi audytowe, logi błędów integracji, logi ponowień i podstawowy dashboard metryk. |

**LangGraph** albo podobny graf stanów pozwala modelować proces jako jawne kroki:

1. przyjęcie wiadomości,
2. pobranie treści,
3. ekstrakcja danych,
4. walidacja schematu,
5. dopasowanie klienta,
6. walidacja SAP,
7. ocena reguł wyboru ścieżki,
8. przygotowanie JIRA,
9. obsługa wyjątków.

Takie podejście pozwala kontrolować każdy etap procesu: wejście, wyjście, walidację, status, błąd i decyzję o dalszej ścieżce. To nie jest swobodny model decyzyjny poza procesem. Dzięki temu łatwiej diagnozować proces, obsługiwać ponowienia, audytować decyzje i bezpiecznie kierować sprawy do ręcznej weryfikacji.

Jeżeli w rozwiązaniu zostanie użyty LangGraph, AI nadal pozostaje ograniczone:

- AI ekstrahuje i sugeruje,
- **Pydantic** waliduje strukturalne wyniki,
- reguły deterministyczne decydują o ścieżce procesu,
- **JIRA** pozostaje miejscem pracy człowieka.

Finalny wybór frameworka powinien zostać potwierdzony podczas konsultacji technicznej. Rekomendowany kierunek to kontrolowany graf stanów, a nie swobodny mechanizm podejmujący decyzje poza procesem.

## 9. Zasady ręcznej weryfikacji

Ręczna weryfikacja jest wymagana, gdy występuje:

- niski poziom pewności AI,
- brak order ID,
- klient nie został znaleziony,
- niezgodność klienta i zamówienia,
- SAP order not found,
- partia nie została znaleziona w SAP,
- wiele defektów w jednej wiadomości,
- nieobsługiwany załącznik,
- nieczytelne zdjęcia,
- niejednoznaczny opis,
- błąd walidacji schematu,
- błąd integracji,
- dowolna decyzja wysokiego ryzyka.

To ważne, ponieważ reklamacje mają wpływ na klienta, koszty, jakość produkcji i audyt. AI może źle zinterpretować wiadomość, zdjęcie albo kontekst biznesowy. Ręczna weryfikacja zmniejsza ryzyko błędnej decyzji, ale nadal pozwala automatyzacji przygotować dane i skrócić czas obsługi.

## 10. Obsługa wyjątków

### Kolejka ręcznej weryfikacji

Kolejka ręcznej weryfikacji obsługuje niepewność biznesową:

- brak order ID,
- klient nie został znaleziony,
- niski poziom pewności AI,
- niezgodność klienta i zamówienia,
- wiele defektów,
- niejasna kategoria,
- wynik AI nie przeszedł walidacji schematu.

Sprawa jest widoczna w JIRA jako element wymagający decyzji specjalisty serwisu.

### Kolejka ponowień

Kolejka ponowień obsługuje tymczasowe błędy techniczne:

- Graph API timeout,
- SAP timeout,
- SAP rate limit,
- JIRA timeout,
- Blob timeout,
- AI service unavailable.

Ponowienie powinno używać backoff, limitu prób i zapisu audytowego. Ponowienie nie może tworzyć duplikatów JIRA ani duplikatów reklamacji.

### Kolejka błędów trwałych

Kolejka błędów trwałych obsługuje błędy powtarzalne albo trwałe:

- przekroczony limit ponowień,
- błąd uprawnień,
- uszkodzony załącznik,
- niepoprawny pakiet danych,
- trwała awaria integracji.

Sprawy w kolejce błędów trwałych wymagają przeglądu operacyjnego przed ponownym przetworzeniem albo zamknięciem.

### Kontrolne skanowanie skrzynki

Kontrolne skanowanie skrzynki ogranicza ryzyko zgubienia reklamacji. Zaplanowane zadanie skanuje Inbox/Junk/Spam, sprawdza `message_id` w Complaint Registry, wykrywa pominięte reklamacje i wysyła je ponownie do usługi przyjęcia wiadomości. To zabezpiecza proces przed opóźnionym odczytem, spamem i awarią webhooka.

## 11. Dane i metryki

**PostgreSQL Complaint Registry** zastępuje Excel jako centralny rejestr operacyjny procesu reklamacyjnego. **JIRA** pozostaje miejscem pracy, ale rejestr przechowuje stan procesu, idempotencję, audyt techniczny i dane do **metryk**.

Rejestr powinien przechowywać:

- complaint ID,
- message ID,
- wynik dopasowania klienta,
- order ID,
- identyfikator partii,
- wynik ekstrakcji AI,
- sugestię kategorii,
- poziom pewności AI,
- wynik walidacji schematu,
- flaga i powód ręcznej weryfikacji,
- referencje do zgłoszeń JIRA,
- linki do załączników w Azure Blob Storage,
- status,
- timestamps,
- wpisy audytowe.

Metryki procesowe:

- liczba reklamacji miesięcznie,
- reklamacje według kategorii,
- średni czas pierwszej odpowiedzi,
- czas od wpływu wiadomości do utworzenia zgłoszenia JIRA,
- backlog,
- udział spraw wymagających ręcznej weryfikacji,
- reklamacje według partii,
- reklamacje według linii produkcyjnej, jeśli dane są dostępne,
- liczba JIRA Correction,
- liczba błędów Graph/SAP/JIRA/Blob,
- liczba reklamacji odzyskanych przez kontrolne skanowanie skrzynki.

Metryki jakości AI:

- rozkład poziomu pewności AI,
- udział spraw skierowanych do ręcznej weryfikacji,
- trafność sugestii kategorii AI na podstawie korekt człowieka,
- procent sugestii AI zaakceptowanych bez zmiany,
- najczęściej korygowane kategorie,
- odsetek spraw z brakującymi informacjami,
- niski poziom pewności AI według kategorii,
- przypadki błędnego wyboru ścieżki procesu,
- liczba przypadków, w których Pydantic / walidacja schematu zakończyła się błędem.

Te metryki pomagają poprawiać model, instrukcje dla modelu, definicje kategorii i progi pewności. Dzięki nim firma może stopniowo zmniejszać udział ręcznej weryfikacji, ale bez ślepego zaufania do AI.

Mapowanie na pytania CEO:

| Pytanie | Metryka |
| --- | --- |
| Ile mamy reklamacji? | Liczba reklamacji miesięcznie. |
| Jakiego typu są reklamacje? | Rozkład kategorii reklamacji. |
| Które partie albo linie mają problemy? | Reklamacje według partii i linii produkcyjnej. |
| Jak długo trwa obsługa? | Czas pierwszej odpowiedzi, czas od wpływu wiadomości do utworzenia JIRA, backlog. |
| Ile pracy nadal wymaga człowieka? | Udział ręcznej weryfikacji i jej powody. |
| Jak dobrze działa AI? | Trafność sugestii, korekty człowieka, poziom pewności AI, błędy walidacji schematu. |
| Gdzie zawodzą integracje? | Liczba błędów Graph/SAP/JIRA/Blob. |

## 12. Zakres MVP

Realistyczny **MVP** obejmuje:

- Microsoft Graph Webhook + pobranie wiadomości,
- archiwizację załączników w **Azure Blob Storage**,
- **PostgreSQL Complaint Registry**,
- ekstrakcję danych z tekstu przez AI,
- walidację wyniku AI przez Pydantic / schemat danych,
- sugestię kategorii i poziom pewności AI,
- dopasowanie klienta,
- walidację zamówienia i partii w SAP,
- utworzenie JIRA Complaint,
- ręczną weryfikację w JIRA,
- podstawowe metryki procesowe i AI,
- kontrolne skanowanie skrzynki dla pominiętych wiadomości i spamu,
- obsługę ponowień dla tymczasowych błędów integracji.

## 13. Poza zakresem

Poza zakresem **MVP**:

- pełna autonomiczna akceptacja albo odrzucanie reklamacji,
- zapisywanie decyzji do SAP,
- zastąpienie JIRA,
- zastąpienie SAP,
- portal klienta,
- pełny dedykowany model computer vision,
- zaawansowany dashboard BI,
- automatyczne rekomendacje jakościowe,
- trenowanie modelu AI od zera,
- samodzielne uczenie się modelu w produkcji bez kontroli,
- automatyczne tworzenie Correction bez kontroli procesu.

## 14. Kluczowe decyzje

1. **PostgreSQL Complaint Registry zastępuje Excel.**
   Excel nie jest dobrym rejestrem operacyjnym dla statusów, audytu, idempotencji i metryk.

2. **JIRA pozostaje miejscem pracy zespołu.**
   Zespół już pracuje w JIRA, więc MVP nie wymaga dedykowanego UI do weryfikacji zgłoszeń.

3. **SAP jest tylko do odczytu w MVP.**
   SAP pozostaje źródłem prawdy dla zamówienia i partii, ale rozwiązanie nie zapisuje do SAP.

4. **AI sugeruje, ale nie decyduje.**
   AI strukturyzuje dane i sugeruje kategorię, ale finalne ryzykowne decyzje należą do procesu i człowieka.

5. **Pydantic waliduje strukturę wyniku AI.**
   Walidacja schematu nie rozstrzyga reklamacji, lecz sprawdza, czy dane po AI nadają się do dalszego procesu.

6. **Reguły deterministyczne wybierają ścieżkę procesu.**
   Wybór ścieżki opiera się na jawnych zasadach, a nie na swobodnej decyzji modelu.

7. **Ręczna weryfikacja chroni sprawy ryzykowne.**
   Review jest mechanizmem kontroli jakości, nie porażką automatyzacji.

8. **Blob przechowuje załączniki, PostgreSQL przechowuje metadane.**
   Zdjęcia i pliki nie powinny być przechowywane bezpośrednio w tabelach procesu.

9. **Ponowienia, kolejka błędów trwałych i kontrolne skanowanie skrzynki chronią przed utratą reklamacji.**
   Proces musi kontrolować awarie integracji i pominięte wiadomości.

10. **Metryki wynikają z rejestru operacyjnego.**
   Metryki opierają się na realnych statusach procesu, a nie na ręcznym liczeniu w Excelu.

## 15. Trade-offs

| Decyzja | Korzyść | Koszt / trade-off |
| --- | --- | --- |
| AI + ręczna weryfikacja | Szybsza strukturyzacja wiadomości przy kontroli ryzyka | Część spraw nadal wymaga człowieka. |
| Pydantic / walidacja schematu | Bezpieczniejsze użycie wyników AI | Wymaga utrzymania schematów i obsługi błędów walidacji. |
| Reguły deterministyczne | Powtarzalny i audytowalny wybór ścieżki procesu | Trzeba utrzymywać progi i reguły biznesowe. |
| PostgreSQL zamiast Excela | Status, audyt, idempotencja i metryki | Dodatkowy komponent utrzymaniowy. |
| JIRA jako narzędzie obsługi zamiast dedykowanego UI | Szybszy MVP i zgodność z pracą zespołu | Mniej ergonomiczne niż dedykowany panel. |
| SAP tylko do odczytu w **MVP** | Mniejsze ryzyko integracyjne i bezpieczeństwa | Brak automatycznego zapisu decyzji do SAP. |
| Blob dla załączników | Skalowalne i bezpieczne przechowywanie zdjęć | Trzeba zarządzać linkami, SAS tokens i retencją. |
| Analiza zdjęć w V2 | MVP pozostaje prosty i wiarygodny | Mniej automatycznego wsparcia dla oceny zdjęć. |
| Kolejka ponowień / kolejka błędów trwałych | Kontrola awarii i audyt błędów | Wymaga monitorowania kolejek. |
| Kontrolne skanowanie skrzynki | Zmniejsza ryzyko zgubienia wiadomości | Dodatkowe zadanie harmonogramowe i logika idempotencji. |
| Prostota MVP vs skalowalność długoterminowa | Szybsze wdrożenie i mniejszy koszt | W przyszłości może być potrzebna osobna kolejka typu Azure Service Bus. |

## 16. Elementy wymagające doprecyzowania przed wdrożeniem

Ta specyfikacja opisuje kierunek procesowy i architektoniczny. Przed wdrożeniem produkcyjnym wymagają doprecyzowania:

- szczegółowy model danych,
- mapowanie pól **JIRA Complaint** i **JIRA Correction**,
- mapowanie odpowiedzi z **SAP ERP PP/QM**,
- reguły dopasowania klienta w Customer DB,
- instrukcje dla modelu AI i wersjonowanie modelu,
- progi poziomu pewności AI,
- szczegółowa polityka ponowień,
- procedura obsługi kolejki błędów trwałych,
- model kontroli dostępu i bezpieczeństwa,
- polityka retencji załączników,
- monitoring i alerting,
- szczegółowy backlog i plan wdrożenia.

Te elementy powinny zostać opisane dopiero po potwierdzeniu zakresu **MVP**, priorytetów biznesowych, dostępności danych i ograniczeń integracyjnych.

## 17. Rekomendowany kolejny krok

Rekomendowanym kolejnym krokiem są trzy warsztaty doprecyzowujące: biznesowo-procesowy, techniczno-integracyjny oraz wdrożeniowo-priorytetyzacyjny. Ich celem byłoby ustalenie finalnego zakresu MVP, budżetu, ryzyk, zależności integracyjnych i kolejności prac.

Dalsze prace powinny obejmować etap discovery/warsztatowy, w którym doprecyzowane zostaną: budżet, zakres MVP, priorytety wdrożenia, szczegółowy backlog, model danych, reguły walidacji, mapowanie integracji oraz wymagania produkcyjne.

## 18. Uzasadnienie rozwiązania

Rozwiązanie jest uzasadnione biznesowo i technicznie, ponieważ porządkuje proces przy użyciu istniejących systemów zamiast zastępować wszystko nową platformą. **SAP ERP PP/QM** pozostaje źródłem prawdy dla zamówień i partii, a **JIRA** pozostaje miejscem pracy zespołu serwisowego i jakości.

AI jest ograniczone do interpretacji nieustrukturyzowanej treści i przygotowania danych. **Pydantic** sprawdza, czy wynik AI ma poprawny format. Walidacja źródeł prawdy, obsługa integracji, limit SAP, wybór ścieżki procesu i audyt pozostają deterministyczne. **Ręczna weryfikacja** chroni przypadki niepewne, niekompletne i ryzykowne.

**PostgreSQL Complaint Registry** zastępuje Excel jako centralny rejestr operacyjny statusu procesu. **Azure Blob Storage** przechowuje zdjęcia i załączniki, a PostgreSQL przechowuje metadane oraz linki. Mechanizmy ponowień, kolejka błędów trwałych i kontrolne skanowanie skrzynki zmniejszają ryzyko zgubienia reklamacji lub utraty kontroli nad błędami integracji.

Rozwiązanie daje **metryki**, których brakuje w **AS-IS**: wolumen, kategorie, backlog, czas od wpływu wiadomości do utworzenia zgłoszenia JIRA, udział ręcznej weryfikacji, jakość sugestii AI, błędy integracji i problemy według partii lub linii produkcyjnej, jeżeli SAP zwraca takie dane. Jest realistyczne dla 600 reklamacji miesięcznie i szczytów do 2000 miesięcznie. Limit SAP wynosi 100 req/min, co przy zakładanym wolumenie jest wystarczającą przepustowością dla tego typu rozwiązania. Integracja powinna jednak respektować limity SAP i obsługiwać ponowienia w kontrolowany sposób.

## 19. Skrót koncepcji rozwiązania

Rozwiązanie porządkuje obsługę reklamacji jako kontrolowany proces przyjęcia, walidacji i dalszej obsługi zgłoszenia. **Microsoft Graph** wykrywa i pobiera wiadomość, **PostgreSQL Complaint Registry** zastępuje Excel jako operacyjny rejestr reklamacji, a **Azure Blob Storage** przechowuje zdjęcia i załączniki.

AI porządkuje treść wiadomości, wyciąga kluczowe dane i sugeruje kategorię defektu, ale nie podejmuje finalnych decyzji. Wynik AI jest sprawdzany strukturalnie, **SAP ERP PP/QM** waliduje zamówienie i partię, a jawne reguły kierują sprawę do właściwej ścieżki: standardowej obsługi, **ręcznej weryfikacji**, ponowienia albo kolejki błędów trwałych.

**JIRA** pozostaje miejscem pracy specjalisty serwisu, który otrzymuje przygotowaną sprawę zamiast zaczynać od surowej wiadomości. Dzięki temu proces staje się bardziej powtarzalny, mierzalny, audytowalny i odporny na zgubienie reklamacji.
