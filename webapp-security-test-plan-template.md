# Szablon planu testów bezpieczeństwa aplikacji webowej

Pusty szablon (checklista) do przeglądu bezpieczeństwa aplikacji webowej z REST API, rolami użytkowników, logowaniem przez magic link i tokenami JWT. Zawiera **metodykę, a nie wyniki**: kolumny na status, wynik, dowód i priorytet są puste i przeznaczone do wypełnienia podczas własnych testów.

> **Zakres i etyka.** Testuj wyłącznie systemy własne, lokalne instancje lub takie, na które masz **pisemną zgodę właściciela**. Wyniki dotyczące cudzych systemów traktuj jako poufne i nie publikuj ich bez zgody. Szablon jest celowo ogólny i nie opisuje żadnej konkretnej aplikacji. Wypełnione wyniki testów trzymaj poza publicznym repozytorium, dopóki wykryte problemy nie zostaną naprawione.

## Jak używać

1. Uruchom aplikację lokalnie (np. `docker-compose up --build`), a dla maili użyj lokalnego łapacza (np. Mailpit).
2. Przygotuj konta testowe: co najmniej dwóch administratorów (A i B) oraz dwóch zwykłych użytkowników (1 i 2).
3. Przechodź po kolejnych punktach, zapisując w tabeli **co się stało** i **dowód** (zrzut ekranu, fragment żądania, log).
4. Ocenę priorytetu opieraj na **wpływie** i **łatwości wykorzystania**, a także na tym, czy problem występuje na wdrożonej instancji.
5. Wynik „zachowanie poprawne, brak podatności” jest pełnoprawnym wynikiem i też warto go zapisać.
6. Sprawdź, gdzie aplikacja trzyma token sesji (ciasteczko czy nagłówek), i dostosuj do tego sposób wywoływania API w testach.

**Status:** `OK` · `PODATNOŚĆ` · `UWAGA` (do omówienia) · `N/D` (nie dotyczy lub nie wykonano)
**Priorytet:** krytyczny · wysoki · średni · niski · informacyjny

## Legenda odniesień

Kolumna „Odniesienia” wskazuje powiązane kategorie, w formacie `Top 10:2025 · API Top 10:2023 · CWE`.

| Kod | Znaczenie |
|---|---|
| A01 | Broken Access Control |
| A02 | Security Misconfiguration |
| A03 | Software Supply Chain Failures |
| A04 | Cryptographic Failures |
| A05 | Injection |
| A06 | Insecure Design |
| A07 | Authentication Failures |
| A08 | Software or Data Integrity Failures |
| A09 | Security Logging & Alerting Failures |
| A10 | Mishandling of Exceptional Conditions |
| API1 | Broken Object Level Authorization |
| API2 | Broken Authentication |
| API3 | Broken Object Property Level Authorization |
| API4 | Unrestricted Resource Consumption |
| API5 | Broken Function Level Authorization |
| API6 | Unrestricted Access to Sensitive Business Flows |
| API7 | Server Side Request Forgery |
| API8 | Security Misconfiguration |
| API9 | Improper Inventory Management |
| CWE-xxx | Wpis w słowniku MITRE CWE |

Kategorie `A01–A10` pochodzą z OWASP Top 10:2025, a `API1–API10` z OWASP API Security Top 10:2023 (w legendzie tylko używane kategorie).

## 1. Kontrola dostępu

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 1.1 | Wywołaj endpointy administracyjne bez tokenu i z tokenem zwykłego użytkownika (oczekiwane 401/403). | A01 · API5 · CWE-862 | | | | |
| 1.2 | Podmień identyfikator zasobu w ścieżce lub body na ID należące do innego użytkownika lub administratora (IDOR) – dla odczytu i modyfikacji. | A01 · API1 · CWE-639 | | | | |
| 1.3 | Sprawdź, czy widoki typu „moje dane” zwracają wyłącznie dane zalogowanego użytkownika. | A01 · API1 · CWE-639 | | | | |
| 1.4 | Sprawdź osobno autoryzację operacji wrażliwych (rozstrzyganie, eksport, usuwanie) – nie tylko odczytu. | A01 · API5 · CWE-285 | | | | |
| 1.5 | Wywołaj API bezpośrednio (curl, Burp), omijając interfejs – uprawnienia muszą być egzekwowane po stronie serwera. | A01 · CWE-602 | | | | |
| 1.6 | Dodaj pola uprzywilejowane (np. `role`) do body rejestracji lub edycji (mass assignment). | A01 · API3 · CWE-915 | | | | |

## 2. Uwierzytelnianie – magic link

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 2.1 | Logowanie adresem spoza dozwolonej domeny oraz obejścia (dopisek do domeny, wielkie litery, plus-addressing, białe znaki). | A07 · API2 · CWE-287 | | | | |
| 2.2 | Użyj tego samego linku logowania dwa razy, kolejno (token replay). | A07 · API2 · CWE-294 | | | | |
| 2.3 | Link po upływie czasu ważności; pusty, losowy i zmodyfikowany token. | A07 · API2 · CWE-613 | | | | |
| 2.4 | Entropia tokenu: wygeneruj kilka i porównaj długość oraz losowość. | A07 · A04 · CWE-330 | | | | |
| 2.5 | Host header: czy link w mailu odzwierciedla podmieniony nagłówek `Host` lub `X-Forwarded-Host`. | A07 · A02 · API8 · CWE-640 | | | | |
| 2.6 | Enumeracja kont: porównaj treść i czas odpowiedzi dla istniejącego i nieistniejącego adresu. | A07 · CWE-204 · CWE-208 | | | | |
| 2.7 | Rate limiting na endpointach logowania i rejestracji oraz możliwość zapchania kolejki wysyłki maili lub zalewania cudzej skrzynki. | A07 · API4 · CWE-307 | | | | |
| 2.8 | Równoległe użycie tego samego linku logowania (wyścig między sprawdzeniem a unieważnieniem tokenu). | A07 · API2 · CWE-367 | | | | |
| 2.9 | Czy operacja zużywająca token odbywa się na żądaniu GET (prefetch skanerów i klientów poczty), czy token w URL trafia do logów serwera i proxy oraz czy wylogowanie przez GET jest podatne na wymuszenie. | A07 · A01 · CWE-598 · CWE-352 | | | | |

## 3. Zaproszenia i rejestracja

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 3.1 | Ponowne użycie kodu zaproszenia oraz jego wygasanie; sprawdź, które operacje zużywają limit użyć (czy także logowanie istniejących kont). | A06 · A07 · CWE-362 | | | | |
| 3.2 | Nieprzewidywalność kodu: wygeneruj kilka obok siebie i porównaj. | A04 · A07 · CWE-330 | | | | |
| 3.3 | Podmiana roli w body rejestracji przy zaproszeniu na niższą rolę. | A01 · API3 · CWE-269 | | | | |
| 3.4 | Zwykły użytkownik nie może generować zaproszeń (oczekiwane 403). | A01 · API5 · CWE-862 | | | | |
| 3.5 | Wyścig na limicie użyć zaproszenia: równoległe żądania z kodem o limicie 1. | A06 · API6 · CWE-362 | | | | |
| 3.6 | Domyślne lub seedowane tokeny i zaproszenia: wartość, limit użyć, termin ważności, rola oraz czy nie wyciekają w logach lub odpowiedziach. | A07 · A02 · CWE-798 · CWE-1188 | | | | |

## 4. Tokeny JWT i sesja

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 4.1 | Zmień claimy (rola, identyfikator) w payloadzie – podpis musi przestać być ważny. | A07 · API2 · CWE-347 | | | | |
| 4.2 | Test `alg: none` oraz zmiany algorytmu podpisu. | A07 · API2 · CWE-347 | | | | |
| 4.3 | Siła i ochrona sekretu podpisu (długość, wartości przykładowe, brak w repo i w odpowiedziach API); podpis własnego tokenu w razie wycieku. | A04 · A02 · CWE-326 · CWE-798 | | | | |
| 4.4 | Token po upływie `exp` oraz ocena długości życia tokenu. | A07 · CWE-613 | | | | |
| 4.5 | Token po usunięciu użytkownika lub zmianie jego roli (brak unieważniania przy tokenach bezstanowych). | A07 · CWE-613 | | | | |
| 4.6 | Przechowywanie tokenu we froncie (localStorage vs ciasteczko HttpOnly) i konsekwencje przy XSS. | A07 · A02 · CWE-922 | | | | |
| 4.7 | Token po wylogowaniu: czy skopiowany wcześniej token nadal działa do czasu wygaśnięcia. | A07 · CWE-613 | | | | |

## 5. Walidacja danych i wstrzykiwanie

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 5.1 | XSS (np. `<img src=x onerror=alert(1)>`) w polach wyświetlanych innym użytkownikom – czy front renderuje to jako tekst. | A05 · CWE-79 | | | | |
| 5.2 | Podstawowe payloady SQLi w parametrach oraz przegląd kodu pod kątem zapytań surowych. | A05 · CWE-89 | | | | |
| 5.3 | Wartości skrajne: ujemne i zerowe liczby, bardzo długie ciągi, dodatkowe pola, błędne typy, brak górnych limitów liczby tworzonych zasobów, duplikaty elementów w listach. | A06 · A10 · API4 · CWE-20 · CWE-770 | | | | |
| 5.4 | Formula injection w eksportach CSV/XLSX: pola wpisywane przez użytkowników, które trafiają do eksportu, a zaczynają się od `=`, `+`, `-`, `@`. | A05 · CWE-1236 | | | | |
| 5.5 | Dane spoza kontekstu (zasób z innej jednostki, wartość poza dozwolonym oknem czasowym). | A06 · CWE-20 | | | | |

## 6. Logika biznesowa i wyścigi

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 6.1 | Reguła „jedno zgłoszenie na jednostkę”: równoległe żądania (skrypt `asyncio`, Burp Turbo Intruder) oraz ponowne zgłoszenie po pierwszym. | A06 · API6 · CWE-362 | | | | |
| 6.2 | Limit zasobu (np. liczba miejsc): równoczesne zapisy przez wielu użytkowników. | A06 · CWE-362 | | | | |
| 6.3 | Idempotentność operacji rozstrzygających: powtórne i równoległe wywołanie. | A06 · CWE-362 | | | | |
| 6.4 | Kolejność kroków: operacje wykonywane przed czasem (np. rozstrzygnięcie przed końcem okna). | A06 · API6 · CWE-841 | | | | |
| 6.5 | Eksport lub raport przed gotowością danych – czytelny błąd zamiast pustego lub uszkodzonego pliku. | A06 · A10 · CWE-755 | | | | |
| 6.6 | Zbiory puste i przypadki brzegowe w statystykach (brak elementów, dzielenie przez zero) oraz poprawność liczników. | A10 · CWE-369 | | | | |
| 6.7 | Zmiana okna czasowego po rozstrzygnięciu: czy można dodać lub zmienić zgłoszenie, gdy wynik jest już ogłoszony. | A06 · API6 · CWE-841 | | | | |

## 7. Konfiguracja, sekrety i zależności

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 7.1 | Tryb debug i endpointy testowe: czy nie ujawniają sekretów ani zawartości bazy i nie wydają tokenów bez uwierzytelnienia; sprawdź wartości domyślne w plikach przykładowych (czy flaga debug nie jest włączona). | A02 · API8 · CWE-489 | | | | |
| 7.2 | Sekrety w repozytorium i historii commitów (`gitleaks`, `trufflehog`). | A02 · CWE-798 | | | | |
| 7.3 | Podatności w zależnościach (`pip-audit`, `npm audit`). | A03 · CWE-1395 | | | | |
| 7.4 | Kontenery i infrastruktura: wystawione porty bazy, domyślne hasła. | A02 · API8 · CWE-1188 | | | | |
| 7.5 | CORS i nagłówki bezpieczeństwa. | A02 · API8 · CWE-942 | | | | |
| 7.6 | Dokumentacja API (np. Swagger) na środowisku produkcyjnym: czy ujawnia endpointy wewnętrzne. | A02 · API9 | | | | |
| 7.7 | Analiza statyczna (`bandit` lub odpowiednik) oraz użytkownik, z którego działa kontener (root). | A02 · API8 · CWE-250 | | | | |
| 7.8 | Przy wyłączonym trybie debug endpointy debug nie są rejestrowane (oczekiwane 404). | A02 · API8 · CWE-489 | | | | |

## 8. Obsługa błędów i logowanie

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 8.1 | Komunikaty błędów nie ujawniają stack trace'ów, treści wyjątków (baza, poczta) ani szczegółów wewnętrznych; błędne dane wejściowe dają 4xx, a nie 500. | A10 · A02 · CWE-209 | | | | |
| 8.2 | Logi aplikacji, serwera i proxy (także przy błędach 500) nie zawierają tokenów (również w query stringu), JWT, sekretów ani danych osobowych. | A09 · CWE-532 | | | | |
| 8.3 | Zdarzenia bezpieczeństwa (nieudane logowania, odrzucone żądania, operacje administracyjne) są rejestrowane. | A09 · CWE-778 | | | | |
| 8.4 | Zachowanie przy awarii zależności (baza, poczta): system nie przechodzi w tryb „fail open”. | A10 · CWE-636 | | | | |

## 9. Pokrycie pozostałych kategorii

| Lp. | Scenariusz testu | Odniesienia | Status | Wynik | Dowód | Priorytet |
|---|---|---|---|---|---|---|
| 9.1 | Funkcje pobierające zasoby z adresów podanych przez użytkownika (SSRF). | A01 · API7 · CWE-918 | | | | |
| 9.2 | Integralność zasobów frontendu: skrypty i style ładowane z zewnętrznych źródeł (SRI), niezaufana deserializacja, niepodpisane aktualizacje. | A08 · A03 · CWE-829 · CWE-502 | | | | |

## Podsumowanie wyników (do wypełnienia)

| Liczba testów | OK | PODATNOŚĆ | UWAGA | N/D |
|---|---|---|---|---|
| | | | | |

**Najważniejsze wnioski i rekomendacje:**

-
-
-

## Źródła

- OWASP Top 10:2025 – <https://owasp.org/Top10/2025/>
- OWASP API Security Top 10 (edycja 2023) – <https://owasp.org/API-Security/>
- OWASP Web Security Testing Guide (WSTG) – <https://owasp.org/www-project-web-security-testing-guide/>
- OWASP Application Security Verification Standard (ASVS) – <https://owasp.org/www-project-application-security-verification-standard/>
- Słownik słabości CWE – <https://cwe.mitre.org/>
