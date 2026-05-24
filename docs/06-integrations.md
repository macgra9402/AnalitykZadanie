# 06. Integracje

## Zasada integracyjna

Integracje są wykonywane przez adaptery kontrolowane przez Complaint Orchestrator. AI nie wywołuje API i nie obsługuje dostępności systemów zewnętrznych.

Każda integracja zapisuje wynik, błąd techniczny i decyzję fallback w Operational Database oraz Audit Log.

## Microsoft 365 / Exchange / Microsoft Graph

| Obszar | Opis |
| --- | --- |
| Purpose | Wykrycie nowej wiadomości reklamacyjnej i pobranie jej treści oraz załączników. |
| Data read/written | Odczyt: `message_id`, nadawca, temat, body, attachment metadata, attachment content. Zapis w procesie: `source_email_id`, `received_at`, status pobrania. |
| Failure mode | Webhook nie dotarł, token wygasł, Graph zwrócił timeout, attachment fetch się nie udał. |
| Retry/manual fallback | Idempotencja po `source_email_id`; retry pobrania; okresowy reconciliation scan skrzynki jako zabezpieczenie operacyjne; human review, jeśli sprawa została wykryta bez kompletnych danych. |
| Security consideration | OAuth2, least privilege do wymaganej skrzynki/folderu, brak sekretów w repo, audyt dostępu do emaili. |

## Azure Blob Storage

| Obszar | Opis |
| --- | --- |
| Purpose | Archiwizacja zdjęć i załączników reklamacyjnych. |
| Data read/written | Zapis: plik, typ MIME, rozmiar, checksum, `blob_uri`, czas archiwizacji. Odczyt: tylko przez uprawnione komponenty procesu lub review. |
| Failure mode | Plik za duży, format nieobsługiwany, upload timeout, błąd SAS token, obraz nieczytelny. |
| Retry/manual fallback | Retry uploadu dla błędów technicznych; unsupported/too large/unreadable image kieruje sprawę do human review; email pozostaje źródłem referencyjnym. |
| Security consideration | Krótkożyjące SAS tokens, brak SAS tokens w repozytorium i logach, retention policy, minimalizacja dostępu. |

## PostgreSQL customer database

| Obszar | Opis |
| --- | --- |
| Purpose | Potwierdzenie klienta na podstawie danych z emaila i nadawcy. |
| Data read/written | Odczyt: klient, email, identyfikator klienta, ewentualne dane dopasowania. Brak zapisu, baza jest read-only. |
| Failure mode | Brak klienta, wiele pasujących klientów, timeout, brak dostępu. |
| Retry/manual fallback | Retry dla błędów technicznych; `not found` lub ambiguous match kieruje sprawę do human review. |
| Security consideration | Read-only account, least privilege, brak modyfikacji danych klienta, logowanie tylko minimalnych danych potrzebnych do audytu. |

## SAP ERP PP/QM REST API

Endpointy:

- `GET /api/v1/orders/{id}`
- `GET /api/v1/batches/{id}`

| Obszar | Opis |
| --- | --- |
| Purpose | Walidacja numeru zamówienia i partii oraz pobranie danych produkcyjnych, jeśli są dostępne. |
| Data read/written | Odczyt: status zamówienia, dane partii, produkt, linia, operator lub inne pola dostępne w SAP. Zapis w procesie: referencja i status walidacji. |
| Failure mode | Invalid order, batch not found, timeout, niedostępność SAP, przekroczenie rate limit. |
| Retry/manual fallback | Backoff i kolejka dla błędów technicznych; kontrola limitu 100 req/min; invalid/not found kieruje sprawę do human review. |
| Security consideration | Autoryzowany dostęp tylko do endpointów order/batch, audyt zapytań, brak zapisu do SAP. |

Limit 100 req/min jest wystarczający dla wolumenu 600 reklamacji miesięcznie i sezonowych wzrostów do około 2000 miesięcznie, bo jedna sprawa wymaga zwykle jednego sprawdzenia zamówienia i ewentualnie jednego sprawdzenia partii. Ryzykiem nie jest średni miesięczny wolumen, tylko krótkie spiętrzenia po weekendzie, awarii albo ponowieniu zaległej kolejki. Dlatego potrzebne są kolejkowanie, backoff, throttling oraz metryka opóźnienia walidacji SAP.

## JIRA Cloud

| Obszar | Opis |
| --- | --- |
| Purpose | Utworzenie i aktualizacja spraw w projekcie `REK`: `Complaint` oraz po potwierdzeniu defektu `Correction`. |
| Data read/written | Zapis: opis reklamacji, kategoria, dane order/batch, linki do załączników, status review, referencje audytowe. Odczyt: status sprawy i klucz JIRA. |
| Failure mode | Brak dostępu, błąd mapowania pól, timeout, duplikat tworzenia, niedostępność JIRA. |
| Retry/manual fallback | Idempotency key po complaint id; retry tworzenia; jeśli JIRA jest niedostępna, sprawa zostaje w Operational Database jako pending JIRA creation i trafia do review operacyjnego. |
| Security consideration | OAuth2/API credentials poza repo, least privilege do projektu `REK`, audyt tworzenia i aktualizacji issue. |

## Operational Database

Brief wskazuje PostgreSQL customer database jako read-only źródło klientów. Oprócz niego automatyzacja potrzebuje własnej warstwy operacyjnej do statusów, idempotencji, audytu technicznego i metryk. To nie zastępuje JIRA ani SAP; jest minimalną bazą procesu.

Przechowywane dane:

- status reklamacji,
- `source_email_id`,
- referencje Blob i JIRA,
- wynik AI i wersje prompt/model,
- decyzje routingu,
- błędy integracji,
- czasy przejść procesu.

## Monitoring and Reporting Layer

Warstwa raportowania korzysta z Operational Database i statusów JIRA. Nie wymaga zaawansowanej platformy BI w MVP.

Metryki:

- intake delay,
- email-to-JIRA time,
- backlog,
- category distribution,
- manual review rate,
- AI confidence distribution,
- integration failures,
- complaint sources by batch/operator/line, jeśli SAP zwraca takie dane,
- SLA / response time,
- duplicate rate.
