# European Threat Detection (ETD) V2.7.11

## Aktualny stan frontendu i wdrożenia

ETD V2.7.11 to produkcyjny frontend projektu European Threat Detection.

Dokument łączy informacje z wcześniejszych README dotyczących bezpieczeństwa, prywatności, Online Now, mapy, źródeł, kategorii oraz wdrożenia. Dokumenty testowe zostały scalone tylko w zakresie funkcji, które faktycznie należą do aktualnego frontendu produkcyjnego.

> **Uwaga:** osobny test tła graficznego `.dashboard-hero` nie jest traktowany jako część wdrożenia produkcyjnego. Był oznaczony jako `TEST / NIE WDRAŻAĆ NA PRODUKCJĘ`.

---

## 1. Aktualna wersja

**Frontend:** ETD V2.7.11  
**Repozytorium:** `whitebearofthenorth-ai/European-Threat-Detection`  
**Gałąź:** `main`  
**Plik główny:** `index.html`

Aktualna korekta produkcyjna dotyczy mechanizmu **Online Now**.

Frontend korzysta z kontrolowanego RPC:

```text
supabaseClient.rpc('touch_active_visitor', {
    p_session_id: onlineSessionId
})
```

Stary RPC:

```text
heartbeat()
```

nie jest używany.

---

## 2. Online Now

Mechanizm Online Now korzysta z kontrolowanej funkcji:

```text
touch_active_visitor(uuid)
```

Funkcja działa jako `SECURITY DEFINER` i zapisuje `last_seen` po stronie serwera.

Odczyt liczby aktywnych użytkowników odbywa się przez:

```text
get_active_visitors()
```

Sesja użytkownika:

- wykorzystuje losowy UUID,
- jest przechowywana w `sessionStorage`,
- jest odświeżana przez heartbeat frontendu co około 30 sekund,
- nie jest bezpośrednio zapisywana przez rolę `anon` do tabeli.

### Czyszczenie sesji

Nieaktywne rekordy usuwa:

```text
cleanup_active_visitors()
```

Warunek:

```text
last_seen <= now() - interval '90 seconds'
```

Automatyczne czyszczenie wykonuje `pg_cron` co minutę.

Job:

```text
etd_cleanup_active_visitors
```

---

## 3. Bezpieczeństwo `active_visitors`

Rola `anon` nie posiada bezpośrednich uprawnień do tabeli `active_visitors`.

Dostęp odbywa się przez kontrolowane funkcje RPC.

Zweryfikowano:

- `touch_active_visitor(uuid)` jako `SECURITY DEFINER`,
- `get_active_visitors()`,
- brak bezpośrednich uprawnień `anon` do `active_visitors`,
- automatyczne czyszczenie przez `pg_cron`.

Brak polityk RLS dla `active_visitors` jest celowy, ponieważ tabela nie jest bezpośrednio dostępna dla `anon`.

---

## 4. Zakres danych

### Mapa

Mapa pobiera alerty z ostatnich:

```text
90 dni
```

Starsze alerty pozostają dostępne w archiwum.

### Sidebar

Panel:

**OSTATNIE ALERTY**

pokazuje wyłącznie alerty z ostatnich:

```text
48 godzin
```

Mapa zachowuje jednocześnie starsze alerty mieszczące się w zakresie 90 dni.

### Archiwum

Archiwum zapewnia dostęp do starszych danych oraz obsługuje:

- paginację,
- wyszukiwanie,
- filtrowanie,
- sortowanie według czasu zdarzenia.

---

## 5. Źródła informacji

Aktualny zestaw sześciu źródeł produkcyjnych:

1. Tagesschau
2. ANSA
3. BBC
4. VRT NWS
5. Sky News
6. France24

Mapowanie źródła w interfejsie:

```text
france24.com -> France24
```

Nie są używane:

- Der Standard,
- WP,
- Le Monde.

---

## 6. Kategorie alertów

Klucze kategorii w bazie danych i backendzie pozostają bez zmian.

Zmianie ulegają wyłącznie etykiety prezentowane użytkownikowi.

| Klucz | Etykieta w interfejsie |
|---|---|
| `terror` | Terroryzm |
| `bomb` | Zagrożenie bombowe |
| `weapon` | Broń / Atak z użyciem broni |
| `knife` | Atak z użyciem noża / narzędzia niebezpiecznego |
| `assault` | Napaść / Przemoc |
| `infrastructure` | Zagrożenie infrastruktury krytycznej |

Zmiana jest wyłącznie prezentacyjna i nie zmienia wartości przechowywanych w bazie.

---

## 7. Bezpieczeństwo frontendu

### Content Security Policy

Strona posiada CSP realizowane przez meta tag.

Ograniczenia obejmują m.in.:

- skrypty,
- CSS,
- obrazy,
- połączenia sieciowe,
- ramki,
- formularze,
- fonty,
- obiekty.

Uwzględniono wymagane usługi:

- Supabase,
- Web3Forms,
- hCaptcha,
- jsDelivr,
- CARTO.

Dodatkowo:

```text
object-src 'none'
base-uri 'self'
form-action 'self' https://api.web3forms.com
upgrade-insecure-requests
```

Dodano również:

```text
strict-origin-when-cross-origin
```

jako Referrer Policy.

---

## 8. Formularze i ochrona antyspamowa

Formularz kontaktowy oraz formularz współpracy korzystają z kilku warstw ochrony.

### Honeypot

Oba formularze zawierają ukryte pole:

```text
botcheck
```

Wypełnienie pola przez automat powoduje odrzucenie wysyłki.

### hCaptcha

Frontend:

- wymaga poprawnej odpowiedzi hCaptcha,
- blokuje wysyłkę bez weryfikacji,
- resetuje hCaptcha po obsłużeniu formularza.

### Web3Forms

Formularze są wysyłane przez HTTPS:

```text
https://api.web3forms.com/submit
```

---

## 9. Polityka prywatności i RODO

Na stronie znajduje się sekcja:

**Polityka prywatności**

Obejmuje ona m.in.:

- administratora danych,
- mechanizm Online Now,
- identyfikator sesji `sessionStorage`,
- formularz kontaktowy,
- formularz współpracy,
- Web3Forms,
- Supabase,
- jsDelivr,
- CARTO / OpenStreetMap,
- hCaptcha,
- GitHub Pages,
- podstawę prawną,
- prawa użytkownika,
- możliwość złożenia skargi do UODO.

Przy formularzach znajduje się również odsyłacz do polityki prywatności.

---

## 10. Leaflet i mapa

Leaflet 1.9.4 jest ładowany przez:

```text
cdn.jsdelivr.net
```

Nie jest używany `unpkg.com` jako źródło biblioteki Leaflet.

Mapa korzysta z podkładu:

```text
CARTO Voyager
```

z wymaganym kluczem CARTO.

Atrybucja obejmuje:

- OpenStreetMap,
- CARTO.

---

## 11. Realtime i synchronizacja danych

Frontend zachowuje architekturę ETD V2.7.11 opartą o:

- Supabase,
- Supabase Realtime,
- identyfikację alertów przez `guid`,
- fallback przez `id`,
- pełne pola danych alertów,
- aktualizację istniejących zdarzeń,
- synchronizację mapy, sidebaru i archiwum.

Obsługiwane są zdarzenia Realtime:

- `INSERT`,
- `UPDATE`,
- `DELETE`.

---

## 12. Bezpieczne linki źródłowe

Adresy źródłowe są walidowane przed wykorzystaniem w linkach.

Akceptowane są wyłącznie schematy:

```text
http:
https:
```

Pozostałe schematy są odrzucane.

Linki zewnętrzne wykorzystują:

```html
target="_blank"
rel="noopener noreferrer"
```

---

## 13. SEO i Open Graph

Frontend zawiera uporządkowane metadane SEO.

### Canonical

```text
https://whitebearofthenorth-ai.github.io/European-Threat-Detection/
```

### Open Graph

Uwzględniono m.in.:

- `og:url`
- `og:site_name`
- `og:locale`
- `og:image`
- `og:image:width`
- `og:image:height`
- `og:image:alt`

### Twitter Card

Używany jest:

```text
summary_large_image
```

z odpowiednią grafiką i opisem.

---

## 14. Grafika Open Graph

W repozytorium znajduje się:

```text
og-image.jpg
```

Grafika jest wykorzystywana przez:

- Open Graph,
- Twitter Card.

---

## 15. Wdrożenie

Wcześniejszy pełny pakiet V2.7.11 obejmował:

```text
index.html
og-image.jpg
README-CATEGORY-LABELS-TEST.txt
```

Aktualna korekta `index.html` zmienia frontendowy RPC Online Now z:

```text
heartbeat()
```

na:

```text
touch_active_visitor(uuid)
```

Jest to zgodne z utwardzonym mechanizmem Supabase i stanem bazy.

---

## 16. Kontrola po wdrożeniu

Po publikacji przez GitHub Pages należy sprawdzić:

1. `Frontend: V2.7.11`
2. `Online teraz: 1` w jednej sesji
3. `Online teraz: 2` w dwóch niezależnych sesjach
4. po zamknięciu jednej sesji licznik powinien spaść po około 90 sekundach
5. brak błędów związanych z `heartbeat()`
6. konsola powinna pokazywać poprawne działanie Online Now
7. mapa, sidebar, archiwum i statystyki powinny działać bez zmian

---

## 17. Punkty kontrolne

### Wcześniejszy pełny deploy V2.7.11

```text
835a4d8
Deploy ETD V2.7.11 frontend security privacy and SEO updates
```

### Aktualna korekta

```text
Online Now:
heartbeat() -> touch_active_visitor(uuid)
```

Po wykonaniu bieżącego commita należy zaktualizować poniżej numer najnowszego commita produkcyjnego.

---

## 18. Status

**ETD V2.7.11 FRONTEND**

Zakres obejmuje:

- bezpieczeństwo,
- prywatność,
- ochronę formularzy,
- SEO,
- Open Graph,
- źródła RSS,
- kategorie,
- mapę,
- sidebar,
- Online Now,
- Supabase,
- CARTO,
- Realtime.

Dokument ten zastępuje wcześniejsze, rozproszone README dotyczące tych elementów.
