ETD V2.7.11 - EVENT ORDER + TAG WIDTH FIX TEST

Baza:
index-v2.7.11-update-badge-about-refresh-test.html

Zmiany:
1. Ostatnie alerty są sortowane według czasu zdarzenia:
   pub_date DESC, następnie created_at DESC.
   updated_at NIE wpływa już na kolejność alertów.

2. Realtime INSERT/UPDATE również utrzymuje kolejność według czasu zdarzenia.
   UPDATE nadal zachowuje badge UPDATE oraz godzinę aktualizacji.

3. Archiwum sortuje rekordy według pub_date DESC, następnie created_at DESC.

4. Tagi kategorii mają szerokość wynikającą z tekstu + paddingu.
   Nie rozciągają się na pozostałą szerokość wiersza.

Zachowane:
- UPDATE badge i godzina aktualizacji
- Realtime UPDATE
- odświeżanie statystyk
- paginacja i filtrowanie archiwum
- Online teraz
- poprawiona sekcja Architektura Technologiczna
- pozostały interfejs bez zmian
