# Wizja biznesowa rozwiązania — Metabot / Metalpol

## Zakres dokumentu

Ten dokument opisuje biznesową wizję usprawnienia procesu reklamacyjnego. Pokazuje, jaki problem rozwiązujemy, co zmienia się dla pracowników i jakie korzyści może uzyskać firma.

Dokument koncentruje się na uporządkowaniu procesu, odpowiedzialności, powtarzalnych czynności, punktów kontroli człowieka i mierzalnych efektów automatyzacji AI.

Nie jest to kompletna specyfikacja wdrożeniowa ani gotowy plan implementacji. Szczegółowy budżet, zakres MVP, harmonogram, konfiguracja systemów, mapowania integracji i finalny backlog powinny zostać doprecyzowane w osobnym etapie discovery/warsztatowym.

## 1. Problem w jednym zdaniu

Dziś reklamacje są obsługiwane ręcznie, przez co firma traci czas, spójność danych i widoczność problemów jakościowych.

## 2. Co dziś nie działa

Obecny proces zależy od ręcznej pracy specjalisty serwisu i od kilku oddzielnych narzędzi.

Najważniejsze problemy:

- wiadomość reklamacyjna może trafić do spamu albo zostać przeczytana z opóźnieniem,
- dane z wiadomości są przepisywane ręcznie do Excela,
- kategorie defektów są przypisywane niespójnie,
- JIRA i SAP są sprawdzane ręcznie,
- zarządzanie nie ma wiarygodnych metryk o reklamacjach,
- specjalista serwisu działa jak ręczny łącznik między Exchange, Excel, JIRA i SAP.

W praktyce oznacza to opóźnienia, backlog, trudniejsze raportowanie i większe ryzyko błędów.

## 3. Co zmienia rozwiązanie

Rozwiązanie porządkuje reklamację zanim trafi ona do pracownika.

System odbiera wiadomość reklamacyjną, rejestruje zgłoszenie, wyciąga kluczowe informacje, zapisuje zdjęcia i załączniki, sprawdza klienta, zamówienie oraz partię, a następnie przygotowuje zgłoszenie JIRA do weryfikacji.

Specjalista serwisu nie zaczyna już od surowej wiadomości i pustego formularza. Otwiera przygotowaną sprawę, widzi najważniejsze dane w jednym miejscu i może skupić się na ocenie, decyzji oraz komunikacji z klientem.

## 4. Rola AI

AI pomaga uporządkować treść reklamacji. Nie podejmuje decyzji za firmę.

AI wspiera proces przez:

- odczytanie i uporządkowanie treści wiadomości e-mail,
- wskazanie numeru zamówienia,
- streszczenie reklamacji,
- zasugerowanie kategorii defektu,
- wykrycie brakujących informacji,
- przygotowanie wersji roboczej odpowiedzi,
- oznaczenie niejasnych spraw do ręcznej weryfikacji.

AI nie uznaje ani nie odrzuca reklamacji. Nie zastępuje specjalisty serwisu. Nie zapisuje danych do SAP. Jego rolą jest przygotowanie informacji, a nie finalna decyzja.

**AI przygotowuje sprawę. Człowiek podejmuje decyzję.**

## 5. Jak zmienia się praca serwisanta

Przed zmianą specjalista serwisu:

- czyta surową wiadomość e-mail,
- kopiuje dane do Excela,
- sprawdza SAP ręcznie,
- tworzy JIRA ręcznie,
- pisze odpowiedź od zera.

Po zmianie specjalista serwisu:

- otwiera przygotowany JIRA Complaint,
- weryfikuje wyciągnięte dane,
- sprawdza sugerowaną kategorię,
- zatwierdza albo koryguje decyzję,
- wysyła lub poprawia przygotowaną odpowiedź.

Zmiana polega na przejściu od ręcznego przepisywania danych do kontroli jakości, decyzji i obsługi klienta.

## 6. Co zyskuje firma

Firma zyskuje bardziej kontrolowany i mierzalny proces reklamacyjny.

Najważniejsze korzyści:

- krótszy czas odpowiedzi do klienta,
- mniej zgubionych lub opóźnionych reklamacji,
- mniej ręcznego kopiowania danych,
- bardziej spójne kategorie defektów,
- lepsza widoczność powtarzających się problemów,
- metryki backlogu i czasu obsługi,
- lepszy obraz problematycznych partii lub linii produkcyjnych,
- proces oparty na rejestrze reklamacji zamiast na ręcznym śledzeniu w Excelu.

## 7. Co zyskuje zarząd i kierownictwo

Zarząd i kierownictwo dostają dane, których dziś brakuje albo które trzeba zbierać ręcznie.

Przykładowe informacje:

- ile reklamacji wpływa miesięcznie,
- jakie kategorie defektów dominują,
- jaki jest backlog,
- jaki jest średni czas odpowiedzi,
- które partie lub linie produkcyjne mają więcej reklamacji, jeśli dane są dostępne,
- ile spraw wymaga ręcznej weryfikacji,
- gdzie pojawiają się błędy integracji.

Dzięki temu reklamacje przestają być tylko operacyjnym obciążeniem, a zaczynają być źródłem informacji o jakości produkcji i obsługi klienta.

## 8. Dlaczego nie pełna automatyzacja

Pełna automatyzacja decyzji reklamacyjnych byłaby ryzykowna. Reklamacje wpływają na relacje z klientami, koszty, decyzje jakościowe i odpowiedzialność firmy.

Dlatego rozwiązanie zostawia człowieka w procesie tam, gdzie sprawa jest niepewna, niekompletna albo ryzykowna. AI może przygotować dane i propozycję odpowiedzi, ale nie powinno samodzielnie decydować o uznaniu albo odrzuceniu reklamacji.

**Celem nie jest usunięcie człowieka z procesu, tylko usunięcie z jego pracy ręcznego przepisywania i szukania informacji.**

## 9. MVP — pierwszy rozsądny etap

Pierwszy etap powinien być praktyczny i możliwy do wdrożenia bez przebudowy całej firmy.

MVP obejmuje:

- automatyczne przetwarzanie wiadomości reklamacyjnych,
- centralny rejestr reklamacji zamiast Excela,
- archiwum zdjęć i załączników,
- wsparcie AI w odczycie danych i sugestii kategorii,
- sprawdzenie zamówienia i partii w SAP,
- przygotowane zgłoszenie JIRA Complaint,
- ręczną weryfikację spraw niepewnych,
- podstawowe metryki dla zarządzania,
- mechanizm wykrywania pominiętych wiadomości z Inbox/Junk/Spam.

To wystarcza, żeby ograniczyć największe straty czasu i poprawić widoczność procesu bez budowania dużej, kosztownej platformy.

## 10. Krótkie podsumowanie

Rozwiązanie zmienia ręczny, rozproszony proces reklamacyjny w proces bardziej kontrolowany, mierzalny i bezpieczny.

AI wspiera osoby obsługujące reklamacje, ale nie przejmuje ich odpowiedzialności. Firma zyskuje krótszy czas obsługi, mniej ręcznej pracy i lepszy obraz problemów jakościowych.
