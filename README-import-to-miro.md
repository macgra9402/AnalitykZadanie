# README - Import Event Storming Board To Miro

Ten pakiet pozwala odtworzyc tablice Event Storming dla Metabot / Metalpol w Miro.

## Pliki

- `miro-event-storming-sticky-notes.csv` - lista karteczek sticky notes z kategoriami, kolorami i wspolrzednymi.
- `miro-event-storming-layout.md` - opis ukladu tablicy i zasad czytania.
- `metabot-event-storming-board.drawio` - fallback wizualny w diagrams.net / draw.io.

## Jak uzyc CSV w Miro

1. Otworz Miro board.
2. Utworz pusta przestrzen na tablicy.
3. Zaimportuj CSV albo skopiuj wiersze z CSV do Miro jako sticky notes, zaleznie od dostepnego mechanizmu importu w Twojej wersji Miro.
4. Zachowaj pola `x` i `y` jako sugerowane wspolrzedne ukladu.
5. Ustaw kolory sticky notes wedlug kolumny `color` albo kategorii `category`.
6. Umiesc TO-BE po lewej stronie, AS-IS po prawej stronie.
7. Zostaw pionowy podzial miedzy diagramami.

## Zalecenia dla connectorow

Nie lacz wszystkich karteczek. Event Storming powinien pozostac warsztatowa mapa zdarzen, a nie flowchart.

Dodaj tylko pomocnicze laczenia tam, gdzie poprawiaja czytelnosc:
- glowne command -> event,
- event -> nastepny command,
- policy -> skutkujacy routing,
- hotspot -> krok, ktorego dotyczy.

## Fallback przez draw.io

Jesli import CSV do Miro nie zachowa ukladu lub kolorow, otworz `metabot-event-storming-board.drawio` w diagrams.net, wyeksportuj do SVG/PDF/PNG i wklej do Miro jako warstwe referencyjna.

To nie jest mind map ani flowchart. Uklad celowo zachowuje sticky-note Event Storming: kategorie, kolory, sekcje, hotspoty i oddzielone obszary TO-BE / AS-IS.
