# Web Application Security Review Framework & Template

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OWASP Top 10](https://img.shields.io/badge/OWASP-Top%2010%202025-blue)](https://owasp.org/Top10/2025/)
[![OWASP API Top 10](https://img.shields.io/badge/OWASP-API%20Top%2010%202023-orange)](https://owasp.org/API-Security/)

Uniwersalna, zanonimizowana checklista do przeprowadzania audytów i testów bezpieczeństwa aplikacji webowych oraz interfejsów REST API.

## O projekcie

Szablon powstał podczas audytu jednego z projektów koła naukowego, aby umożliwić sprawny podział zadań między wieloma audytorami oraz ustrukturyzować przeprowadzanie testów. 

Ze względu na zasady odpowiedzialnego ujawniania podatności oraz dbałość o bezpieczeństwo audytowanego serwisu, rzeczywiste wyniki i zidentyfikowane problemy zostały wyłączone z dokumentu. 
Repozytorium udostępnia wyłącznie uniwersalną metodykę badawczą opartą o standardy OWASP i CWE.

Projekt ma charakter otwarty. W miarę potrzeb planowane jest dodawanie osobnych szablonów dedykowanych dla innych typów architektur oraz konkretnych modułów funkcjonalnych.

## Cel projektu

- Ustrukturyzowanie procesu audytu bezpieczeństwa warstwy aplikacji i API.
- Mapowanie scenariuszy testowych bezpośrednio do uznanych standardów branżowych (**OWASP Top 10:2025**, **OWASP API Security Top 10:2023**, **CWE**).
- Stworzenie powtarzalnej metodyki do wielokrotnego użytku przy kolejnych projektach.

---

## Zawartość repozytorium i planowany rozwój

### Dostępne szablony

- [`webapp-security-test-plan-template.md`](./webapp-security-test-plan-template.md) – Szablon testów bezpieczeństwa dla aplikacji ze stosem REST API, logowaniem magic link i tokenami JWT (9 obszarów testowych).

### Planowane moduły

- `file-upload-security-template.md` – Szablon do testowania mechanizmów wgrywania, walidacji i przechowywania plików.
- `oauth2-oidc-template.md` – Testy integracji SSO oraz zewnętrznych dostawców tożsamości.

---

## Jak używać szablonu

1. Skopiuj plik szablonu (np. `webapp-security-test-plan-template.md`) do prywatnego repozytorium testowanego projektu.
2. Skonfiguruj lokalne środowisko testowe (np. w oparciu o Docker i narzędzia do przechwytywania wiadomości e-mail, jak Mailpit).
3. Przeprowadzaj testy punkt po punkcie, uzupełniając kolumny `Status`, `Wynik`, `Dowód` oraz `Priorytet`.
4. Wykorzystaj zawarte odnośniki **CWE / OWASP** do przygotowania zaleceń naprawczych dla zespołu deweloperskiego.

---

## Etyka i odpowiedzialność

Ten szablon służy wyłącznie autoryzowanym testom bezpieczeństwa oraz celom edukacyjnym.

- **Zasada pisemnej zgody:** Testuj wyłącznie systemy, do których posiadasz prawo własności lub wyraźną zgody właściciela na przeprowadzenie audytu.
- **Poufność danych:** Wypełnione wyniki testów, szczegóły podatności oraz logi konkretnych aplikacji nie mogą być publikowane w publicznych repozytoriach.

---

## Licencja

Projekt udostępniany jest na licencji [MIT License](LICENSE).
