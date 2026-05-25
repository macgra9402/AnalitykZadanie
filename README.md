# Metabot AI Complaint Automation — koncepcja rozwiązania

## Cel dokumentacji

Repozytorium przedstawia koncepcję usprawnienia procesu obsługi reklamacji dla Metabot / Metalpol. Materiały pokazują zrozumienie procesu **AS-IS**, kierunek procesu **TO-BE**, rolę automatyzacji AI, integracje z istniejącymi systemami, zakres MVP, ryzyka, metryki i główne decyzje projektowe.

Dokumentacja została przygotowana z perspektywy analityka automatyzacji AI. Koncentruje się na uporządkowaniu procesu, określeniu roli AI, integracji istniejących systemów, punktów kontroli człowieka, metryk oraz granic MVP. Nie jest to pełna specyfikacja wdrożeniowa ani implementacja techniczna.

## Zakres

Dokumentacja ma charakter procesowo-architektoniczny na poziomie analitycznym. Nie jest kompletną specyfikacją produkcyjnego wdrożenia ani pełnym backlogiem implementacyjnym.

Szczegóły takie jak pełny model danych, mapowanie pól JIRA, mapowanie odpowiedzi SAP, progi poziomu pewności AI, polityka ponowień, konfiguracja produkcyjna i harmonogram wdrożenia powinny zostać doprecyzowane w etapie discovery/warsztatowym.

## Struktura repozytorium

- [docs/business-overview.md](docs/business-overview.md) - biznesowy opis problemu, zmiany procesu i wartości rozwiązania.
- [docs/solution-specification.md](docs/solution-specification.md) - główna specyfikacja koncepcji rozwiązania: proces, architektura, integracje, AI, MVP, ryzyka i trade-offy.
- [diagrams/metabot-as-is.drawio](diagrams/metabot-as-is.drawio) - diagram AS-IS Event Storming obecnego procesu reklamacyjnego.
- [diagrams/metabot-to-be-main-flow.drawio](diagrams/metabot-to-be-main-flow.drawio) - diagram TO-BE Event Storming procesu docelowego.

## Jak czytać materiały

1. Zacznij od [business-overview.md](docs/business-overview.md), żeby zrozumieć problem biznesowy i wartość zmiany.
2. Następnie przejdź do [solution-specification.md](docs/solution-specification.md), gdzie opisano proces, integracje, architekturę, AI i zakres MVP.
3. Na końcu przejrzyj diagramy AS-IS i TO-BE, które pokazują przejście od ręcznej obsługi reklamacji do kontrolowanego procesu wspieranego automatyzacją.

## Najważniejsza zasada rozwiązania

AI porządkuje i sugeruje. Systemy źródłowe walidują fakty. Jawne reguły kierują procesem. Człowiek podejmuje decyzje w sprawach ryzykownych.

## Dalsze doprecyzowanie

Koncepcja powinna zostać rozwinięta podczas warsztatów biznesowo-procesowych i techniczno-integracyjnych. Ich celem byłoby potwierdzenie finalnego zakresu MVP, budżetu, zależności integracyjnych, szczegółowego backlogu i wymagań produkcyjnych.
