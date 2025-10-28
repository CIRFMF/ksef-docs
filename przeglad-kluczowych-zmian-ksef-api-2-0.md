# KSeF API 2.0 – Przegląd kluczowych zmian
09.06.2025

# Wprowadzenie

Niniejszy dokument skierowany jest do zespołów technicznych i programistów mających doświadczenie z wersją 1.0 API KSeF. Zawiera przegląd najważniejszych zmian wprowadzonych w wersji 2.0, wraz z omówieniem nowych możliwości oraz praktycznych usprawnień w integracji.

Celem dokumentu jest:

- Wskazanie głównych różnic względem wersji 1.0
- Przedstawienie korzyści z migracji na wersję 2.0
- Ułatwienie przygotowania integracji do wymagań systemu wdrażanych 1 lutego 2026 r.

---

## Dokumentacja i narzędzia wspierające integrację z KSeF API 2.0

Aby ułatwić przejście na nową wersję API oraz zapewnić prawidłową implementację, udostępniono zestaw oficjalnych materiałów i narzędzi wspierających integratorów:

**Dokumentacja techniczna (OpenAPI)**

Wersja 2.0 API KSeF została opisana w standardzie OpenAPI, co umożliwia zarówno łatwe przeglądanie dokumentacji przez programistów, jak i automatyczne generowanie kodu integracyjnego.

* **Dokumentacja** (interaktywna wersja online):
  Interfejs dokumentacyjny w formie portalu technicznego, zawierający opisy metod, struktur danych, parametrów i przykładów. Przeznaczony do użytku przez programistów i analityków integracyjnych.
  <br> Środowisko testowe (TE): \[[link](https://ksef-test.mf.gov.pl/docs/v2/index.html)\]
  <br> Środowisko produkcyjne (PRD)

* **Specyfikacja** (plik JSON OpenAPI):  
  Surowy plik specyfikacji OpenAPI w formacie JSON, przeznaczony do wykorzystania w narzędziach automatyzujących integrację (np. generatory kodu, walidatory kontraktów API).
  <br> Środowisko testowe (TE): \[[link](https://ksef-test.mf.gov.pl/docs/v2/openapi.json)\]
  <br> Środowisko produkcyjne (PRD)

**Oficjalna biblioteka integracyjna KSeF 2.0 Client (open source)**

Publiczna biblioteka udostępniona na zasadach open source, rozwijana równolegle z kolejnymi wersjami API i utrzymywana w pełnej zgodności ze specyfikacją. Stanowi rekomendowane narzędzie integracyjne, umożliwiające bieżące śledzenie zmian oraz ograniczenie ryzyka niezgodności z aktualnymi wydaniami systemu.

* **C\#:** \[[link](https://github.com/CIRFMF/ksef-client-csharp)\]

* **Java:** \[[link](https://github.com/CIRFMF/ksef-client-java)\]

**Opublikowane paczki**

Biblioteka KSeF 2.0 Client będzie dostępna w oficjalnych repozytoriach pakietów dla najpopularniejszych języków programowania. Dla platformy .NET zostanie opublikowana jako pakiet NuGet, natomiast dla środowiska Java – jako artefakt Maven Central. Publikacja w tych repozytoriach umożliwi łatwe włączenie biblioteki do projektów oraz automatyczne śledzenie aktualizacji zgodnych z kolejnymi wersjami API.

**Przewodnik krok po kroku**

* **Przewodnik integracyjny / tutorial:**  
  Praktyczne instrukcje krok po kroku wraz z fragmentami kodu ilustrujące sposób korzystania z kluczowych endpointów systemu.
  <br/>\[[link](https://github.com/CIRFMF/ksef-docs)\]

# Kluczowe zmiany w API 2.0

## Nowy model uwierzytelniania oparty o JWT ##

W wersji 1.0 uwierzytelnienie było ściśle powiązane z otwarciem sesji interaktywnej, co wprowadzało wiele ograniczeń i komplikowało integrację.

W wersji 2.0:

* Uwierzytelnianie zostało **wydzielone jako osobny proces**, niezależny od inicjalizacji sesji.

* Wprowadzono **standardowe tokeny JWT**, które są wykorzystywane do autoryzacji wszystkich operacji chronionych.

Korzyści:

* zgodność z praktykami rynkowymi,  
* możliwość wielokrotnego użycia tokena do tworzenia wielu sesji,  
* **obsługa odświeżania i unieważniania tokenów**.

Szczegóły procesu uwierzytelniania: \[[link](https://github.com/CIRFMF/ksef-docs/blob/main/uwierzytelnianie.md)\]

## Ujednolicony proces inicjalizacji dla sesji wsadowej i interaktywnej

W API 2.0 proces otwierania sesji został ujednolicony i uniezależniony od trybu pracy. Po uzyskaniu tokena uwierzytelniającego, można otworzyć zarówno sesję interaktywną: POST /sessions/online, jak i wsadową: POST /sessions/batch.

W obu przypadkach przekazywany jest prosty JSON zawierający:

* kod formularza (formCode),

* zaszyfrowany klucz AES do szyfrowania danych faktur (encryptionKey).

W przypadku wysyłki wsadowej przekazywana jest również lista cząstkowych plików wraz z metadanymi wchodzących w skład paczki.

Szczegóły i przykłady użycia: 
* wysyłka interaktywna \[[link](https://github.com/CIRFMF/ksef-docs/blob/main/sesja-interaktywna.md)\]
* wysyłka wsadowa \[[link](https://github.com/CIRFMF/ksef-docs/blob/main/sesja-wsadowa.md)\]

## Obowiązkowe szyfrowanie wszystkich faktur

W wersji 1.0 szyfrowanie faktur było obowiązkowe tylko w trybie wsadowym. W trybie interaktywnym możliwość szyfrowania istniała, ale była opcjonalna.

W wersji 2.0 szyfrowanie wszystkich faktur – zarówno w trybie wsadowym, jak i interaktywnym – **jest wymagane**.

Każda faktura lub paczka faktur musi być zaszyfrowana lokalnie przez klienta za pomocą **klucza AES**, który:

* jest generowany indywidualnie dla każdej sesji,

* przekazywany do systemu w trakcie otwierania sesji (encryptionKey).


## Szyfrowanie asymetryczne

W wersji 2.0 wprowadzono `RSA-OAEP` z `SHA-256` i `MGF1-SHA256`. Szyfrowanie tokenów KSeF realizowane jest oddzielnym kluczem niż szyfrowanie klucza symetrycznego używanego do faktur.

Aktualne **certyfikaty kluczy publicznych** zwracane są przez publiczny endpoint: GET `/security/public-key-certificates`

## Spójność i nowa konwencja nazewnicza w API 2.0

Jedną z kluczowych zmian w KSeF API 2.0 jest ujednolicenie i uproszczenie konwencji nazewniczej zasobów, parametrów oraz modeli json. W wersji 1.0 API zawierało szereg niespójności i nadmiarowej złożoności wynikających z ewolucji systemu.

W wersji 2.0:

* **Endpointy** zyskały czytelne, REST-owe nazewnictwo (np. sessions/online, auth/token, permissions/entities/grants).

* **Nazwy operacji** zostały uproszczone i odzwierciedlają rzeczywistą akcję (np. grant, revoke, refresh).

* Uporządkowano strukturę **nagłówków, parametrów i formatów danych**, tak aby była spójna i zgodna z dobrymi praktykami projektowania REST API. 

* **Struktury danych** są płaskie i klarowne – typy identyfikatorów i uprawnień mają jawnie określone typy enum (Nip, Pesel, Fingerprint), bez konieczności analizowania podtypów.

Zmiany w wersji 2.0 obejmują również aktualizację nazw oraz struktur danych. Choć pełna mapa tych zmian nie została przedstawiona w niniejszym dokumencie, jest ona dostępna w dokumentacji OpenAPI v2 oraz w przykładach kodu w oficjalnym repozytorium GitHub.

Należy podkreślić, że zmiany nie mają charakteru drastycznego – **nie wpływają** na ogólną logikę działania systemu KSeF, a jedynie porządkują i upraszczają nazewnictwo oraz struktury, czyniąc API bardziej przejrzystym i intuicyjnym w użyciu.

Migracja do wersji 2.0 powinna być traktowana jako zmiana kontraktu integracyjnego i wymaga dostosowania implementacji po stronie systemów zewnętrznych. Zalecane jest wykorzystanie oficjalnej biblioteki integracyjnej **KSeF 2.0 Client**, rozwijanej i utrzymywanej przez zespół odpowiedzialny za API. Biblioteka ta implementuje wszystkie dostępne endpointy i modele danych, co znacząco ułatwia proces migracji oraz zapewnia stabilne wsparcie również w przyszłych wersjach systemu.

## Nowy moduł do zarządzania certyfikatami wewnętrznymi

W ramach wersji KSeF API 2.0 wprowadzono mechanizmy umożliwiające wydawanie oraz zarządzanie wewnętrznymi **certyfikatami KSeF** \[link do dokumentacji\]. Certyfikaty będą umożliwiały uwierzytelnienie w KSeF oraz są niezbędne do wystawienia faktury w trybie offline \[link do dokumentacji\].

Podmioty, które pomyślnie przejdą proces uwierzytelnienia, będą mogły:

* złożyć wniosek o wydanie certyfikatu wewnętrznego KSeF, zawierającego wybrane atrybuty z certyfikatu podpisu wykorzystanego podczas uwierzytelnienia, 

* pobrać wystawiony certyfikat w postaci cyfrowej,

* sprawdzić status złożonego wniosku o wydanie certyfikatu,

* pobrać listę metadanych wydanych certyfikatów,

* sprawdzić przysługujący limit liczby certyfikatów.

## Usprawnienie procesu wysyłki wsadowej

W wersji KSeF API 2.0 wprowadzono istotne udoskonalenie w zakresie przetwarzania sesji wsadowych. Dotychczasowe rozwiązanie dostępne w API 1.0 było nieefektywne – w przypadku, gdy choćby jedna faktura w paczce zawierała błąd, odrzucana była cała przesyłka. Takie podejście skutecznie ograniczało wykorzystanie trybu wsadowego przez integratorów i powodowało znaczne utrudnienia operacyjne.

W nowym rozwiązaniu, przy przesyłaniu paczki dokumentów:

* każda faktura przetwarzana jest niezależnie,

* ewentualne błędy wpływają wyłącznie na konkretne faktury, a nie na całość przesyłki,

* liczba błędnych faktur jest zwracana dla statusu sesji,

* dostępny jest dedykowany endpoint umożliwiający pobranie szczegółowego statusu błędnie przetworzonych faktur wraz z informacją o ewentualnym błędzie.

Zmiana ta znacząco podnosi niezawodność i efektywność trybu wsadowego i opiera się na tym samym modelu przesyłania paczek bez ryzyka utraty całej paczki z powodu pojedynczych błędów.

## Weryfikacja duplikatów faktur  
Zmieniono sposób wykrywania duplikatów – obecnie sprawdzane są dane biznesowe faktury (Podmiot1:NIP, RodzajFaktury, P_2), a nie skrót pliku. Szczegóły – [Weryfikacja duplikatów](faktury/weryfikacja-faktury.md).

## Zmiana w module Uprawnień

Zmiany w module uprawnień wiążą się ze zmianą niektórych aspektów logiki ich funkcjonowania w KSeF 2.0. 

W odpowiedzi na zgłaszane uwagi, w wersji 2.0 systemu wprowadzono mechanizm nadawania uprawnień w sposób pośredni, który zastępuje dotychczasową zasadę dziedziczenia uprawnień do przeglądania i wystawiania faktur. Nowy interfejs umożliwia rozdzielenie przeglądania faktur klienta (partnera) oraz możliwości wystawiania faktur w jego imieniu od przeglądania faktur oraz wystawiania faktur własnych podmiotu (np. biura rachunkowego).

Mechanizm polega na nadaniu podmiotowi przez klienta uprawnienia do przeglądania lub wystawiania faktur z włączoną specjalną opcją zezwalającą na dalsze przekazywanie tego uprawnienia przez uprawniony podmiot. Po otrzymaniu takiego uprawnienia podmiot może je nadać np. swoim pracownikom. Po wykonaniu tych czynności, pracownicy ci będą mieli możliwość obsługi wskazanego klienta w zakresie określonym przez nadane uprawnienia.

Możliwe jest również nadanie przez podmiot tzw. uprawnień generalnych, które pozwalają uprawnionemu w ten sposób pracownikowi obsługiwać wszystkich klientów podmiotu – oczywiście w takim zakresie, w jakim klienci ci uprawnili ten podmiot i przy uwzględnieniu zakresu uprawnień pracownika (do przeglądania i/lub wystawiania faktur). 

Dzięki takiemu mechanizmowi, nadawanie i funkcjonowanie uprawnień do przeglądania i wystawiania faktur w ramach samego podmiotu nie są powiązane z uprawnieniami do obsługi faktur klientów. Daje to podmiotom lepsze możliwości profilowania uprawnień pracowników, niż uprzednio funkcjonujący w KSeF mechanizm dziedziczenia uprawnień. Polegał on bowiem na nadawaniu uprawnień pracownikom do przeglądania i wystawiania faktur w ramach podmiotu, a jeśli podmiot dysponował odpowiednimi uprawnieniami od klientów, to uprawnienia te automatycznie przechodziły na pracowników (były dziedziczone). W rezultacie – pracownik mógł obsługiwać klientów wyłącznie wtedy, gdy równocześnie miał prawo do przeglądania i/lub wystawiania faktur podmiotu. A to w wielu wypadkach było uprawnienie nadmiarowe, co mogło powodować problemy w organizacjach.

Ponadto w systemie wprowadzono nowy typ uprawnienia oraz nowe możliwości logowania, co umożliwia obsługę samofakturowanie przez podmioty unijne. Możliwe jest obecnie logowanie w kontekście określonym przez podmiot polski (identyfikowany przez NIP) oraz podmiot z kraju UE identyfikowany numerem VAT kraju unijnego. W tak określonym kontekście możliwe jest wystawianie przez uwierzytelnionych przedstawicieli wskazanego podmiotu unijnego faktur w trybie samofakturowania w imieniu wskazanego podmiotu polskiego.

W ramach definicji API wszystkie uprawnienia zostały uporządkowane w logiczne grupy odpowiadające poszczególnym obszarom funkcjonalnym systemu.

## Limity wywołań API (rate limiting) ##
W wersji KSeF API 2.0 wprowadzono precyzyjny i przewidywalny mechanizm ograniczeń liczby wywołań (rate limits), który zastępuje dotychczasowe rozwiązania znane z wersji 1.0.
Każdy endpoint w systemie objęty jest limitem liczby żądań w zadanych przedziałach czasowych: na sekundę, minutę, godzinę.

Zakresy i wartości limitów są:
* publicznie udostępniane: [limity API](limity/limity-api.md),  
* zróżnicowane w zależności od środowiska (środowisko testowe ma limity mniej restrykcyjne niż produkcyjne),  
* dostosowane do charakteru operacji:  
  * dla endpointów chronionych – limity stosowane są per kontekst i adres IP,  
  * dla endpointów otwartych – limity stosowane są per adres IP.

Nowy model limitów został zaprojektowany tak, aby nie ograniczać typowego testowania aplikacji integrujących się z systemem.
Rozwiązanie to zapewnia większą przejrzystość, przewidywalność oraz lepszą odporność systemu, zarówno w środowiskach testowych, jak i produkcyjnych.

## Pomocnicze API do generowania danych testowych (środowisko testowe)

W środowisku testowym KSeF API 2.0 udostępnione zostanie dedykowane **pomocnicze API do generowania danych testowych**, umożliwiające szybkie tworzenie firm, struktur organizacyjnych i kontekstów niezbędnych do przeprowadzania testów integracyjnych.

Dzięki temu rozwiązaniu możliwe będzie m.in.:

* **symulowanie założenia nowego podmiotu gospodarczego**,

* **symulacja nadania uprawnień przez ZAW-FA**,

* **tworzenie jednostek w ramach struktury JST**,

* tworzenie podatkowych **grup VAT** (GVAT) wraz z podmiotami powiązanymi,

* **definiowanie organów egzekucyjnych** **i komorników sądowych**.

Standardowo, proces rejestracji firm i nadania uprawnień dla rzeczywistych podmiotów w środowisku produkcyjnym wymaga działań formalnych (np. wizyty w urzędzie skarbowym). W środowisku testowym takie dane nie istnieją. Dlatego **pomocnicze API jest niezbędnym narzędziem**, umożliwiającym integratorom samodzielne tworzenie podmiotów testowych, na których mogą swobodnie realizować i weryfikować pełne scenariusze działania swoich aplikacji.
