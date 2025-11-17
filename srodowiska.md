## Środowiska KSeF API 2.0 
01.09.2025

Poniżej zestawiono informacje o publicznych środowiskach.

| Skrót   | Środowisko                       | Opis                                                                                                                                   | Adres (API)                                                                                           | Termin udostępnienia      | Dopuszczalne formaty faktur do wysyłki |
|---------|----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|---------------------------|---------------------|
| **TE**  | Testowe <br/> (Release Candidate)      | Środowisko RC służące wczesnej weryfikacji planowanych zmian przed wdrożeniem do produkcji. Umożliwia testy zarówno za pomocą Aplikacji Podatnika, jak i własnych wywołań API. | [https://ksef-test.mf.gov.pl/api/v2](https://ksef-test.mf.gov.pl/api/v2)                             | 30.09.2025                | FA(2), FA(3), FA_PEF (3), FA_KOR_PEF (3)   |
| **TR**  | Przedprodukcyjne (Demo/Preprod)  | Środowisko odpowiadające konfiguracji produkcyjnej, przeznaczone do końcowej walidacji integracji w warunkach zbliżonych do produkcji.     | [https://ksef-demo.mf.gov.pl/api/v2](https://ksef-demo.mf.gov.pl/api/v2)                             | 15.10.2025                | FA(3), FA_PEF (3), FA_KOR_PEF (3)          |
| **PRD** | Produkcyjne                      | Środowisko do wystawiania i odbierania faktur o pełnej mocy prawnej, z gwarantowanym SLA i właściwymi danymi produkcyjnymi.               | [https://ksef.mf.gov.pl/api/v2](https://ksef.mf.gov.pl/api/v2)                                       | 01.02.2026                | FA(3), FA_PEF (3), FA_KOR_PEF (3)          |

Dokumentacja dla każdego środowiska dostępna jest pod adresem `/docs/v2`.

> <font color="red">Uwaga:</font> Środowisko TE (testowe) służy wyłącznie do testowania integracji z API KSeF. Nie należy przesyłać do niego **faktur produkcyjnych** ani rzeczywistych danych podmiotów.

W środowisku testowym dopuszczalne jest uwierzytelnianie przy użyciu samopodpisanych certyfikatów (self-signed), co w praktyce oznacza, że wielu integratorów może [uwierzytelnić się](uwierzytelnianie.md#proces-uwierzytelniania) w kontekście tej samej firmy.
Z tego względu dane wprowadzone w środowisku TE nie są odizolowane i mogą być współdzielone między integratorami.
Do testów należy używać losowych identyfikatorów NIP, unikając jakichkolwiek danych rzeczywistych.

### Prace serwisowe na środowiskach testowych
W związku z planowanym, systematycznym rozwojem Krajowego Systemu e-Faktur (KSeF 2.0), **od dnia 1 października 2025 r.** na środowiskach testowych Systemu mogą być prowadzone cykliczne prace serwisowe.

Prace te będą realizowane w godzinach od **16:00 do 18:00**. W tym czasie mogą wystąpić przejściowe utrudnienia w dostępie do środowisk testowych.

Po zakończeniu prac w [changelogu](api-changelog.md) publikowane będą **wyłącznie zmiany mające wpływ na integrację** np. zmiany zachowań API, modyfikacje kontraktów, schematów XSD, limitów itp. Zmiany niewpływające na API i niezauważalne z perspektywy integracji np. wewnętrzne poprawki jakości, wydajności, bezpieczeństwa **mogą nie być komunikowane** lub będą prezentowane zbiorczo.