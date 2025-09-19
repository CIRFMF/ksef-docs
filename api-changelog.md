## Zmiany w API 2.0

### Wersja 2.0.0 RC5

- **UPO**
  - Dodano element `Uwierzytelnienie`, który porządkuje dane z nagłówka UPO i rozszerza je o dodatkowe informacje; zastępuje dotychczasowe `IdentyfikatorPodatkowyPodmiotu` oraz `SkrotZlozonejStruktury`.
  - `Uwierzytelnienie` zawiera:
    - `IdKontekstu` – identyfikator kontekstu uwierzytelnienia,
    - dowód uwierzytelnienia (w zależności od metody): 
      - `NumerReferencyjnyTokenaKSeF` - identyfikator tokenu uwierzytelniającego w systemie KSeF,
      - `SkrotDokumentuUwierzytelniajacego` - wartość funkcji skrótu dokumentu uwierzytelniającego w postaci otrzymanej przez system (łącznie z podpisem elektronicznym).
  - W elemencie `Dokument` dodano:
    - NipSprzedawcy,
    - DataWystawieniaFaktury,
    - DataNadaniaNumeruKSeF.
  - W elemencie `Dokument` usunięto `DataPrzeslaniaDokumentu`.
  - Schemat UPO: 
    - faktura: [link](/faktury/upo/schemy/upo-faktura-v3-1.xsd)
    - sesja: [link](/faktury/upo/schemy/upo-sesja-v4-2.xsd).

- **Uwierzytelnianie**  
  Doprecyzowano kody statusów w GET `/auth/{referenceNumber}`, `/auth/sessions`: 
  - 415 (brak uprawnień), 
  - 425 (uwierzytelnienie unieważnione), 
  - 450 (błędny token: nieprawidłowy token, nieprawidłowy czas, unieważniony, nieaktywny), 
  - 460 (błąd certyfikatu: nieważny, błąd weryfikacji łańcucha, niezaufany łańcuch, odwołany, niepoprawny).  
  Pozostają 100 (w toku), 200 (sukces) i 500 (nieznany błąd).

- **Certyfikaty**    
  Zmiana struktury odpowiedzi w  GET `certificates/enrollments/data` ("Pobranie danych do wniosku certyfikacyjnego"):
  - Usunięto: givenNames (tablica string, nullable).
  - Dodano: givenName (string).
  - Charakter zmiany: breaking (zmiana nazwy i typu pola z tablicy na tekst).

- **Tokeny KSeF**  
  Dodano kod błedu dla odpowiedzi POST `/api/v2/tokens` ("Wygenerowanie nowego tokena"): `26002` - "Nie można wygenerować tokena dla obecnego typu kontekstu". Token może być generowany wyłącznie w kontekście `Nip` lub `InternalId`.

- **Pobieranie metadanych faktur (GET `/invoices/query/metadata`)**  
  - Usunięto właściwość `totalCount` z odpowiedzi wyszukiwania metadanych.
  - Dodano właściwość `seller.nip` w filtrze żądania i w zwracanych metadanych. Właściwość `seller.identifier` oznaczono jako deprecated (zostanie usunięta w następnym wydaniu).
  - Dodano właściwość `authorizedSubject.nip` w zwracanych metadanych. Właściwość `authorizedSubject.identifier` oznaczono jako deprecated (zostanie usunięta w następnym wydaniu).

- **Eksport paczki faktur (POST `/invoices/exports`)**
  - Dodano właściwość `seller.nip` w filtrze żądania. Właściwość `seller.identifier` oznaczono jako deprecated (zostanie usunięta w następnym wydaniu).

- **Uprawnienia**
  - Rozszerzono DELETE `/api/v2/permissions/common/grants/{permissionId}` o uprawnienie `VatUeManage`. Wymagane uprawnienia: CredentialsManage lub `VatUeManage`.
  - Rozszerzono żądanie POST `/permissions/eu-entities/administration/grants` ("Nadanie uprawnień administratora podmiotu unijnego") o "Nazwę podmiotu" `subjectName`.
  - Dodano uprawnienie `Introspection` do wszystkich endpointów sprawdzających status w sesji GET `/sessions/{referenceNumber}/...` Każdy z tych endpointów można teraz wywołać posiadając `InvoiceWrite` lub `Introspection`.

- **Załączniki do faktur**  
  Dodano endpoint GET `/api/v2/permissions/attachments/status` do sprawdzania statusu zgody na wystawianie faktur z załącznikiem.

- **Pobranie listy sesji**  
  Rozszerzono uprawnienia dla GET `/api/v2/sessions`: dodano `InvoiceWrite`. Posiadając uprawnienie `InvoiceWrite`, można pobierać wyłącznie sesje utworzone przez podmiot uwierzytelniający; posiadając uprawnienie `Introspection`, można pobierać wszystkie sesje.

- **OpenAPI**
  - Dodano uniwersalny kod błędu walidacji danych wejściowych `21405` do wszystkich endpointów. Treść błędu z walidatora zwracana w odpowiedzi.
  - Dodano odpowiedź 400 z walidacją zwracającą kod błędu `30001` („Podmiot lub uprawnienie już istnieje.”) dla POST `/api/v2/testdata/subject` oraz POST `/api/v2/testdata/person`.
  - Zaktualizowano przykłady (example) w definicjach endpointów.

### Wersja 2.0.0 RC4

- **Certyfikaty KSeF**
  - Dodano nową właściwość `type` w certyfikatach KSeF.
  - Dostępne typy certyfikatów:
    - `Authentication` – certyfikat do uwierzytelnienia w systemie KSeF,
    - `Offline` – certyfikat ograniczony wyłącznie do potwierdzania autentyczności wystawcy i integralności faktury w trybie offline (KOD II).
  - Zaktualizowano dokumentację procesów `/certificates/enrollments`, `/certificates/query`, `/certificates/retrieve`.

- **Kody QR**
  - Doprecyzowano, że KOD II może być generowany wyłącznie w oparciu o certyfikat typu `Offline`.
  - Dodano ostrzeżenie bezpieczeństwa, że certyfikaty `Authentication` nie mogą być używane do wystawiania faktur offline.

- **Status sesji**
  - Aktualizacja autoryzacji - pobieranie informacji o sesji, fakturach i UPO wymaga uprawnienia: ```InvoiceWrite```.
  - Zmieniono kod statusu *trwa przetwarzanie*: z `300` na `150` dla sesji wsadowej.

- **Pobieranie metadanych faktur (`/invoices/query/metadata`)**  
Rozszerzono model odpowiedzi o pola:
  - `fileHash` – skrót SHA256 faktury,
  - `hashOfCorrectedInvoice` – skrót SHA256 korygowanej faktury offline,
  - `thirdSubjects` – lista trzecich podmiotów,
  - `authorizedSubject` – podmiot upoważniony (nowy obiekt `InvoiceMetadataAuthorizedSubject` zawierający `identifier`, `name`, `role`),
 - Dodano możliwość filtrowania po typie dokumentu (`InvoiceQueryFormType`), dostępne wartości: `FA`, `PEF`, `RR`. 
 - Pole `schemaType` oznaczone jako deprecated – planowane do usunięcia w przyszłych wersjach API.


- **Dokumentacja**
  - Dodano dokument opisujący [numer KSeF](faktury/numer-ksef.md).
  - Dodano dokument opisujący [korektę techniczną](offline/korekta-techniczna.md) dla faktur wystawionych w trybie offline.
  - Doprecyzowano sposób [wykrywania duplikatów](faktury/weryfikacja-faktury.md)

- **OpenAPI**
  - Pobranie listy metadanych faktur 
    - Dodano właściwość: `hasMore` (boolean) – informuje o dostępności kolejnej strony wyników. Właściwość `totalCount` została oznaczona jako deprecated (pozostaje chwilowo w odpowiedzi dla zgodności wstecznej).
    - W filtrowaniu po zakresie `dateRange` właściwość `to` (data końcowa zakresu) nie jest już obowiązkowa.
  - Wyszukiwanie nadanych uprawnień - w odpowiedzi dodano właściwość `hasMore`, usunięto `pageSize`, `pageOffset`.
  - Pobranie statusu uwierzytelniania - usunięto z odpowiedzi redundantne `referenceNumber`, `isCurrent`.
  - Ujednolicenie stronicowania - endpoint `/sessions/{referenceNumber}/invoices` (pobranie faktur sesji) przechodzi na paginację opartą o nagłówek żądania `x-continuation-token`; usunięto parametr `pageOffset`, `pageSize` pozostaje bez zmian. Pierwsza strona bez nagłówka; kolejne strony pobiera się przez przekazanie wartości tokenu zwróconej przez API. Zmiana spójna z innymi zasobami korzystającymi z `x-continuation-token` (np. `/auth/sessions`, `/sessions/{referenceNumber}/invoices/failed`).
  - Usunięto obsługę identyfikatora `InternalId` w polu `targetIdentifier` podczas nadawania uprawnień pośrednich (`/permissions/indirect/grants`). Od teraz dopuszczalny jest wyłącznie identyfikator `Nip`.
  - Status operacji nadawania uprawnień – rozszerzono listę możliwych kodów statusu w odpowiedzi:
    - 410 – Podane identyfikatory są niezgodne lub pozostają w niewłaściwej relacji.
    - 420 – Użyte poświadczenia nie mają uprawnień do wykonania tej operacji.
    - 430 – Kontekst identyfikatora nie odpowiada wymaganej roli lub uprawnieniom.
    - 440 – Operacja niedozwolona dla wskazanych powiązań identyfikatorów.
    - 450 – Operacja niedozwolona dla wskazanego identyfikatora lub jego typu.
  - Dodano obsługę błędu **21418** – „Przekazany token kontynuacji ma nieprawidłowy format” we wszystkich endpointach wykorzystujących mechanizm paginacji z użyciem `continuationToken` (`/auth/sessions`, `/sessions`, `/sessions/{referenceNumber}/invoices`, `/sessions/{referenceNumber}/invoices/failed`, `/tokens`).
  - Doprecyzowano proces pobierania paczki faktur:
    - `/invoices/exports` – rozpoczęcie procesu tworzenia paczki faktur,
    - `/invoices/async-query/{operationReferenceNumber}` – sprawdzenie statusu i odbiór gotowej paczki.
  - Zmieniono nazwę modelu `InvoiceMetadataQueryRequest` na `QueryInvoicesMetadataReponse`.
  - Rozszerzono typ `PersonPermissionsAuthorIdentifier` o nową wartość `System` (Identyfikator systemowy). Wartość ta wykorzystywana jest do oznaczania uprawnień nadawanych przez KSeF na podstawie złożonego wniosku ZAW-FA. Zmiana dotyczy endpointu: `/permissions/query/persons/grants`.

### Wersja 2.0.0 RC3

- **Dodanie endpointu do pobierania listy metadanych faktur**  
  - `/invoices/query` (mock) zastąpiony przez `/invoices/query/metadata` – produkcyjny endpoint do pobierania metadanych faktur
  - Aktualizacja powiązanych modeli danych.

- **Aktualizacja mockowego endpointu `invoices/async-query` do inicjalizacji zapytania o pobranie faktur**  
  Zaktualizowano powiązane modele danych.

- **OpenAPI**
  - Uzupełniono specyfikację endpointów o wymagane uprawnienia (`x-required-permissions`).
  - Dodano odpowiedzi `403 Forbidden` i `401 Unauthorized` w specyfikacji endpointów.
  - Dodano atrybut ```required``` w odpowiedziach zapytań o uprawnienia.
  - Zaktualizowano opis endpointu  ```/tokens```
  - Usunięto zduplikowane definicje ```enum```
  - Ujednolicono model odpowiedzi SessionInvoiceStatusResponse w ```/sessions/{referenceNumber}/invoices``` oraz ```/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}```.
  - Dodano status walidacji 400: „Uwierzytelnianie zakończone niepowodzeniem | Brak przypisanych uprawnień”.

- **Status sesji**
  - Dodano status ```Cancelled``` - "Sesja anulowania. Został przekroczony czas na wysyłkę w sesji wsadowej, lub nie przesłano żadnych faktur w sesji interaktywnej."
  - Dodano nowe kody błedów:
    - 415 - "Brak możliwości wysyłania faktury z załącznikiem"
    - 440 - "Sesja anulowana, przekroczono czas wysyłki"
    - 445 - "Błąd weryfikacji, brak poprawnych faktur"

- **Status wysyłki faktur**
  - Dodano datę ```AcquisitionDate``` - data nadania numeru KSeF.
  - Pole ```ReceiveDate``` zastąpiono ```InvoicingDate``` – data przyjęcia faktury do systemu KSeF.  

- **Wysyłka faktur w sesji**
  - Dodano [walidację](faktury/weryfikacja-faktury.md#ograniczenia-ilo%C5%9Bciowe) rozmiaru paczki zip (100 MB) i liczby paczek (50) w sesji wsadowej
  - Dodano [walidację](faktury/weryfikacja-faktury.md#ograniczenia-ilo%C5%9Bciowe) liczby faktur w sesji interaktywnej i wsadowej.
  - Zmieniono kod statusu „Trwa przetwarzanie” z 300 na 150.

- **Uwierzytelnienie z wykorzystaniem podpisu XAdES**  
  - Poprawka ContextIdentifier w xsd AuthTokenRequest. Należy użyć poprawionej wersji [schematu XSD](https://ksef-test.mf.gov.pl/docs/v2/schemas/authv2.xsd). [Przygotowanie dokumentu XML](uwierzytelnianie.md#1-przygotowanie-dokumentu-xml-authtokenrequest)
  - Dodano kod błędu`21117` - „Nieprawidłowy identyfikator podmiotu dla wskazanego typu kontekstu”.

- **Usunięcie endpointu do anonimowego pobierania faktury ```invoices/download```**  
  Funkcjonalność pobierania faktur bez uwierzytelnienia została usunięta; dostępna wyłącznie w webowym narzędziu KSeF do weryfikacji i pobierania faktur.

- **Dane testowe - obsługa faktur z załącznikami**  
  Dodano nowe endpointy umożliwiające testowanie wysyłki faktur z załącznikami.

- **Certificaty KSeF - Walidacja typu i długości klucza w CSR**  
  - Uzupełniono opis endpointu POST ```/certificates/enrollments``` o wymagania dotyczące typów kluczy prywatnych w CSR (RSA, EC),
  - Dodano nowy kod błędu 25010 w odpowiedzi 400: „Nieprawidłowy typ lub długość klucza.”

- **Aktualizacja formatu certyfikatów publicznych**  
  `/security/public-key-certificates` – zwraca certyfikaty w formacie DER zakodowane w Base64.


### Wersja 2.0.0 RC2
- **Nowe endpointy do zarządzania sesjami uwierzytelniania**  
  Umożliwiają przeglądanie oraz unieważnianie aktywnych sesji uwierzytelniających.  
  [Zarządzanie sesjami uwierzytelniania](auth/sesje.md)

- **Nowy endpoint do pobierania listy sesji wysyłki faktur**\
  `/sessions` – umożliwia pobranie metadanych dla sesji wysyłkowych (interaktywnych i wsadowych), z możliwością filtrowania m.in. po statusie, dacie zamknięcia i typie sesji.\
  [Pobieranie listy sesji](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md#1-pobranie-listy-sesji)
  

- **Zmiana w listowaniu uprawnień**  
  `/permissions/query/authorizations/grants` – dodano typ zapytania (queryType) w filtrowaniu [uprawnień podmiotowych](uprawnienia.md#pobranie-listy-uprawnień-podmiotowych-do-obsługi-faktur).

- **Obsługa nowej wersji schematu faktury FA(3)**  
  W ramach otwierania sesji interaktywnej i wsadowej możliwy jest wybór schemy FA(3).

- **Dodanie pola invoiceFileName w odpowiedzi dla sesji wsadowej**\
  `/sessions/{referenceNumber}/invoices` – dodano pole invoiceFileName zawierające nazwę pliku faktury. Pole występuje tylko dla sesji wsadowych.
   [Pobranie informacji na temat przesłanych faktur](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md#3-pobranie-informacji-na-temat-przesłanych-faktur)