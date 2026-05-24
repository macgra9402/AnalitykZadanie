# Metabot AI Automation Analyst - case study

Repozytorium zawiera dokumentację rozwiązania dla zadania rekrutacyjnego Metabot Sp. z o.o. To case study dokumentacyjne, a nie produkcyjna implementacja systemu.

Zakres opiera się wyłącznie na dostarczonym briefie: proces reklamacji producenta komponentów metalowych dla branży automotive, dostępne systemy Microsoft 365 / Exchange, SAP ERP PP/QM REST API, JIRA Cloud, PostgreSQL customer database oraz Azure Blob Storage.

## Struktura repozytorium

- [docs/01-business-context.md](docs/01-business-context.md) - kontekst biznesowy i problemy AS-IS.
- [docs/02-as-is-event-storming.md](docs/02-as-is-event-storming.md) - Event Storming obecnego procesu.
- [docs/03-to-be-event-storming.md](docs/03-to-be-event-storming.md) - Event Storming procesu docelowego.
- [docs/04-solution-specification.md](docs/04-solution-specification.md) - główna specyfikacja techniczno-biznesowa.
- [docs/05-ai-automation-flow.md](docs/05-ai-automation-flow.md) - logika AI automation i human-in-the-loop.
- [docs/06-integrations.md](docs/06-integrations.md) - integracje z systemami dostępnymi w briefie.
- [docs/07-risks-and-tradeoffs.md](docs/07-risks-and-tradeoffs.md) - ryzyka, ograniczenia i trade-offy.
- [docs/08-interview-preparation.md](docs/08-interview-preparation.md) - notatki Q&A do rozmowy technicznej.
- [diagrams/metabot-as-is.drawio](diagrams/metabot-as-is.drawio) - AS-IS Complaint Process.
- [diagrams/metabot-to-be-main-flow.drawio](diagrams/metabot-to-be-main-flow.drawio) - TO-BE Main Flow.
- [diagrams/metabot-exception-handling.drawio](diagrams/metabot-exception-handling.drawio) - obsługa wyjątków, retry i reconciliation.
- [diagrams/metabot-solution-architecture.drawio](diagrams/metabot-solution-architecture.drawio) - architektura rozwiązania.
- [diagrams/metabot-jira-ticket-and-metrics.drawio](diagrams/metabot-jira-ticket-and-metrics.drawio) - struktura JIRA Complaint i metryki.
- [miro-event-storming-sticky-notes.csv](miro-event-storming-sticky-notes.csv) - pakiet sticky notes do odtworzenia tablicy Event Storming w Miro.
- [miro-event-storming-layout.md](miro-event-storming-layout.md) - opis ukladu tablicy TO-BE / AS-IS i kolorow.
- [README-import-to-miro.md](README-import-to-miro.md) - instrukcja odtworzenia tablicy w Miro.
- [metabot-event-storming-board.drawio](metabot-event-storming-board.drawio) - fallback Event Storming w stylu sticky-note.
- [examples/ai-extraction-result.example.json](examples/ai-extraction-result.example.json) - przykładowy wynik ekstrakcji AI.

## Jak czytać dokumenty

Najpierw przeczytaj kontekst biznesowy, potem AS-IS i TO-BE Event Storming. Następnie przejdź do specyfikacji rozwiązania i przepływu AI automation. Dokumenty o integracjach, ryzykach i przygotowaniu do rozmowy są uzupełnieniem do obrony decyzji.

## Komponenty rozwiązania

Docelowa architektura rozdziela:

- Email Intake Service,
- Attachment Storage Service,
- AI Extraction and Classification Service,
- SAP Integration Adapter,
- JIRA Integration Adapter,
- Customer Database Adapter,
- Complaint Orchestrator,
- Operational Database,
- Monitoring and Reporting Layer,
- Human Review Workflow.

Komponenty korzystają tylko z systemów wskazanych w briefie: Microsoft 365 / Exchange / Microsoft Graph, SAP ERP PP/QM REST API, JIRA Cloud, PostgreSQL customer database i Azure Blob Storage.

## Najważniejsza zasada architektoniczna

Rozwiązanie projektuje AI-assisted complaint intake and triage, a nie w pełni autonomiczny system decyzji reklamacyjnych.

AI interpretuje treść, sugeruje kategorię i przygotowuje draft odpowiedzi. Walidacja źródeł prawdy, obsługa integracji, rate limit, decyzje wysokiego ryzyka, audyt i bezpieczeństwo pozostają deterministyczne oraz kontrolowane przez proces.
