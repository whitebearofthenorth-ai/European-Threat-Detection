# ETD V2.7.11 - HERO BACKGROUND GRAPHIC TEST

## Cel testu

Dodano przesłaną grafikę ETD jako subtelne tło sekcji nagłówkowej dashboardu.

Grafika:
- działa wyłącznie jako warstwa wizualna,
- znajduje się w istniejącym `.dashboard-hero`,
- jest przyciemniona i częściowo wygaszona gradientem,
- wykorzystuje osobny plik `etd-hero-map-bg.jpg`, skupiony na mapie Europy,
- nie przykrywa tekstu interfejsu,
- nie wpływa na mapę Leaflet.

## Zachowane bez zmian

- alerty z ostatnich 48 godzin w bocznym panelu,
- pełny zestaw alertów na mapie,
- miejscowość + kraj,
- CARTO Voyager + API key,
- Leaflet 1.9.4,
- Supabase / Realtime,
- statystyki,
- archiwum / paginacja,
- Online Now,
- pozostała logika ETD V2.7.11,
- poprawki kosmetyczne z poprzedniego testu.

## Zakres zmian

Zmiana dotyczy wyłącznie warstwy wizualnej `.dashboard-hero` oraz dodania lokalnego assetu graficznego.

Nie zmieniano logiki JavaScript, zapytań Supabase ani mechanizmu Realtime.

## Test

**TEST / NIE WDRAŻAĆ NA PRODUKCJĘ**

Przed wdrożeniem sprawdzić:
1. wygląd dashboardu na desktopie,
2. czy tekst nagłówka pozostaje czytelny,
3. wygląd na węższym ekranie,
4. czy mapa, alerty, archiwum i statystyki działają jak wcześniej.
