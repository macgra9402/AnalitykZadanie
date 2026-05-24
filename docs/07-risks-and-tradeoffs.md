# 07. Ryzyka i trade-offy

## Ryzyka

### Błędna interpretacja AI

AI może źle odczytać numer zamówienia, kategorię, język albo opis defektu. Dlatego wynik AI jest sugestią, przechodzi schema validation, a routing opiera się na regułach, SAP i PostgreSQL.

### Niepełne dane w emailu

Klient może nie podać numeru zamówienia, numeru partii lub pełnego opisu. Taki przypadek trafia do human review i może dostać draft prośby o uzupełnienie.

### Awaria SAP, JIRA lub Graph

Integracje są punktami zależności. System zapisuje błąd, stosuje retry/backoff i kieruje sprawę do review, jeżeli brak danych blokuje bezpieczną obsługę.

### Rate limit SAP

SAP ma limit 100 req/min. Normalne wolumeny reklamacji powinny być obsługiwalne, ale peak albo retry po awarii może wygenerować kolejkę. Potrzebne są throttling, backoff i metryka opóźnienia walidacji SAP.

### Duplikaty

Klient może wysłać kilka emaili w tej samej sprawie. W MVP wykrycie duplikatu jest ostrożne i prowadzi do human review, zamiast automatycznie zamykać zgłoszenie.

### Załączniki

Zdjęcie może być za duże, w nieobsługiwanym formacie albo nieczytelne. System powinien zapisać status archiwizacji, ale nie powinien udawać, że analiza zdjęcia jest wiarygodna.

### Dane osobowe i zdjęcia

Email i załączniki mogą zawierać dane osobowe lub wrażliwe informacje produkcyjne. Potrzebne są minimalizacja danych, retention policy, least privilege i audyt.

## Trade-offy

| Decyzja | Korzyść | Koszt / ryzyko |
| --- | --- | --- |
| LLM vs rules | LLM dobrze czyta nieustrukturyzowany email | LLM nie jest źródłem prawdy, więc potrzebne są reguły i walidacje |
| Human-in-the-loop vs full automation | Mniejsze ryzyko błędnych decyzji reklamacyjnych | Część spraw nadal wymaga pracy specjalisty |
| JIRA review queue vs custom UI | Szybciej, bo JIRA już istnieje | Mniej ergonomiczny review niż dedykowany ekran |
| Image analysis in MVP vs later | MVP może dodać ostrożne image hints | Brak automatycznej ekspertyzy jakościowej zdjęć |
| Excel coexistence during transition | Łatwiejsza zmiana procesu i porównanie metryk | Ryzyko równoległych źródeł statusu, więc okres przejściowy musi być krótki |
| Operational Database vs only JIRA | Idempotencja, audyt techniczny i metryki | Dodatkowy komponent utrzymaniowy |
| Speed of delivery vs auditability | MVP szybciej pokaże wartość | Nie wolno pominąć logów, wersji promptu i decyzji routingu |
| Zachowanie emaila jako kanału | Brak zmiany po stronie klienta | Trzeba obsłużyć spam, braki i nieustrukturyzowane treści |
| SAP jako źródło prawdy | Wiarygodna walidacja order/batch | Limit 100 req/min i zależność od dostępności |

## Jak unikamy overengineeringu

- Nie budujemy portalu, bo brief wskazuje email jako obecny kanał.
- Nie trenujemy własnego modelu vision, bo MVP potrzebuje triage, nie automatycznej ekspertyzy zdjęć.
- Nie zastępujemy SAP ani JIRA.
- Nie tworzymy złożonego workflow engine, jeśli wystarczą jasne reguły routingu.
- Nie pozwalamy AI wywoływać API ani podejmować decyzji audytowych.
- Nie wprowadzamy systemów spoza briefu jako zależności MVP.
- Operational Database ograniczamy do idempotencji, statusów, audytu i metryk procesu.
