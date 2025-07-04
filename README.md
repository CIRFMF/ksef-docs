# **KSeF 2.0 \- Przewodnik dla Integratorów**
26.06.2025

Niniejszy dokument stanowi kompendium wiedzy dla deweloperów, analityków i integratorów systemów, którzy realizują integrację z Krajowym Systemem e-Faktur (KSeF) w wersji 2.0. Przewodnik koncentruje się na aspektach technicznych i praktycznych związanych z komunikacją z API systemu KSeF.

##  Wprowadzenie do KSeF 2.0

Krajowy System e-Faktur (KSeF) to centralny system teleinformatyczny służący do wystawiania i pobierania faktur ustrukturyzowanych w postaci elektronicznej.

## Spis treści
Przewodnik został podzielony na tematyczne sekcje, odpowiadające kluczowym funkcjom i obszarom integracyjnym w API KSeF:
* [Przegląd kluczowych zmian w KSeF 2.0](przeglad-kluczowych-zmian-ksef-api-2-0.md)
* [Uwierzytelnianie](uwierzytelnianie.md)
* [Uprawnienia](uprawnienia.md)
* [Certyfikaty KSeF](certyfikaty-KSeF.md)
* [Sesja interaktywna](sesja-interaktywna.md)
* [Sesja wsadowa](sesja-wsadowa.md)
* [Pobieranie faktur](pobieranie-faktur.md)
* [Zarządzanie tokenami KSeF](tokeny-ksef.md)

Dla każdego obszaru przewidziano:

* szczegółowy opis działania,
* przykład wywołania w językach C# i Java,
* powiązania ze specyfikacją [OpenAPI](https://ksef-test.mf.gov.pl/docs/v2) i kodem biblioteki referencyjnej.

Przykłady kodu prezentowane w przewodniku zostały przygotowane w oparciu o oficjalne biblioteki open source:
* [ksef-client-csharp](https://github.com/CIRFMF/ksef-client-csharp) – biblioteka w języku C#
* [ksef-client-java](https://github.com/CIRFMF/ksef-client-java) – biblioteka w języku Java

Obie biblioteki są utrzymywane i rozwijane przez zespoły Ministerstwa Finansów oraz udostępniane publicznie na zasadach open source. Zapewniają pełne wsparcie funkcjonalności API KSeF 2.0, w tym obsługę wszystkich endpointów, modeli danych oraz przykładowych scenariuszy wywołań. Wykorzystanie tych bibliotek pozwala znacząco uprościć proces integracji oraz minimalizuje ryzyko błędnej interpretacji kontraktów API.


## Środowiska Systemu
* **Środowisko produkcyjne (prod):**
  * Służy do wystawiania i odbierania faktur mających pełne skutki prawne.  
* **Środowisko przedprodukcyjne (demo):** 
  * Środowisko testowe oparte o faktyczne dane uwierzytelniające, wymaga posiadania rzeczywistych uprawnień. Faktury nie mają skutków prawnych.
* **Środowisko testowe (test):** https://ksef-test.mf.gov.pl/api/v2 (```dostępne od 30 września 2025```)
  * Przeznaczone do testów funkcjonalności, faktury wystawione w tym środowisku nie mają skutków prawnych. Można używać samodzielnie wygenerowanych podpisów i pieczęci.
