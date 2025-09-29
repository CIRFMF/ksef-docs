## Zmiany w API 2.0

### Wersja 2.0.0 RC5.1
- Usunięto nadmiarowe wartości enum `None`, `AllPartners` w następujących miejscach:
  - we właściwości `contextIdentifier.type` żądania POST `/auth/ksef-token` ("Uwierzytelnienie z wykorzystaniem tokena KSeF"),
  - we właściwościach `authorIdentifier.type` oraz `contextIdentifier.type` w modelu odpowiedzi GET `/tokens` ("Pobranie listy wygenerowanych tokenów").

- Ujednoznaczniono model odpowiedzi GET `/tokens`:  
  właściwości `authorIdentifier.type`, `authorIdentifier.value`, `contextIdentifier.type`, `contextIdentifier.value` oznaczono jako zawsze zwracane (required, non-nullable).

- **Sesja wsadowa**  
  Usunięto nadmiarowy kod błędu `21401`	- "Dokument nie jest zgodny ze schemą (json)".

- **Pobieranie metadanych faktur (GET `/invoices/query/metadata`)**  
  - Dodano w odpowiedzi (zawsze zwracaną) właściwość `isTruncated` (boolean) – "Określa, czy wynik został obcięty z powodu przekroczenia limitu liczby faktur (10 000)",
  - Oznaczono właściwość `amount.type` jako wymaganą w filtrze żądania.

- **Eksport paczki faktur: zlecenie (POST `/invoices/exports`)**
  - Oznaczono `operationReferenceNumber` w modelu odpowiedzi jako zawsze zwracane,
  - Oznaczono właściwość `amount.type` jako wymaganą w filtrze żądania.

### Wersja 2.0.0 RC5

- **Obsługa faktur PEF i dostawców usług Peppol**
  - Dodano obsługę faktur `PEF` wysyłanych przez dostawcę usług Peppol. Nowe możliwości nie zmieniają dotychczasowych zachowań KSeF dla innych formatów, są rozszerzeniem API.
  - Wprowadzono nowy typ kontekstu uwierzytelniania: `PeppolId`, umożliwiający pracę w kontekście dostawcy usług Peppol.
  - Automatyczna rejestracja dostawcy: przy pierwszym uwierzytelnieniu dostawcy usług Peppol (z użyciem dedykowanego certyfikatu) następuje jego automatyczna rejestracja w systemie.
  - Dodano enpoint GET `/peppol/query` ("Lista dostawców usług Peppol") zwracający zarejestrowanych dostawców.
  - Zaktualizowano reguły dostępu dla otwarcia i zamknięcia sesji, wysyłka faktur wymagaja uprawnienia `PefInvoiceWrite`.
  - Dodano nowe schematy faktur: `FA_PEF (3)`, `FA_KOR_PEF (3)`,
  - Rozszerzono `ContextIdentifier` o `PeppolId` w xsd `AuthTokenRequest`.

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
  - Ujednolicono schemat UPO. UPO dla faktury i dla sesji korzystają ze wspólnej schemy [upo-v4-2.xsd](/faktury/upo/schemy/upo-v4-2.xsd). Zastępuje dotychczasowe upo-faktura-v3-0.xsd i upo-sesja-v4-1.xsd.

- **Limity żądań API**  
  Dodano specyfikację [limitów żądań API](limity-api.md).    

- **Uwierzytelnianie**  
  - Doprecyzowano kody statusów w GET `/auth/{referenceNumber}`, `/auth/sessions`: 
    - `415` (brak uprawnień), 
    - `425` (uwierzytelnienie unieważnione), 
    - `450` (błędny token: nieprawidłowy token, nieprawidłowy czas, unieważniony, nieaktywny), 
    - `460` (błąd certyfikatu: nieważny, błąd weryfikacji łańcucha, niezaufany łańcuch, odwołany, niepoprawny).  
  - Aktualizacja opcjonalnej polityki IP w XSD `AuthTokenRequest`:
    Zastąpiono `IpAddressPolicy` nową strukturą `AuthorizationPolicy`/`AllowedIps`. Zaktualizowano dokument [Uwierzytelnianie](uwierzytelnianie.md).

- **Autoryzacja**
  - Rozszerzono reguły dostępu o `VatUeManage`, `SubunitManage` dla DELETE `/permissions/common/grants/{permissionId}`: operację można wykonać, jeżeli podmiot posiada `CredentialsManage`, `VatUeManage` lub `SubunitManage`.
  - Rozszerzono reguły dostępu o `Introspection` dla GET `/sessions/{referenceNumber}/...`: każdy z tych endpointów można teraz wywołać posiadając `InvoiceWrite` lub `Introspection`.
  - Rozszerzono reguły dostępu o `InvoiceWrite` dla GET `/sessions` ("Pobranie listy sesji"): posiadając uprawnienie `InvoiceWrite`, można pobierać wyłącznie sesje utworzone przez podmiot uwierzytelniający; posiadając uprawnienie `Introspection`, można pobierać wszystkie sesje.
  - Zmieniono reguły dostępu dla DELETE `/tokens/{referenceNumber}`: usunięto wymóg uprawnienia `CredentialsManage`.

- **Pobranie danych do wniosku certyfikacyjnego (GET `certificates/enrollments/data`)**    
  - Zmiana struktury odpowiedzi:
    - Usunięto: givenNames (tablica string).
    - Dodano: givenName (string).
    - Charakter zmiany: breaking (zmiana nazwy i typu pola z tablicy na tekst).
  - Dodano kod błędu `25011` — „Nieprawidłowy algorytm podpisu CSR”.
  - Uściślono wymagania dotyczące klucza prywatnego używanego do podpisu CSR w [Certyfikaty KSeF](certyfikaty-KSeF.md).

- **Tokeny KSeF**  
  - Dodano kod błędu dla odpowiedzi POST `/tokens` ("Wygenerowanie nowego tokena"): `26002` - "Nie można wygenerować tokena dla obecnego typu kontekstu". Token może być generowany wyłącznie w kontekście `Nip` lub `InternalId`.
  - Rozszerzono katalog uprawnień możliwych do przypisania tokenowi: dodano `SubunitManage` oraz `EnforcementOperations`.
  - Dodano parametry zapytania do filtrowania wyników dla GET `/tokens`:
    - `description` - wyszukiwanie w opisie tokena (bez rozróżniania wielkości liter), min. 3 znaki,
    - `authorIdentifier` - wyszukiwanie po identyfikatorze twórcy (bez rozróżniania wielkości liter), min. 3 znaki,
    - `authorIdentifierType` - typ identyfikatora twórcy używany przy authorIdentifier (Nip, Pesel, Fingerprint).
  - Dodano właściwość 
    - `lastUseDate` - "Data ostatniego użycia tokena",
    - `statusDetails` - "Dodatkowe informacje na temat statusu, zwracane w przypadku błędów"  
    w odpowiedziach dla:
    - GET `/tokens` ("lista tokenów"),
    - GET `/tokens/{referenceNumber}` ("status tokena").

- **Pobieranie metadanych faktur (GET `/invoices/query/metadata`)**  
  - Filtry:
    - stronicowanie: zwiększono maksymalny rozmiar strony do 250 rekordów,
    - usunięto właściwość `schemaType` (z wartościami `FA1`, `FA2`, `FA3`), wcześniej oznaczoną jako deprecated,
    - dodano `seller.nip`; `seller.identifier` oznaczono jako deprecated (zostanie usunięte w następnym wydaniu),
    - dodano `authorizedSubject.nip`; `authorizedSubject.identifier` oznaczono jako deprecated (zostanie usunięte w następnym wydaniu),
    - doprecyzowano opis: brak wartości w `dateRange.to` oznacza użycie bieżącej daty i czasu (UTC),
    - doprecyzowano maksymalny dozwolony zakres `DateRange` na 2 lata.
  - Sortowanie:
    - wyniki są sortowane rosnąco według typu daty wskazanej w `DateRange`; do pobierania przyrostowego zalecany typ `PermanentStorage`,
  - Model odpowiedzi:
    - usunięto właściwość `totalCount`,
    - zmieniono nazwę `fileHash` na `invoiceHash`,
    - dodano `seller.nip`; `seller.identifier` oznaczono jako deprecated (zostanie usunięte w następnym wydaniu),
    - dodano `authorizedSubject.nip`; `authorizedSubject.identifier` oznaczono jako deprecated (zostanie usunięte w następnym wydaniu),
    - oznaczono `invoiceHash` jako zawsze zwracane,
    - oznaczono `invoicingMode` jako zawsze zwracane,
    - oznaczono `authorizedSubject.role` ("Podmiot upoważniony") jako zawsze zwracane,
    - oznaczono `invoiceMetadataAuthorizedSubject.role` ("Nip podmiotu upoważnionego") jako zawsze zwracane,
    - oznaczono `invoiceMetadataThirdSubject.role` ("Lista podmiotów trzecich") jako zawsze zwracane.
  - Usunięto oznaczenia [Mock] z opisów właściwości.

- **Eksport paczki faktur: zlecenie (POST `/invoices/exports`)**
  - Filtry:
    - dodano `seller.nip`; `seller.identifier` oznaczono jako deprecated (zostanie usunięte w następnym wydaniu),
  - Usunięto oznaczenia [Mock].
  - Zmieniono kod błędu: z `21180` na `21181` ("Nieprawidłowe żądanie eksportu faktur").
  - Doprecyzowano zasady sortowania. Faktury w paczce są sortowane rosnąco według typu daty wskazanego w `DateRange` podczas inicjalizacji eksportu.

  - **Eksport paczki faktur: status (GET `/invoices/exports/{operationReferenceNumber}`)**
    - Opisy statusów: uzupełniono dokumentację statusu eksportu:
      - `100` - "Eksport faktur w toku" 
      - `200` - "Eksport faktur zakończony sukcesem" 
      - `415` - "Błąd odszyfrowania dostarczonego klucza"  
      - `500` - "Nieznany błąd ({statusCode})"
    - Model odpowiedzi `package`:
      - dodano:
        - `invoiceCount` - "Łączna liczba faktur w paczce. Maksymalna liczba faktur w paczce to 10 000",
        - `size` - "Rozmiar paczki w bajtach. Maksymalny rozmiar paczki to 1 GiB (1 073 741 824 bajtów)",
        - `isTruncated` - "Określa, czy wynik eksportu został ucięty z powodu przekroczenia limitu liczby faktur lub wielkości paczki",
        - `lastIssueDate` - "Data wystawienia ostatniej faktury ujętej w paczce.\nPole występuje wyłącznie wtedy, gdy paczka została ucięta i eksport był filtrowany po typie daty `Issue`",
        - `lastInvoicingDate` - "Data przyjęcia ostatniej faktury ujętej w paczce.\nPole występuje wyłącznie wtedy, gdy paczka została ucięta i eksport był filtrowany po typie daty `Invoicing`",
        - `lastPermanentStorageDate` - "Data trwałego zapisu ostatniej faktury ujętej w paczce.\nPole występuje wyłącznie wtedy, gdy paczka została ucięta i eksport był filtrowany po typie daty `PermanentStorage`".
    - Model odpowiedzi `package.parts`
      - usunięto `fileName`, `headers`,
      - dodano:
        - `partName` - "Nazwa pliku części paczki",
        - `partSize` - "Rozmiar części paczki w bajtach. Maksymalny rozmiar części to 50MiB (52 428 800 bajtów)",
        - `partHash` - "Skrót SHA256 pliku części paczki, zakodowany w formacie Base64",
        - `encryptedPartSize` - "Rozmiar zaszyfrowanej części paczki w bajtach",
        - `encryptedPartHash` - "Skrót SHA256 zaszyfrowanej części paczki, zakodowany w formacie Base64",
        - `expirationDate` - "Moment wygaśnięcia linku do pobrania części",
      - oznaczono wszystkich właściwości w `package` jako zawsze zwracane,
    - Usunięto oznaczenia [Mock].

- **Uprawnienia**
  - Rozszerzono żądanie POST `/permissions/eu-entities/administration/grants` ("Nadanie uprawnień administratora podmiotu unijnego") o "Nazwę podmiotu" `subjectName`.
  - Rozszerzono żądanie POST `/permissions/query/persons/grants` o nową wartość `System` dla filtru identyfikatora podmiotu nadającego uprawnienia `authorIdentifier` i usunięcie wymagalność z pola `authorIdentifier.value`.
  - Rozszerzono żądanie POST `/permissions/query/persons/grants` o nową wartość `AllPartners` dla filtru identyfikator podmiotu docelowego `targetIdentifier` i usunięcie wymagalność z pola `targetIdentifier.value`.
  - Dodano żądanie POST `/permissions/query/personal/grants` do pobrania listy własnych uprawnień.
  - Dodano do żądania POST `/permissions/indirect/grants` ("Nadanie uprawnień w sposób pośredni") nową wartość `AllPartners` dla "identyfikatora podmiotu docelowego", oznaczającą uprawnienia generalne

- **Pobranie faktury (GET `/invoices/ksef/{ksefNumber}`)**  
   Dodano kod błędu dla odpowiedzi 400: `21165` - "Faktura o podanym numerze KSeF nie jest jeszcze dostępna".

- **Załączniki do faktur**  
  Dodano endpoint GET `/permissions/attachments/status` do sprawdzania statusu zgody na wystawianie faktur z załącznikiem.

- **Pobranie listy sesji**  
  Rozszerzono uprawnienia dla GET `/sessions`: dodano `InvoiceWrite`. Posiadając uprawnienie `InvoiceWrite`, można pobierać wyłącznie sesje utworzone przez podmiot uwierzytelniający; posiadając uprawnienie `Introspection`, można pobierać wszystkie sesje.

- **Sesja interaktywnej**  
  - Zaktualizowano kody błędów dla POST `/sessions/online/{referenceNumber}/invoices` ("Wysłanie faktury"):
    - usunięto `21154` - "Sesja interaktywna zakończona", 
    - dodano `21180` - "Status sesji nie pozwala na wykonanie operacji".
  - Dodano błąd `21180` - "Status sesji nie pozwala na wykonanie operacji" dla POST `/sessions/online/{referenceNumber}/close` ("Zamknięcie sesji interaktywnej").

- **Sesja wsadowa**  
  - Dodano błąd `21180` - "Status sesji nie pozwala na wykonanie operacji" dla POST `/sessions/batch/{referenceNumber}/close` ("Zamknięcie sesji wsadowej").

- **Status faktury w sesji**  
  Rozszerzono odpowiedź dla GET `/sessions/{referenceNumber}/invoices` ("Pobranie faktur sesji") oraz GET `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}` ("Pobranie statusu faktury z sesji") o właściwości:
  - `permanentStorageDate` – data trwałego zapisu faktury w repozytorium KSeF (od tego momentu faktura jest dostępna do pobrania),
  - `upoDownloadUrl` – adres do pobrania UPO.

- **OpenAPI**
  - Dodano uniwersalny kod błędu walidacji danych wejściowych `21405` do wszystkich endpointów. Treść błędu z walidatora zwracana w odpowiedzi.
  - Dodano odpowiedź 400 z walidacją zwracającą kod błędu `30001` („Podmiot lub uprawnienie już istnieje.”) dla POST `/testdata/subject` oraz POST `/testdata/person`.
  - Zaktualizowano przykłady (example) w definicjach endpointów.

- **Dokumentacja**
  - Doprecyzowano algorytmy podpisu i przykłady w [Kody QR](kody-qr.md).
  - Zaktualizowano przykłady kodu w C# i Javie.

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