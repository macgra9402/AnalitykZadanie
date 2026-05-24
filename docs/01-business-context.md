# 01. Kontekst biznesowy

## Firma i proces

Metabot Sp. z o.o. jest fikcyjnym producentem metalowych komponentów dla branży automotive. Obsługa reklamacji klientowskich odbywa się obecnie ręcznie, głównie przez email, Excel, JIRA i SAP.

Klient wysyła email reklamacyjny z numerem zamówienia, opisem problemu i zdjęciami defektu. Specjalista serwisowy odczytuje wiadomość, przepisuje dane do Excela, klasyfikuje defekt, zakłada zgłoszenie JIRA Complaint, sprawdza dane zamówienia lub partii w SAP, odpowiada klientowi i w potwierdzonych przypadkach tworzy JIRA Correction dla jakości.

## Problemy AS-IS

- Email może trafić do spamu albo zostać przeczytany za późno.
- W okresach szczytu rośnie backlog i czas pierwszej reakcji.
- Klasyfikacja defektów jest niespójna między specjalistami.
- Excel pełni rolę ręcznego mostu procesowego, ale nie jest stabilnym źródłem audytu.
- SAP i JIRA nie są zintegrowane, więc dane są kopiowane ręcznie.
- Brakuje metryk: liczby reklamacji, rozkładu kategorii, czasu do JIRA, udziału review manualnego i awarii integracji.

## Cel biznesowy

Celem jest usprawnienie intake i triage reklamacji przez automatyczne wykrywanie emaili, archiwizację zdjęć, ekstrakcję danych przez AI, walidację w systemach źródłowych i utworzenie JIRA Complaint.

Rozwiązanie ma skracać czas od przyjścia emaila do utworzenia sprawy, ograniczać przepisywanie danych oraz zwiększać spójność klasyfikacji. Nie ma zastępować człowieka w decyzjach wysokiego ryzyka.

## Dostępne systemy

- Microsoft 365 / Exchange: Microsoft Graph API, OAuth2, webhook dla nowych emaili.
- SAP ERP PP/QM REST API:
  - `GET /api/v1/orders/{id}`
  - `GET /api/v1/batches/{id}`
  - rate limit 100 req/min.
- JIRA Cloud: projekt `REK`, issue types `Complaint` i `Correction`.
- Wewnętrzna baza PostgreSQL klientów: read-only.
- Azure Blob Storage z SAS tokens do archiwum zdjęć defektów.

## Ograniczenie rozwiązania

Projekt nie zakłada portalu klienta, zastąpienia SAP lub JIRA, pełnej automatycznej akceptacji/rejekcji reklamacji ani trenowania własnego modelu computer vision. Priorytetem jest realny MVP możliwy do uzasadnienia biznesowo i technicznie.
