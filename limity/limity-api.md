## Limity żądań API
12.09.2025

***
> **Informacja dla środowiska testowego (TE)**
>
> - Limity wywołań API na TE są **wyłączone** i obecnie **nie są planowane**.
> - Parametry wykorzystania są **monitorowane**. W przypadku spadku wydajności lub generowania **ponadnormatywnego obciążenia** przez wybrane integracje, mogą zostać **wprowadzone limity ochronne**, aby utrzymać stabilność i równe warunki dostępu dla wszystkich użytkowników.
> - Ewentualne zmiany zostaną opublikowane w dokumentacji oraz w changelogu.
***


Ze względu na skalę działania KSeF oraz jego publiczny charakter, wprowadzono mechanizmy ograniczające intensywność żądań API. Ich celem jest ochrona stabilności systemu, ochrona przed cyber zagrożeniami oraz zapewnienie równych warunków dostępu dla wszystkich użytkowników. Limity określają maksymalną liczbę zapytań, które można wykonać w określonym czasie i wymuszają taki sposób integracji, który jest zgodny z założeniami architektury systemu.

### Ogólne zasady limitów

#### 1. Sposób naliczania limitów
Wszystkie żądania do API KSeF podlegają limitom. Ograniczenia te obejmują każde wywołanie chronionego endpointu. Dla potrzeb rozliczania ruchu żądania są grupowane według pary: kontekst i adres IP.

- **kontekst** - określony przez `ContextIdentifier` (`Nip`, `InternalId` lub `NipVatUe`) przekazany przy uwierzytelnianiu.
- **adres IP** - adres IP z którego nawiązywane jest połączenie sieciowe.

Limity żądań są naliczane niezależnie dla każdej pary: kontekst i adres IP. Oznacza to, że ruch w tym samym kontekście, ale z różnych adresów IP, rozliczany jest osobno.

Przykład  
Biuro księgowe A pobiera faktury w imieniu firmy B, korzystając z kontekstu firmy B (NIP) i łącząc się z KSeF z adresu IP1.
Równocześnie firma B pobiera faktury samodzielnie, w tym samym kontekście (swoim NIP-ie), ale z innego adresu IP - IP2. Mimo wspólnego kontekstu, różne adresy IP powodują, że limity są naliczane niezależnie.
W takiej sytuacji system traktuje każde połączenie jako oddzielną parę (kontekst + adres IP) i nalicza limity niezależnie: osobno dla biura księgowego A i osobno dla firmy B.

**Jednostki limitów**  
W tabelach limitów stosowane są następujące oznaczenia:
- req/s - liczba żądań na sekundę,
- req/min - liczba żądań na minutę,
- req/h - liczba żądań na godzinę.

**Model naliczania limitów (przesuwające się okno czasowe - sliding/rolling window)**  
Limity są egzekwowane w modelu przesuwającego się okna czasowego. W każdej chwili zliczane są żądania wykonane w okresie:

- dla progu req/h - w ostatnich 60 minutach,
- dla progu req/min - w ostatnich 60 sekundach,
- dla progu req/s - w ostatniej sekundzie.

Okna nie są wyrównywane do pełnych godzin ani minut (nie "zerują się" o :00). Wszystkie progi (req/s, req/min, req/h) obowiązują równolegle - blokada jest wyzwalana przy pierwszym przekroczeniu któregokolwiek z nich.

#### 2. Po przekroczeniu limitu system blokuje dostęp do API
W przypadku przekroczenia limitów żądań API zwracany jest kod HTTP **429 Too Many Requests**, a kolejne żądania są tymczasowo blokowane.  
Okres trwania blokady jest **dynamiczny** i zależy od częstotliwości oraz skali przekroczeń. Dokładny czas blokady jest zwracany w nagłówku odpowiedzi `Retry-After` (w sekundach). Wielokrotne przekroczenia mogą skutkować znacznym wydłużeniem blokady.

Przykład odpowiedzi 429:
```json
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 30

{
  "status": {
    "code": 429,
    "description": "Too Many Requests",
    "details": [ "Przekroczono limit 20 żądań na minutę. Spróbuj ponownie po 30 sekundach." ]
  }
}

```

#### 3. Rejestrowanie przekroczeń
Wszystkie przypadki przekroczenia limitów żądań są rejestrowane i analizowane przez mechanizmy bezpieczeństwa. Dane te służą do monitorowania stabilności API oraz wykrywania potencjalnych nadużyć. 
Szczególną uwagę system zwraca na wzorce wskazujące na próby obchodzenia limitów, np. poprzez równoległe i systemowe używanie wielu adresów IP w ramach jednego kontekstu. Takie działania mogą zostać uznane za zagrożenie bezpieczeństwa.

W przypadku powtarzających się naruszeń lub skrajnego obciążenia system może automatycznie zastosować działania ochronne, takie jak:
- blokada dostępu do API KSeF dla danego podmiotu lub zakresu adresów IP,
- ograniczenie dostępności dla najbardziej obciążających kontekstów.

#### 4. Wyższe limity w godzinach nocnych
W godzinach 20:00-06:00 obowiązują wyższe limity pobierania niż w ciągu dnia. 
Szczegółowe wartości zostaną określone w początkowym okresie działania KSeF 2.0, po dostrojeniu parametrów do rzeczywistych obciążeń.

#### 5. Wstępne założenia limitów
Limity żądań API zostały określone na podstawie przewidywanych scenariuszy wykorzystania systemu i modeli obciążeń. 

Rzeczywisty charakter ruchu będzie zależeć od sposobu implementacji integracji w systemach zewnętrznych oraz od generowanych przez nie wzorców obciążenia. Oznacza to, że limity ustalone na etapie projektowym mogą różnić się od wartości utrzymywanych w środowisku produkcyjnym.

Z tego względu limity mają charakter dynamiczny i mogą być dostosowywane w zależności od warunków eksploatacyjnych oraz zachowania integratorów. W szczególności dopuszcza się ich czasowe obniżenie w przypadku intensywnego lub nieefektywnego korzystania z API.

## Pobieranie faktur - limity

### Założenia architektoniczne
API KSeF w obszarze pobierania faktur zostało zaprojektowane jako mechanizm **synchronizacji dokumentów** pomiędzy centralnym repozytorium a lokalną bazą danych systemów zewnętrznych. Kluczowym założeniem jest, aby operacje biznesowe, takie jak wyszukiwanie, filtrowanie czy raportowanie, były wykonywane na danych przechowywanych lokalnie, które zostały wcześniej zsynchronizowane z KSeF. Takie podejście zwiększa stabilność działania, minimalizuje ryzyko przeciążenia systemu i pozwala na bardziej elastyczne wykorzystanie danych przez aplikacje klienckie.

API KSeF w zakresie pobierania faktur nie jest przeznaczone do obsługi bezpośrednich operacji użytkowników końcowych w czasie rzeczywistym. Oznacza to, że nie powinno być wykorzystywane do:
- pobierania pojedynczych faktur na żądanie użytkownika, np. podgląd faktury,
- pobierania listy metadanych faktur lub inicjowania eksportu paczek w reakcji na bieżące akcje w aplikacji, z wyjątkiem sytuacji, gdy użytkownik świadomie uruchamia synchronizację danych.

### Rekomendowany sposób integracji w zakresie pobierania
Do synchronizacji przyrostowej wykorzystywany jest endpoint `/invoices/query/metadata`. Szczegółowe zasady synchronizacji przyrostowej opisano w osobnym dokumencie.

W zależności od wolumenu faktur można stosować różne podejścia do ich pobierania:
1. **Scenariusze niskiego wolumenu** - jeżeli liczba faktur jest ograniczona i możliwa do obsłużenia w ramach dostępnych limitów w oczekiwanym czasie, można pobierać je synchronicznie wywołując `/invoices/ksef/{ksefNumber}` dla wybranych dokumentów.
2. **Scenariusze wysokiego wolumenu** - jeżeli liczba dokumentów jest znacząca i obsługa w trybie synchronicznym staje się niepraktyczna, zaleca się użycie mechanizmu eksportu (`/invoices/exports`). Eksport działa asynchronicznie, jest kolejkowany i dzięki temu nie wpływa negatywnie na wydajność systemu.
3. **Operacje biznesowe** - niezależnie od wybranej strategii, wszystkie działania użytkowników (wyszukiwanie, filtrowanie, raportowanie) powinny być realizowane na **lokalnej bazie danych**, zsynchronizowanej wcześniej z KSeF.

### Tryby synchronizacji i pobierania faktur  
Pobranie faktury do systemu księgowego może być realizowane w trzech trybach:
1. **Na żądanie użytkownika** - synchronizacja przyrostowa uruchamiana jest **manualnie** przez użytkownika, od ostatniego potwierdzonego punktu kontrolnego.
2. **Cyklicznie** - synchronizacja przyrostowa wykonywana jest automatycznie zgodnie z harmonogramem systemu.
3. **Tryb mieszany** - synchronizacja przyrostowa działa cyklicznie, a dodatkowo użytkownik może ją uruchomić manualnie na żądanie.

### Częstotliwość odpytywania
- **Nie zaleca się harmonogramów o wysokiej częstotliwości**. W środowisku produkcyjnym interwał cykliczny nie powinien być krótszy niż 15 minut dla każdego podmiotu występującego na fakturze (Podmiot 1, Podmiot 2, Podmiot 3, Podmiot upoważniony).
- **Profile niskiego wolumenu.** Zaleca się pobieranie na żądanie, uzupełnione cyklem np. raz na dobę w oknie nocnym.
- **Data otrzymania faktury.** Datą otrzymania faktury jest data nadania numeru KSeF. Numer ten jest przypisywany automatycznie przez system w chwili przetworzenia faktury i nie zależy od momentu jej pobrania do systemu księgowego.

### Przykłady niezalecanej implementacji
Niewłaściwa integracja może prowadzić do blokady w API. Do najczęstszych błędów należą:
1. Synchronizacja wyłącznie poprzez pobieranie pojedynczych faktur (ścieżka synchroniczna), bez wykorzystania eksportu paczek faktur.
Takie podejście jest dopuszczalne jedynie w profilach o niskim wolumenie; przy większej liczbie dokumentów należy korzystać z mechanizmu `/invoices/exports`.
2. Obsługa żądań użytkowników końcowych (np. wyświetlanie pełnej treści faktury w aplikacji, pobieranie pliku XML) poprzez bezpośrednie wywołania API KSeF zamiast korzystania z lokalnej bazy.

### Szczegółowe limity

| Endpoint | | req/s | req/min | req/h |
|----------|---|-------|---------|-------|
| Pobranie listy metadanych faktur | POST /invoices/query/metadata | 8 | 16 | 20 |
| Eksport paczki faktur | POST /invoices/exports | 4 | 8 | 20 |
| Pobranie faktury po numerze KSeF | GET /invoices/ksef/{ksefNumber} | 8 | 16 | 64 |

**Uwaga:** Jeżeli w scenariuszach biznesowych organizacji dostępne limity pobierania faktur są niewystarczające, prosimy o kontakt z działem wsparcia KSeF w celu indywidualnej analizy i dobrania odpowiedniego rozwiązania.

## Wysyłka faktur - limity

### Założenia architektoniczne
- Wysyłka faktur bez względu na typ wysyłki jest kolejkowana.
- Przetwarzanie jest zoptymalizowane pod jak najszybsze potwierdzenie poprawności faktury i zwrócenie numeru KSeF.

#### Wysyłka wsadowa (paczki faktur):

- Paczka faktur jest traktowana jako jedna wiadomość w kolejce (referencja do paczki zamiast osobnych wpisów dla każdej faktury) i przetwarzana z takim samym priorytetem jak pojedynczy dokument.
- Wysyłka w paczce ogranicza narzut sieciowy i operacyjny, ponieważ:
	- wykonywana jest mniejsza liczba żądań HTTP,
	- operacje na zawartości (odszyfrowanie, walidacja, zapis) wykonywane są batchowo, co stanowi najwydajniejszy sposób obsługi wielu dokumentów jednocześnie.
- Kompresja wsadu. Ze względu na format XML i wysoką powtarzalność elementów między fakturami (stała struktura, podobne nazwy pól, powtarzalne bloki) osiągany współczynnik kompresji jest zwykle bardzo korzystny, co znacząco zmniejsza wolumen danych i skraca czas transmisji. W praktyce szybciej jest przesłać jedną paczkę zawierającą np. 100 faktur niż 100 pojedynczych faktur w sesji interaktywnej.
- Limity. Mechanizm limitów działa niezależnie od trybu wysyłki. Wysyłka wsadowa z natury zmniejsza liczbę żądań i ułatwia efektywne wykorzystanie dostępnych limitów.
- Zastosowanie. Tryb wsadowy zalecany jest wszędzie tam, gdzie w jednym oknie operacyjnym przekazywanych jest więcej niż jeden dokument. W szczególności sprawdza się przy rozliczeniach cyklicznych klientów, w e-commerce oraz w zautomatyzowanych procesach fakturowania.

Przykładowe scenariusze zastosowania trybu wsadowego:
- **Sklep internetowy (e-commerce).** Zamówienia i płatności są przetwarzane asynchronicznie, a faktury wystawiane automatycznie przez system ERP lub moduł fakturowania. Pojedyncza faktura nie musi być przesyłane do KSeF natychmiast po wystawieniu. Wydzielony proces może agregować wystawione faktury i cyklicznie - np. co 5 minut - przesyłać je w paczkach wsadowych do KSeF, co znacząco ogranicza liczbę żądań HTTP i optymalizuje wykorzystanie limitów.
- **Usługi subskrypcyjne / rozliczenia cykliczne.** Faktury są generowane zbiorczo raz dziennie lub raz w miesiącu (np. w telekomunikacji lub mediach) i przesyłane w jednej paczce w ramach zaplanowanej sesji wsadowej.
- **Zautomatyzowane procesy fakturowania w przedsiębiorstwach.** Występują np. w sektorze dystrybucji, logistyki, produkcji lub usług zlecanych w modelu B2B. Faktury generowane są automatycznie na podstawie zdarzeń systemowych (dostawy, realizacje zleceń) i przesyłane zbiorczo, np. po zakończeniu operacji.

**Zalecenie:** dla zapewnienia maksymalnej efektywności integracji, zaleca się agregowanie dokumentów w jednej sesji wsadowej wszędzie tam, gdzie jest to możliwe z punktu widzenia procesów biznesowych. Pozwala to ograniczyć liczbę żądań API oraz optymalizuje wykorzystanie dostępnych limitów.

**Szczegółowe limity**

| Endpoint | | req/s | req/min | req/h |
|----------|---|-------|---------|-------|
| Otwarcie sesji wsadowej * | POST /sessions/batch | 10 | 20 | 120 |
| Zamknięcie sesji wsadowej | POST /sessions/batch/{referenceNumber}/close | 10 | 20 | 120 |

**Wysyłka części paczki** - żądania przesyłające części paczki w ramach jednej sesji wsadowej nie są objęte limitami API. W przypadku paczek podzielonych na wiele części zaleca się ich równoległe (wielowątkowe) przesyłanie, co znacząco skraca czas wysyłki.

#### Wysyłka interaktywna (pojedynczo)
Tryb interaktywny został zaprojektowany z myślą o scenariuszach wymagających szybkiej rejestracji pojedynczych faktur i natychmiastowego uzyskania numeru KSeF. W odróżnieniu od sesji wsadowej, każda faktura przesyłana jest niezależnie w ramach aktywnej sesji interaktywnej. Jego celem jest maksymalne skrócenie czasu potrzebnego na uzyskanie numeru KSeF dla pojedynczego dokumentu. Zastosowanie obejmuje scenariusze o niskim wolumenie, w których przesyłane są pojedyncze faktury.

Przykładowe scenariusze zastosowania trybu interaktywnego:
- **Punkt sprzedaży detalicznej (POS)**. Po zakończeniu transakcji wystawiana jest faktura, a system rejestruje ją natychmiast w KSeF i zwraca numer KSeF celem wydruku lub prezentacji klientowi.
- **Aplikacje mobilne i lekkie systemy sprzedażowe** nie posiadające mechanizmu kolejkowania lub buforowania, wysyłające faktury od razu po ich wystawieniu.
- **Zdarzenia jednostkowe lub nieregularne** np. pojedyncza faktura korygująca.

Tryb interaktywny, mimo większego narzutu sieciowego w przypadku większych wolumenów, jest niezbędnym uzupełnieniem trybu wsadowego w scenariuszach wymagających bieżącej reakcji lub natychmiastowej rejestracji dokumentu w KSeF. Należy go stosować wyłącznie tam, gdzie natychmiastowe przetworzenie faktury jest kluczowe dla procesu biznesowego lub gdy skala operacji nie uzasadnia wykorzystania sesji wsadowej.

**Szczegółowe limity**

| Endpoint | | req/s | req/min | req/h |
|----------|---|-------|---------|-------|
| Otwarcie sesji interaktywnej | POST /sessions/online | 10 | 30 | 120 |
| Wysłanie faktury * | POST /sessions/online/{referenceNumber}/invoices | 10 | 30 | 180 |
| Zamknięcie sesji interaktywnej | POST /sessions/online/{referenceNumber}/close | 10 | 30 | 120 |

\* **Uwaga:** Jeżeli w scenariuszach biznesowych organizacji regularnie osiągane są limity wysyłki w sesji interaktywnej, w pierwszej kolejności należy rozważyć zastosowanie trybu wsadowego, który pozwala efektywniej wykorzystać dostępne zasoby i limity.
W sytuacjach, gdy użycie sesji interaktywnej jest niezbędne, a osiągane limity pozostają niewystarczające, prosimy o kontakt z działem wsparcia KSeF w celu indywidualnej analizy i pomocy w doborze rozwiązania.

### Status sesji i faktur

**Szczegółowe limity**

| Endpoint | | req/s | req/min | req/h |
|----------|---|-------|---------|-------|
| Pobranie statusu faktury z sesji | GET /sessions/{referenceNumber}/invoices/{invoiceReferenceNumber} | 30 | 120 | 720 |
| Pobranie listy sesji | GET /sessions | 5 | 10 | 60 |
| Pobranie faktur sesji | GET /sessions/{referenceNumber}/invoices | 10 | 20 | 200 |
| Pobranie niepoprawnie przetworzonych faktur sesji | GET /sessions/{referenceNumber}/invoices/failed | 10 | 20 | 200 |
| Pozostałe | GET /sessions/* | 10 | 120 | 720 |

## Pozostałe

Domyślne limity obowiązują dla wszystkich zasobów API, które nie mają określonych wartości szczegółowych w niniejszej dokumentacji. Każdy taki endpoint posiada własny licznik limitów, a jego żądania nie wpływają na inne zasoby.

Limity te dotyczą wyłącznie zasobów chronionych. Nie obejmują one publicznych zasobów API, takich jak `/auth/challenge`, które nie wymagają uwierzytelnienia i posiadają własne mechanizmy ochrony - limit wynosi 60 żądań na sekundę dla jednego adresu IP.

| Endpoint | | req/s | req/min | req/h |
|----------|---|-------|---------|-------|
| Pozostałe | POST/GET /* | 10 | 30 | 120 |

Powiązane dokumenty:
- [Limity](limity.md)
