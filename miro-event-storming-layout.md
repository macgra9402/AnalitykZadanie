# Miro Event Storming Layout - Metabot / Metalpol

Ten plik opisuje uklad tablicy sticky-note Event Storming przygotowanej do odtworzenia w Miro.

## Model importu

Podstawowym formatem jest `miro-event-storming-sticky-notes.csv`. Kazdy wiersz reprezentuje jedna karteczke sticky note.

Kolumny:
- `diagram` - `TO-BE`, `AS-IS`, `LEGEND` albo `BOARD`.
- `section` - sekcja procesu.
- `category` - typ Event Storming.
- `label` - krotki tekst na karteczce.
- `x`, `y` - sugerowane wspolrzedne na tablicy.
- `width`, `height` - sugerowany rozmiar karteczki.
- `color` - kolor HEX zgodny z legenda.
- `notes` - komentarz pomocniczy, nie musi byc widoczny na karteczce.

## Uklad tablicy

- Legenda znajduje sie na gorze tablicy.
- TO-BE Event Storming jest po lewej stronie.
- AS-IS Event Storming jest po prawej stronie.
- Miedzy obszarami jest pionowy divider.
- Glowne strumienie zdarzen ida szeroko od lewej do prawej.
- Wyjatki, reconciliation i zasady architektury sa oddzielone od glownego strumienia TO-BE.

## Kolory

- ACTOR - zolty `#fff2a8`
- COMMAND - jasnoniebieski `#cfe8ff`
- EVENT - pomaranczowy `#ffc78f`
- SYSTEM / EXTERNAL SYSTEM - rozowy / magenta `#f7b0d8`
- POLICY / RULE - fioletowy `#d7c6ff`
- READ MODEL / DATA STORE - jasnozielony `#c9f2a7`
- HOTSPOT / RISK / PAIN POINT - lososiowy `#ffb3b3`
- SECTION HEADER - jasnoszary `#eeeeee`

## TO-BE sekcje

1. Email Intake
2. Registration & Storage
3. AI Automation
4. Customer & SAP Validation
5. Orchestration
6. JIRA & Human Review
7. Correction / Response / Metrics
8. Exception Handling
9. Mailbox Reconciliation
10. Architecture Principles

AI Automation jest celowo rozbita na kilka par command/event, zeby pokazac, ze AI nie jest magicznym botem, tylko etapem interpretacji danych.

## AS-IS sekcje

1. Email Intake
2. Manual Reading
3. Excel Registration
4. Manual Classification
5. JIRA Complaint
6. SAP Verification
7. Customer Response
8. Correction Path
9. Process Pain Points

AS-IS nie zawiera automatyzacji, AI, webhookow ani integracji technicznych. Pokazuje obecny reczny proces i bolesne miejsca.

## Zasady laczenia karteczek

W Miro nie trzeba laczyc wszystkiego. Zalecane sa tylko glowne laczenia w strumieniu zdarzen:
- command -> event,
- system blisko zdarzenia, ktore obsluguje,
- policy pod decyzja/routingiem,
- hotspot pod krokiem, ktorego dotyczy.

Unikaj nadmiaru connectorow. Styl ma przypominac warsztat Event Storming, nie diagram przeplywu technicznego.

## Widoczne notatki SAP w TO-BE

Przy SAP znajduje sie karta `SAP API Note`:

SAP ERP PP/QM read-only REST API:
- `GET /api/v1/orders/{id}`
- `GET /api/v1/batches/{id}`
- rate limit: `100 req/min`
- MVP does not modify SAP data

Ta notatka powinna byc widoczna obok karteczek SAP w Miro.
