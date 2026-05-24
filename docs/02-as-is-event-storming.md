# 02. AS-IS Event Storming

## Cel

Ten dokument opisuje obecny proces reklamacyjny w języku Event Storming. Główny diagram AS-IS znajduje się w [diagrams/metabot-as-is.drawio](../diagrams/metabot-as-is.drawio).

## Legenda pojęć

- Domain Event: coś, co już się wydarzyło.
- Command: intencja albo prośba wykonania działania.
- Actor: osoba lub system inicjujący command.
- External System: system spoza domeny procesu, np. Exchange, Excel, JIRA, SAP.
- Policy / Business Rule: reguła decydująca o następnym kroku.
- Pain Point: opóźnienie, praca ręczna, niespójność albo brak informacji.

## Aktorzy

- Customer: wysyła reklamację.
- Service Specialist: czyta email, przepisuje dane, klasyfikuje defekt i komunikuje się z klientem.
- Quality Team: obsługuje korekcję po potwierdzeniu defektu.

## External Systems

- Exchange: skrzynka email.
- Excel: ręczna ewidencja i śledzenie statusu.
- JIRA: zgłoszenia Complaint i Correction.
- SAP: dane zamówień i partii.

## Główne Commands

- Send Complaint Email.
- Read Complaint Email.
- Copy Complaint Data To Excel.
- Classify Defect.
- Create JIRA Complaint.
- Check Order And Batch In SAP.
- Reply To Customer.
- Create JIRA Correction.

## Główne Domain Events

- Complaint Email Sent.
- Complaint Email Received.
- Complaint Email Read.
- Complaint Data Copied To Excel.
- Defect Classified Manually.
- JIRA Complaint Created.
- SAP Data Checked.
- Customer Reply Sent.
- Correction Ticket Created.

## Policies / Business Rules

- Jeżeli email wygląda jak reklamacja, specjalista przepisuje dane do Excela.
- Jeżeli numer zamówienia lub opis jest niepełny, specjalista prosi klienta o uzupełnienie.
- Jeżeli dane SAP potwierdzają zamówienie i partię, specjalista kontynuuje obsługę.
- Jeżeli defekt jest potwierdzony, tworzony jest JIRA Correction dla Quality Team.

## Pain Points

- Brak automatycznego wykrywania nowych reklamacji.
- Ryzyko spamu lub późnego odczytu.
- Manualne kopiowanie do Excela.
- Manualna i niespójna klasyfikacja.
- Brak integracji SAP i JIRA.
- Brak spójnego audytu i metryk procesu.
- Backlog rośnie w okresach szczytu.

## AS-IS Flowchart Explanation

Flowchart [diagrams/metabot-as-is.drawio](../diagrams/metabot-as-is.drawio) pokazuje obecny, manualny przebieg obsługi reklamacji od emaila klienta do odpowiedzi końcowej. Nie opisuje procesu docelowego ani automatyzacji; pokazuje stan aktualny z ręczną obsługą emaila, Excela, JIRA i SAP.

Manualne są kluczowe kroki operacyjne: odczyt emaila przez Service Specialist, przepisanie danych do `Rejestr Reklamacji 2026.xlsx`, klasyfikacja defektu, utworzenie JIRA Complaint, sprawdzenie zamówienia i partii w SAP oraz utworzenie JIRA Correction po potwierdzeniu defektu.

Główne bottlenecki są widoczne w dwóch hotspotach: email może trafić do spamu albo zostać przeczytany z opóźnieniem, a klasyfikacja defektu jest subiektywna i niespójna. Dodatkowo Excel pełni rolę ręcznego rejestru, więc proces zależy od przepisywania danych i dyscypliny specjalistów.

Proces nie skaluje się dobrze, ponieważ każdy wzrost liczby reklamacji zwiększa liczbę ręcznych odczytów, kopiowań, klasyfikacji i sprawdzeń w systemach. W peakach tworzy to backlog, opóźnia odpowiedzi do klientów i utrudnia spójne raportowanie.

To dobry obszar do automatyzacji, ponieważ ma powtarzalny przepływ, ustrukturyzowane punkty decyzyjne, znane systemy źródłowe oraz wyraźne miejsca strat czasu. Automatyzacja powinna jednak wynikać z tych pain pointów, a nie zastępować oceny człowieka w decyzjach reklamacyjnych.
