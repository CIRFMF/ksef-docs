## Zmiany w API 2.0

### Wersja 2.0.0 RC5.6

- **Pobranie statusu sesji (GET `/sessions/{referenceNumber}`)**  
  Dodano w odpowiedzi pole `UpoPageResponse.downloadUrlExpirationDate` - data i godzina wygaśnięcia adresu pobrania UPO; po tym momencie `downloadUrl` nie jest już aktywny.

- **Pobranie listy metadanych certyfikatów (POST `/certificates/query`)**  
  Rozszerzono odpowiedź (`CertificateListItem`) o właściwość `requestDate` - data złożenia wniosku certyfikacyjnego.  

- **Pobranie listy dostawców usług Peppol (GET `/peppol/query`)**  
  - Rozszerzono model odpowiedzi o pole `dateCreated` - data rejestracji dostawcy usług Peppol w systemie.
  - Oznaczono właściwość `dateCreated`, `id`, `name` w modelu odpowiedzi jako zawsze zwracaną.
  - Zdefiniowano schemat `PeppolI` (string, 9 znaków) i zastosowano w `PeppolProvider`.

- **OpenAPI**  
  - Dodano metadane `x-sort` do wszystkich endpointów zwracających listy. W opisach endpointów dodano sekcję Sortowanie z domyślnym porządkiem (np. "requestDate (Desc)").
  - Dodano ograniczenia długości dla właściwości typu string: `minLength` oraz `maxLength`.
  - Doprecyzowano format właściwości reprezentujących dane binarne kodowane `Base64`: ustawiono format: `byte` (`encryptedInvoiceContent`, `encryptedSymmetricKey`, `initializationVector`, `encryptedToken`).
  - Zdefiniowano wspólny schemat `Sha256HashBase64` i zastosowano go do wszystkich właściwości reprezentujących skrót `SHA-256` w `Base64` (m.in. `invoiceHash`).
  - Zdefiniowano wspólny schemat `ReferenceNumber` (string, długość 36) i zastosowano go do wszystkich parametrów i właściwości reprezentujących numer referencyjny operacji asynchronicznej (w ścieżkach, zapytaniach i odpowiedziach).
  - Zdefiniowano wspólny schemat `Nip` (string, 10 znaków, regex pattern) i zastosowano go do wszystkich właściwości reprezentujących NIP.
  - Zdefiniowano schemat `Pesel` (string, 11 znaków, regex pattern) i zastosowano go we właściwości reprezentującej PESEL.
  - Zdefiniowano wspólny schemat `KsefNumber` (string, 35-36 znaków, regex pattern) i zastosowano go do wszystkich właściwości reprezentujących numer KSeF.  
  - Zdefiniowano schemat `Challenge` (string, 36 znaków) i zastosowano w `AuthenticationChallengeResponse`.`challenge`.
  - Zdefiniowano wspólny schemat `PermissionId` (string, 36 znaków) i zastosowano go we wszystkich miejscach: w parametrach oraz we właściwościach odpowiedzi.
  - Dodano wyrażenia regularne dla wybranych pól tekstowych.

### Wersja 2.0.0 RC5.5

- **Pobranie aktualnych limitów API (GET `/api/v2/rate-limits`)**  
  Dodano endpoint zwracający efektywne limity wywołań API w układzie `perSecond`/`perMinute`/`perHour` dla poszczególnych obszarów (m.in. `onlineSession`, `batchSession`, `invoiceSend`, `invoiceStatus`, `invoiceExport`, `invoiceDownload`, `other`).

- **Status faktury w sesji**  
  Rozszerzono odpowiedź dla GET `/sessions/{referenceNumber}/invoices` ("Pobranie faktur sesji") oraz GET `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}` ("Pobranie statusu faktury z sesji") o właściwości: `upoDownloadUrlExpirationDate` - "data i godzina wygaśnięcia adresu. Po tej dacie link `UpoDownloadUrl` nie będzie już aktywny". Rozszerzono opis `upoDownloadUrl`.

- **Usunięcie pól \*InMib (zmiana zgodna z zapowiedzią z 5.3)**  
  Usunięto właściwości `maxInvoiceSizeInMib` oraz `maxInvoiceWithAttachmentSizeInMib`.
  Zmiana dotyczy:
    - GET `/limits/context` – odpowiedzi (`onlineSession`, `batchSession`),
    - POST `/testdata/limits/context/session` – modelu żądania (`onlineSession`, `batchSession`),
    - Modeli: `BatchSessionContextLimitsOverride`, `BatchSessionEffectiveContextLimits`, `OnlineSessionContextLimitsOverride`, `OnlineSessionEffectiveContextLimits`.
  Do wskazywania rozmiarów używane są wyłącznie pola *InMB (1 MB = 1 000 000 B).

- **Usunięcie `operationReferenceNumber` (zmiana zgodna z zapowiedzią z 5.3)**  
  Usunięto właściwość `operationReferenceNumber` z modelu odpowiedzi; jedyną obowiązującą nazwą jest `referenceNumber`. Zmiana obejmuje:
  - GET `/invoices/exports/{referenceNumber}` - "Status eksportu paczki faktur",
  - POST `/permissions/operations/{referenceNumber}` - "Pobranie statusu operacji uprawnień".

- **Eksport paczki faktur (POST `/invoices/exports`)**  
  - Dodano nowy kod błędu: `21182` - "Osiągnięto limit trwających eksportów. Dla uwierzytelnionego podmiotu w bieżącym kontekście osiągnięto maksymalny limit {count} równoczesnych eksportów faktur. Spróbuj ponownie później".
  - Rozszerzono modelu odpowiedzi o właściwość `packageExpirationDate` wskazującą datę wygaśnięcia przygotowanej paczki. Po upływie tej daty paczka nie będzie dostępna do pobrania.
  - Dodano kod błędu `210` - "Eksport faktur wygasł i nie jest już dostępny do pobrania".

- **Status eksportu paczki faktur (GET `/invoices/exports/{referenceNumber}`)**  
  Doprecyzowano opisy pól linków do pobrania części paczki:
  - `url` - "Adres URL, pod który należy wysłać żądanie pobrania części paczki. Link jest generowany dynamicznie w momencie odpytania o status operacji eksportu. Nie podlega limitom API i nie wymaga przesyłania tokenu dostępowego przy pobraniu".
  - `expirationDate` - "Data i godzina wygaśnięcia linku umożliwiającego pobranie części paczki.Po upływie tego momentu link przestaje być aktywny".

- **Autoryzacja**
  - Rozszerzono reguły dostępu o `SubunitManage` dla POST `/permissions/query/persons/grants`: operację można wykonać, jeżeli podmiot posiada `CredentialsManage`, `CredentialsRead`, `SubunitManage`.
  - Nadanie uprawnień w sposób pośredni (POST `/permissions/indirect/grants`)
    Zaktualizowano opis właściwości `targetIdentifier.description`: doprecyzowano, że brak identyfikatora kontekstu oznacza nadanie uprawnienia pośredniego typu generalnego.

- **OpenAPI**  
  Zwiększono maksymalną wartość `pageSize` z 100 do 500 dla endpointów:
  - GET `/sessions`
  - GET `/sessions/{referenceNumber}/invoices`
  - GET `/sessions/{referenceNumber}/invoices/failed`

### Wersja 2.0.0 RC5.4

- **Pobranie listy metadanych faktur (POST /invoices/query/metadata)**  
  - Dodano parametr `sortOrder`, umożliwiający określenie kierunku sortowania wyników.

- **Status sesji**  
  Usunięto błąd uniemożliwiający uzupełnianie tej właściwości w odpowiedziach API dotyczących faktur (pole nie było wcześniej zwracane). Wartość jest uzupełniana asynchronicznie w momencie trwałego zapisu i może być tymczasowo null.

- **Dane testowe (tylko na środowiskach testowych)**
  - Zmiana limitów API dla bieżącego kontekstu (POST `testdata/rate-limits`)  
  Dodano endpoint umożliwiający tymczasowe nadpisanie efektywnych limitów API dla bieżącego kontekstu. Zmiana przygotowuje uruchomienie symulatora limitów na środowisku TE.
  - Przywrócenie domyślnych limitów (DELETE `/testdata/rate-limits`)
  Dodano endpoint przywracający domyślne wartości limitów dla bieżącego kontekstu.

- **OpenAPI**  
  - Doprecyzowano definicje parametrów tablicowych w query; zastosowano `style: form`. Wiele wartości należy przekazywać przez powtórzenie parametru, np. `?statuses=InProgress&statuses=Succeeded`. Zmiana dokumentacyjna, bez wpływu na działanie API.
  - Zaktualizowano opisy właściwości (`partUploadRequests`, `encryptedSymmetricKey`, `initializationVector`).

### Wersja 2.0.0 RC5.3

- **Eksport paczki faktur (POST `/invoices/exports`)**  
  Dodano możliwość dołączenia pliku `_metadata.json` do paczki eksportu. Plik ma postać obiektu JSON z tablicą `invoices`, zawierającą obiekty `InvoiceMetadata` (model zwracany przez POST `/invoices/query/metadata`).
  Włączenie (preview): do nagłówka żądania należy dodać `X-KSeF-Feature`: `include-metadata`.
  Od 2025-10-27 zmienia się domyślne zachowanie endpointu - paczka eksportu będzie zawsze zawierać plik `_metadata.json` (nagłówek nie będzie wymagany).

- **Status faktury**  
  - W przypadku przetworzenia z błędem, gdy możliwe było odczytanie numeru faktury (np. kod błędu `440` - duplikat faktury), odpowiedź zawiera właściwość `invoiceNumber` z odczytanym numerem.
  - Oznaczono właściwość `invoiceHash`, `referenceNumber` w modelu odpowiedzi jako zawsze zwracaną.

- **Standaryzacja jednostek rozmiaru (MB, SI)**  
  Ujednolicono zapis limitów w dokumentacji i API: wartości prezentowane w MB (SI), gdzie 1 MB = 1 000 000 B.

- **Pobranie limitów dla bieżącego kontekstu (GET `/limits/context`)**  
  Dodano w modelu odpowiedzi `maxInvoiceSizeInMB`, `maxInvoiceWithAttachmentSizeInMB` dla właściwości `onlineSession` i `batchSession`.
  Właściwości `maxInvoiceSizeInMib`, `maxInvoiceWithAttachmentSizeInMib` oznaczono jako deprecated (planowane usunięcie: 2025-10-27).

- **Zmiana limitów sesji dla bieżącego kontekstu (POST `/testdata/limits/context/session`)**  
  Dodano w modelu żądania `maxInvoiceSizeInMB`, `maxInvoiceWithAttachmentSizeInMB` dla właściwości `onlineSession` i `batchSession`.
  Właściwości `maxInvoiceSizeInMib`, `maxInvoiceWithAttachmentSizeInMib` oznaczono jako deprecated (planowane usunięcie: 2025-10-27).

- **Status eksportu paczki faktur (GET `/invoices/exports/{referenceNumber}`)**  
  Zmiana nazwy parametru ścieżki z `operationReferenceNumber` na `referenceNumber`.  
  Zmiana nie wpływa na kontrakt HTTP (ścieżka i znaczenie wartości bez zmian) ani na zachowanie endpointu.

- **Uprawnienia**  
  - Zaktualizowano opisy endpointów przykłady endpointów z obszaru permissions/*. Zmiana dotyczy wyłącznie dokumentacji (doprecyzowanie opisów, formatów i przykładów); brak zmian w zachowaniu API oraz kontrakcie.
  - Zmiana nazwy parametru ścieżki z `operationReferenceNumber` na `referenceNumber` w "Pobranie statusu operacji" (POST `/permissions/operations/{referenceNumber}`).  
  Zmiana nie wpływa na kontrakt HTTP (ścieżka i znaczenie wartości bez zmian) ani na zachowanie endpointu.
  - "Nadanie uprawnień w sposób pośredni" (POST `permissions/indirect/grants`)  
    Dodano obsługę identyfikatora wewnętrznego - rozszerzono właściwość `targetIdentifier` o wartość `InternalId`.
  - "Pobranie listy własnych uprawnień" (POST `/permissions/query/personal/grants`)  
      - Rozszerzono w modelu żądania właściwość `targetIdentifier` o wartość `InternalId` (możliwość wskazania identyfikatora wewnętrznego).
      - Usunięto w modelu odpowiedzi wartość `PersonalPermissionScope.Owner`. Uprawnienia właścicielskie (nadawane przez ZAW-FA lub powiązanie NIP/PESEL) nie są zwracane.

- **Status uwierzytelniania (GET `/auth/{referenceNumber}`)**  
  Rozszerzono tabelę kodów błędów o `470` - "Uwierzytelnianie zakończone niepowodzeniem" z doprecyzowaniem: "Próba wykorzystania metod autoryzacyjnych osoby zmarłej".

- **Obsługa faktur PEF**  
  Zmieniono wartości enuma (`FormCode`):
    - `FA_PEF (3)` na `PEF (3)`,
    - `FA_KOR_PEF (3)` na `PEF_KOR (3)`.

- **Wygenerowanie nowego tokena (POST `/tokens`)**  
  - W modelu żądania (`GenerateTokenRequest`) oznaczono pola `description` i `permissions` jako wymagane.
  - W modelu odpowiedzi (`GenerateTokenResponse`) oznaczono pola `referenceNumber` i `token` jako zawsze zwracane.

- **Statusu tokena KSeF (GET /tokens/{referenceNumber})**
  - Oznaczono właściwość `authorIdentifier`, `contextIdentifier`, `dateCreated`, `description`, `referenceNumber`, `requestedPermissions`, `status` w modelu odpowiedzi jako zawsze zwracaną.

- **Pobranie listy wygenerowanych tokenów (GET /tokens)**
  - Oznaczono właściwość `authorIdentifier`, `contextIdentifier`, `dateCreated`, `description`, `referenceNumber`, `requestedPermissions`, `status` w modelu odpowiedzi jako zawsze zwracaną.

- **Dane testowe - utworzenie osoby fizycznej (POST `/testdata/person`)**  
  Rozszerzono żądanie o właściwość `isDeceased` (boolean) umożliwiając utworzenie testowej osoby zmarłej (np. do scenariuszy weryfikujących kod statusu `470`).

- **OpenAPI**
  - Doprecyzowano ograniczenia dla właściwości typu integer w requests poprzez dodanie atrybutów `minimum` / `exclusiveMinimum`, `maximum` / `exclusiveMaximum`.  
  - Rozszerzono odpowiedź o pole `referenceNumber` (zawiera tę samą wartość, co dotychczasowe `operationReferenceNumber`). Oznaczono `operationReferenceNumber` jako `deprecated` i zostanie usunięte z odpowiedzi 2025-10-27; należy przejść na `referenceNumber`. Charakter zmiany: przejściowy rename z zachowaniem kompatybilności (obie właściwości zwracane równolegle do daty usunięcia).  
  Dotyczy endpointów:
    - POST `/permissions/persons/grants`,
    - POST `/permissions/entities/grants`,
    - POST `/permissions/authorizations/grants`,
    - POST `/permissions/indirect/grants`,
    - POST `/permissions/subunits/grants`,
    - POST `/permissions/eu-entities/administration/grants`,
    - POST `/permissions/eu-entities/grants`,
    - DELETE `/permissions/common/grants/{permissionId}`,
    - DELETE `/permissions/authorizations/grants/{permissionId}`,
    - POST `/invoices/exports`.
  - Usunięto atrybut `required` z właściwości `pageSize` w żądaniu GET `/sessions` ("Pobranie listy sesji").
  - Zaktualizowano przykłady (example) w definicjach endpointów.

### Wersja 2.0.0 RC5.2
- **Uprawnienia** 
  - "Nadanie uprawnień administratora podmiotu podrzędnego" (POST `/permissions/subunits/grants`)  
  Dodano właściwość `subunitName` ("Nazwa jednostki podrzędnej") w żądaniu. Pole jest wymagane, gdy jednostka podrzędna identyfikowana jest identyfikatorem wewnętrznym.
  - "Pobranie listy uprawnień administratorów jednostek i podmiotów podrzędnych" (POST `/permissions/query/subunits/grants`)  
  Dodano w odpowiedzi właściwość `subunitName` ("Nazwa jednostki podrzędnej").
  - "Pobranie listy uprawnień do pracy w KSeF nadanych osobom fizycznym lub podmiotom" (POST `permissions/query/persons/grants`)  
    Usunięto z wyników uprawnienie typu `Owner`. Uprawnienie `Owner` jest przypisywane systemowo do osoby fizycznej i nie podlega nadawaniu, więc nie powinno pojawiać się na liście nadanych uprawnień.
  - "Pobranie listy własnych uprawnień" (POST `/permissions/query/personal/grants`)  
    Rozszerzono enum filtra `PersonalPermissionType` o wartość `VatUeManage`.

- **Limity**  
  - Dodano endpointy do sprawdzania ustawionych limitów (kontekst, podmiot uwierzytelniony):
    - GET `/limits/context`
    - GET `/limits/subject`
  - Dodano endpointy do zarządzania limitami (kontekst, podmiot uwierzytelniony) w środowisku testowym:
    - POST/DELETE `/testdata/limits/context/session`
    - POST/DELETE `/testdata/limits/subject/certificate`
  - Zaktualizowano [Limity](limity/limity.md).

- **Status faktury**  
  Dodano właściwość `invoicingMode` w modelu odpowiedzi. Zaktualizowano dokumentację: [Automatyczne określanie trybu wysyłki offline](offline/automatyczne-okreslanie-trybu-offline.md).

- **OpenAPI**
  - Doprecyzowano ograniczenia dla właściwości typu integer w requests poprzez dodanie atrybutów `minimum` / `exclusiveMinimum`, `maximum` / `exclusiveMaximum` oraz wartości domyślnych `default`.
  - Zaktualizowano przykłady (example) w definicjach endpointów.
  - Doprecyzowano opisy endpointów.
  - Dodano atrybut `required` dla wymaganych właściwości w żądaniach i odpowiedziach.

### Wersja 2.0.0 RC5.1

- **Pobranie listy metadanych certyfikatów (POST /certificates/query)**  
  Zmieniono reprezentację identyfikatora podmiotu z pary właściwości `subjectIdentifier` + `subjectIdentifierType` na obiekt złożony `subjectIdentifier` { `type`, `value` }.

- **Pobranie listy metadanych faktur (POST /invoices/query/metadata)**
  - Zmieniono reprezentację wybranych identyfikatorów z par właściwości typ + value na obiekty złożone { type, value }: 
    - `invoiceMetadataBuyer.identifier` + `invoiceMetadataBuyer.identifierType` na obiekt złożony `invoiceMetadataBuyerIdentifier` { `type`, `value` },
    - `invoiceMetadataThirdSubject.identifier` + `invoiceMetadataThirdSubject.identifierType` na obiekt złożony `InvoiceMetadataThirdSubjectIdentifier` { `type`, `value` }.
  - Usunięto `obsoleted` właściwości `Identitifer` z obiektów  `InvoiceMetadataSeller` oraz `InvoiceMetadataAuthorizedSubject`.
  - Zmieniono właściwość `invoiceQuerySeller` na `sellerNip` w filtrze żądania.
  - Zmieniono właściwość `invoiceQueryBuyer` na `invoiceQueryBuyerIdentifier` z właściwościami { `type`, `value` } w filtrze żądania.

- **Uprawnienia**  
  Zmieniono reprezentację wybranych identyfikatorów z par właściwości typ + value na obiekty złożone { type, value }: 
    - "Pobranie listy własnych uprawnień" (POST `/permissions/query/personal/grants`):  
      - `contextIdentifier` + `contextIdentifierType` -> `contextIdentifier` { `type`, `value` },
      - `authorizedIdentifier` + `authorizedIdentifierType` -> `authorizedIdentifier` { `type`, `value` },
      - `targetIdentifier` + `targetIdentifierType` -> `targetIdentifier` { type, value }.  
    - "Pobranie listy uprawnień do pracy w KSeF nadanych osobom fizycznym lub podmiotom" (POST `/permissions/query/persons/grants`),
      - `contextIdentifier` + `contextIdentifierType` -> `contextIdentifier` { `type`, `value` },
      - `authorizedIdentifier` + `authorizedIdentifierType` -> `authorizedIdentifier` { `type`, `value` },
      - `targetIdentifier` + `targetIdentifierType` -> `targetIdentifier` { `type`, `value` },
      - `authorIdentifier` + `authorIdentifierType` -> `authorIdentifier` { `type`, `value` }.    
    - "Pobranie listy uprawnień administratorów jednostek i podmiotów podrzędnych" (POST `/permissions/query/subunits/grants`):
      - `authorizedIdentifier` + `authorizedIdentifierType` -> `authorizedIdentifier` { `type`, `value` },
      - `subunitIdentifier` + `subunitIdentifierType` -> `subunitIdentifier` { `type`, `value` },
      - `authorIdentifier` + `authorIdentifierType` -> `authorIdentifier` { `type`, `value` }.
    - "Pobranie listy ról podmiotu" (POST `/permissions/query/entities/roles`):
      - `parentEntityIdentifier` + `parentEntityIdentifierType` -> `parentEntityIdentifier` { `type`, `value` }.
    - "Pobranie listy podmiotów podrzędnych" (POST `/permissions/query/subordinate-entities/roles`):
      - `subordinateEntityIdentifier` + `subordinateEntityIdentifierType` -> `subordinateEntityIdentifier` { `type`, `value` }.
    - "Pobranie listy uprawnień podmiotowych do obsługi faktur" (POST `/permissions/query/authorizations/grants`):
      - `authorizedEntityIdentifier` + `authorizedEntityIdentifierType` -> `authorizedEntityIdentifier` { `type`, `value` },
      - `authorizingEntityIdentifier` + `authorizingEntityIdentifierType` -> `authorizingEntityIdentifier` { `type`, `value` },
      - `authorIdentifier` + `authorIdentifierType` -> `authorIdentifier` { `type`, `value` }
    - "Pobranie listy uprawnień administratorów lub reprezentantów podmiotów unijnych uprawnionych do samofakturowania" (POST `/permissions/query/eu-entities/grants`):
      - `authorIdentifier` + `authorIdentifierType` -> `authorIdentifier` { `type`, `value` }        

- **Nadanie uprawnień administratora podmiotu unijnego (POST permissions/eu-entities/administration/grants)**  
  Zmieniono nazwę właściwości w żądaniu z `subjectName` na `euEntityName`.

- **Uwierzytelnienie z wykorzystaniem tokena KSeF**  
  Usunięto nadmiarowe wartości enum `None`, `AllPartners` we właściwości `contextIdentifier.type` żądania POST `/auth/ksef-token`.

- **Tokeny KSeF**  
  - Ujednoznaczniono model odpowiedzi GET `/tokens`: właściwości `authorIdentifier.type`, `authorIdentifier.value`, `contextIdentifier.type`, `contextIdentifier.value` są zawsze zwracane (required, non-nullable),
  - Usunięto nadmiarowe wartości enum `None`, `AllPartners` we właściwościach `authorIdentifier.type` oraz `contextIdentifier.type` w modelu odpowiedzi GET `/tokens` ("Pobranie listy wygenerowanych tokenów").

- **Sesja wsadowa**  
  Usunięto nadmiarowy kod błędu `21401`	- "Dokument nie jest zgodny ze schemą (json)".

- **Pobranie statusu sesji (GET /sessions/{referenceNumber})**  
  - Dodano kod błędu `420` - "Przekroczony limit faktur w sesji".

- **Pobieranie metadanych faktur (GET `/invoices/query/metadata`)**  
  - Dodano w odpowiedzi (zawsze zwracaną) właściwość `isTruncated` (boolean) – "Określa, czy wynik został obcięty z powodu przekroczenia limitu liczby faktur (10 000)",
  - Oznaczono właściwość `amount.type` w filtrze żądania jako wymaganą.

- **Eksport paczki faktur: zlecenie (POST `/invoices/exports`)**
  - Oznaczono właściwość `operationReferenceNumber` w modelu odpowiedzi jako zawsze zwracaną,
  - Oznaczono właściwość `amount.type` w filtrze żądania jako wymaganą.

- **Pobranie listy uprawnień do pracy w KSeF nadanych osobom fizycznym lub podmiotom (POST /permissions/query/persons/grants)**  
  - Dodano `contextIdentifier` w filtrze żądania i w modelu odpowiedzi.

- **OpenAPI**  
  Usunięto nieużywany `operationId` ze specyfikacji. Zmiana porządkująca.

### Wersja 2.0.0 RC5

- **Obsługa faktur PEF i dostawców usług Peppol**
  - Dodano obsługę faktur `PEF` wysyłanych przez dostawcę usług Peppol. Nowe możliwości nie zmieniają dotychczasowych zachowań KSeF dla innych formatów, są rozszerzeniem API.
  - Wprowadzono nowy typ kontekstu uwierzytelniania: `PeppolId`, umożliwiający pracę w kontekście dostawcy usług Peppol.
  - Automatyczna rejestracja dostawcy: przy pierwszym uwierzytelnieniu dostawcy usług Peppol (z użyciem dedykowanego certyfikatu) następuje jego automatyczna rejestracja w systemie.
  - Dodano endpoint GET `/peppol/query` ("Lista dostawców usług Peppol") zwracający zarejestrowanych dostawców.
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
  Dodano specyfikację [limitów żądań API](limity/limity-api.md).    

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
  - Zmieniono nazwę modelu `InvoiceMetadataQueryRequest` na `QueryInvoicesMetadataResponse`.
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
  - Dodano nowe kody błędów:
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

- **Certyfikaty KSeF - Walidacja typu i długości klucza w CSR**  
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