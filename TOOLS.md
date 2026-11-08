# Zestaw narzędzi do testów bezpieczeństwa aplikacji webowej

Lista narzędzi pasujących do kolejnych obszarów szablonu [`webapp-security-test-plan-template.md`](webapp-security-test-plan-template.md). Dla każdego obszaru wskazuje, czego użyć i do czego, tak aby od razu było wiadomo, jak wykonać dany wiersz checklisty.

Wszystkie narzędzia są darmowe i działają lokalnie. Używaj ich **wyłącznie na własnej lub lokalnej instancji aplikacji albo za pisemną zgodą właściciela** (zasady w README i w szablonie).

## Zestaw minimum

Jeśli masz mało czasu, te sześć pozycji pokrywa większość testów z sekcji 1–6 i 7:

1. **Docker + Docker Compose**: uruchomienie aplikacji lokalnie.
2. **Mailpit**: przechwytywanie maili z magic linkami.
3. **Burp Suite Community** lub **OWASP ZAP**: proxy, Repeater, podmiana żądań.
4. **Python 3** (`httpx`, `requests`, `asyncio`): skrypty do wyścigów i powtarzania żądań.
5. **jwt_tool** lub **jwt.io**: manipulacja tokenami JWT.
6. **gitleaks, bandit, pip-audit**: sekrety, kod i zależności.

## 0. Środowisko laboratoryjne

Przed wszystkim innym przygotuj bezpieczne miejsce do testów.

- **Docker Compose**: uruchamia aplikację i bazę danych jednym poleceniem (`docker compose up --build`).
- **PostgreSQL (`psql` w kontenerze)**: podgląd stanu bazy po testach (np. czy nie powstały duplikaty).
- **Mailpit**: lokalny „łapacz” maili. Widzisz magic linki bez wysyłania prawdziwych wiadomości.
- **Przeglądarka z DevTools**: podgląd ciasteczek, `localStorage`, żądań i odpowiedzi.

## 1. Kontrola dostępu (wiersze 1.1–1.6)

Cel: sprawdzić, czy każda rola widzi i może zmieniać wyłącznie to, do czego ma prawo.

- **Burp Repeater**: ponawianie żądań z tokenem innej roli lub bez tokenu.
- **Autorize** (rozszerzenie Burpa): automatycznie porównuje odpowiedzi dla różnych ról.
- **Swagger UI** (`/docs`) i **curl**: szybkie bezpośrednie wywołania API z pominięciem frontendu.
- **Postman** lub **Bruno**: zapisane kolekcje żądań dla kont testowych (administrator A/B, użytkownik 1/2).

## 2. Uwierzytelnianie – magic link (wiersze 2.1–2.9)

Cel: sprawdzić, czy link logowania jest jednorazowy, wygasa i nie da się go nadużyć.

- **Mailpit**: odbiór linków i podgląd ich treści (m.in. czy zawiera podmieniony `Host`).
- **Burp** i **curl**: zmiana nagłówków (`Host`, `X-Forwarded-Host`), ponowne użycie tokenu.
- **Python (`asyncio` + `httpx`)**: równoległe użycie tego samego linku, pomiar czasu odpowiedzi (enumeracja kont), seria żądań do sprawdzenia rate limitingu.

## 3. Zaproszenia i rejestracja (wiersze 3.1–3.6)

Cel: sprawdzić, czy kody zaproszeń są nieprzewidywalne, wygasają i nie pozwalają na podniesienie roli.

- **Burp Repeater**: podmiana pól w body rejestracji (np. `role`), ponowne użycie kodu.
- **Python** (`secrets` do porównań, własny skrypt do równoległych żądań): analiza losowości kilku kodów i wyścig na limicie użyć.
- **curl**: szybkie wywołanie endpointów zaproszeń kontem o niższej roli.

## 4. Tokeny JWT i sesja (wiersze 4.1–4.7)

Cel: sprawdzić, czy token jest poprawnie podpisany, wygasa i traci ważność tam, gdzie powinien.

- **jwt_tool**: automatyczne testy (`alg: none`, zmiana algorytmu, słaby sekret).
- **jwt.io**: szybkie podejrzenie payloadu i claimów (`exp`, rola, identyfikator).
- **PyJWT**: własne tokeny podpisane znanym sekretem w razie wycieku.
- **DevTools → Application → Storage**: gdzie przechowywany jest token (`localStorage` czy ciasteczko HttpOnly).

## 5. Walidacja danych i wstrzykiwanie (wiersze 5.1–5.5)

Cel: sprawdzić, jak aplikacja zachowuje się przy niepoprawnych, skrajnych i złośliwych danych.

- **Burp Repeater/Intruder**: wysyłanie payloadów XSS, wartości skrajnych i dodatkowych pól.
- **sqlmap**: automatyczne sprawdzenie parametrów pod kątem SQLi (tylko lokalnie, z łagodnymi ustawieniami).
- **DevTools**: sprawdzenie, czy dane są renderowane jako tekst, czy jako HTML.
- **LibreOffice/Excel** i **openpyxl**: otwarcie wyeksportowanego pliku i sprawdzenie, czy pola zaczynające się od `=`, `+`, `-`, `@` nie są interpretowane jako formuły.

## 6. Logika biznesowa i wyścigi (wiersze 6.1–6.7)

Cel: sprawdzić, czy reguły (jedno zgłoszenie, limit miejsc, kolejność kroków) wytrzymują równoległe i niestandardowe użycie.

- **Python (`asyncio` + `httpx`)**: wiele żądań w tej samej chwili.
- **Turbo Intruder** (rozszerzenie Burpa): wysyłanie serii żądań z minimalnym opóźnieniem.
- **Burp Repeater (grupa żądań)**: równoległe wysłanie kilku żądań naraz.
- **`psql`**: sprawdzenie w bazie, czy stan po teście jest spójny.

## 7. Konfiguracja, sekrety i zależności (wiersze 7.1–7.8)

Cel: sprawdzić, czy konfiguracja, repozytorium i zależności nie wprowadzają ryzyka.

- **gitleaks** lub **trufflehog**: sekrety w repozytorium i jego historii.
- **bandit**: analiza statyczna kodu Pythona.
- **pip-audit** i `npm audit`: znane podatności w zależnościach.
- **trivy**: skan obrazów Dockera i konfiguracji.
- **nmap** (tylko lokalnie): które porty są wystawione (np. PostgreSQL).
- **curl -I** lub Burp: nagłówki bezpieczeństwa i CORS.

## 8. Obsługa błędów i logowanie (wiersze 8.1–8.4)

Cel: sprawdzić, czy błędy nie ujawniają szczegółów, a logi nie zawierają danych wrażliwych.

- **`docker compose logs`** i **`grep`**: przegląd logów pod kątem tokenów, JWT, adresów e-mail i stack trace'ów.
- **Burp/curl**: wywoływanie błędów (złe typy, brakujące pola) i ocena odpowiedzi.
- **Ręczne wyłączenie zależności** (np. kontenera bazy lub poczty) i obserwacja, czy system „zawodzi bezpiecznie”.

## 9. Pozostałe kategorie (wiersze 9.1–9.2)

Cel: pokryć tematy spoza głównych obszarów.

- **Prosty lokalny serwer** (`python -m http.server`): sprawdzenie, czy funkcja pobierająca zasoby z adresów podanych przez użytkownika odpytuje wskazany adres (SSRF).
- **Przegląd kodu HTML** i narzędzia deweloperskie: czy skrypty i style z zewnętrznych źródeł mają atrybut SRI.

## Dokumentowanie wyników

- **Notatki:** wypełniony lokalnie plik szablonu (Markdown) lub Obsidian.
- **Dowody:** zrzuty ekranu, zapisane żądania z Burpa albo polecenia `curl` zapisane do pliku.
- **Przechowywanie:** w repozytorium **prywatnym**; wypełnionych wyników nie umieszczaj w publicznym repozytorium, dopóki wykryte problemy nie zostaną naprawione.

## Uwagi praktyczne

- W Burp Community Intruder jest spowolniony, ale Repeater i Turbo Intruder wystarczają do testów wyścigów na tej skali. ZAP ma zbliżone możliwości.
- Narzędzia typu `sqlmap` i `nmap` uruchamiaj wyłącznie na własnej, lokalnej instancji.
- Wersje narzędzi i sposób instalacji sprawdzaj w ich oficjalnych repozytoriach, bo zmieniają się z czasem.
