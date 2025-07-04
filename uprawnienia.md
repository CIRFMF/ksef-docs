## Uprawnienia
28.06.2025

### Wprowadzenie – kontekst biznesowy
System Krajowego Systemu e-Faktur (KSeF) wprowadza zaawansowany mechanizm zarządzania uprawnieniami, który stanowi fundament bezpiecznego i zgodnego z przepisami korzystania z systemu przez różne podmioty. Uprawnienia decydują o tym, kto może wykonywać określone operacje w KSeF – takie jak wystawianie faktur, przeglądanie dokumentów, nadawanie dalszych uprawnień czy zarządzanie jednostkami podrzędnymi.

### Cele zarządzania uprawnieniami:
- Bezpieczeństwo danych – ograniczenie dostępu do informacji tylko do osób i podmiotów, które są do tego formalnie uprawnione.
- Zgodność z przepisami – zapewnienie, że operacje są wykonywane przez właściwe jednostki zgodnie z wymogami ustawowymi (np. Ustawa VAT).
- Audytowalność – każda operacja związana z nadaniem lub odebraniem uprawnień jest rejestrowana i może być poddana analizie.

### Kto nadaje uprawnienia?
Uprawnienia mogą być nadawane przez:

- właściciela podmiotu (rola Owner),
- administratora podmiotu podrzędnego,
- administratora jednostki podrzędnej,
- administratora podmiotu unijnego,
- inny podmiot lub osobę posiadającą uprawnienie "CredentialsManage".

W praktyce oznacza to, że każda organizacja musi zarządzać uprawnieniami swoich pracowników np. nadając uprawnienia pracownikowi działu księgowości podczas onboardingu lub też odbierając uprawnienia gdy taki pracownik kończy stosunek pracy.

### Kiedy nadaje się uprawnienia?
#### Przyklady:  
- podczas rozpoczęcia współpracy z nowym pracownikiem,
- w przypadku gdy firma wchodzi we współpracę np. z biurem rachunkowym powinna nadać uprawnienia do odczytu faktur na to biuro rachunkowe, aby to biuro mogło rozliczać faktury tej firmy,
- w zwiazku ze zmianami relacji pomiedzy podmiotami.

### Struktura nadanych uprawnień:
Uprawnienia są przypisywane:
- osobom fizycznym – identyfikowanym przez PESEL, NIP lub fingerprint certyfikatu,
- podmiotom gospodarczym – identyfikowanym przez NIP lub NIP VAT UE lub Identyfikatorem wewnętrznym,
- pośrednikom i reprezentantom – działającym w kontekście innego podmiotu.

Dostęp do funkcji systemu zależy od typu nadanego uprawnienia oraz kontekstu, w którym jest ono realizowane (np. podmiot główny, podmiot podrzędny, jednostka unijna).

##  Słowniczek pojęć (dot. uprawnień KSeF)

| Termin                          | Definicja |
|---------------------------------|-----------|
| **Uprawnienie**                 | Zezwolenie na wykonanie określonych operacji w KSeF, np. `InvoiceWrite`, `CredentialsManage`. |
| **Właściciel**                       | Właściciel podmiotu – domyślny pełen dostęp do operacji w ramach danego NIP. |
| **Administrator podmiotu podrzędnego**              | Osoba z uprawnieniami do zarządzania uprawnieniami (`CredentialsManage`) w kontekście podmiotu podrzędnego. Może nadawać uprawnienia (np. `InvoiceWrite`). Podmiotem podrzędnym może być np. członek grupy VAT. |
| **Administrator jednostki podrzędnej**              | Osoba z uprawnieniami do zarządzania uprawnieniami  (`CredentialsManage`) w jednostce podrzędnej. Może nadawać uprawnienia (np. `InvoiceWrite`). |
| **Administrator podmiotu unijnego**              | Osoba z uprawnieniami do zarządzania uprawnieniami (`CredentialsManage`) w kontekście złożonym identyfikowanym za pomocą NipVatUe. Może nadawać uprawnienia (np. `InvoiceRead`). |
| **Podmiot pośredniczący**   | Podmiot, który otrzymał uprawnienie z flagą `canDelegate = true` i może przekazać to uprawnienie dalej. Mogą to być tylko uprawnienia `InvoiceWrite` i `InvoiceRead`. |
| **Podmiot docelowy**  | Podmiot, w którego kontekście obowiązuje dane uprawnienie – np. firma, dla której biuro rachunkowe przegląda faktury. |
| **Nadane w sposób bezpośredni**       | Uprawnienie nadane wprost danemu użytkownikowi lub podmiotowi przez właściciela lub administratora. |
| **Nadanie w sposób pośredni**          | Uprawnienie nadane przez pośrednika w imieniu innego podmiotu – tylko dla `InvoiceRead` i `InvoiceWrite`. |
| **`canDelegate`**              | Flaga techniczna (`true` / `false`) pozwalająca na delegowanie uprawnień. Tylko `InvoiceRead` oraz `InvoiceWrite` mogą mieć `canDelegate = true`. Może być wykorzystana tylko podczas nadawania uprawnienia podmiotowi do obsługi faktur |
| **`subjectIdentifier`**        | Dane identyfikujące odbiorcę uprawnień (osobę lub podmiot): `Nip`, `Pesel`, `Fingerprint`. |
| **`targetIdentifier` / `contextIdentifier`** | Dane identyfikujące kontekst, w którym działa uprawnienie – np. NIP klienta, Identyfikator wewnętrzny jednostki organizacyjnej. |
| **Fingerprint**                | Skrót (hash) liczony algorytmem SHA-256, identyfikujący certyfikat kwalifikowany lub podpis elektroniczny. Używany m.in. w identyfikacji osób lub podmiotów zagranicznych. |
| **InternalId**                 | Wewnętrzny identyfikator jednostki podrzędnej w systemie KSeF, dwuczłonowy identyfikator składający sie z nr NIP oraz pięciu cyfr `nip-5_cyfr`.  |
| **NipVatUe**                   | Identyfikator złożony, czyli dwuczłonowy identyfikator składający się z nr NIP podmiotu polskiego oraz nr VAT UE podmiotu unijnego, które są oddzielone za pomocą separatora `nip-vat_ue`. |
| **Odbieranie**                     | Operacja odebrania wcześniej nadanego uprawnienia (np. przez API `DELETE /permissions/...`). |
| **`permissionId`**             | Techniczny identyfikator nadanego uprawnienia – wymagany m.in. przy operacjach odebrania. |
| **`operationReferenceNumber`** | Identyfikator operacji (np. nadania lub odebrania uprawnień), zwracany przez API, wykorzystywany do sprawdzenia statusu. |
| **Status operacji**            | Bieżący stan procesu nadania/odebrania uprawnień: `100`, `200`, `400` itp. |

## Model ról i uprawnień (macierz uprawnień)

System KSeF umożliwia przypisywanie uprawnień w sposób precyzyjny, z uwzględnieniem różnych typów użytkowników i ich roli w strukturze organizacyjnej lub procesowej. Uprawnienia mogą być nadawane zarówno bezpośrednio, jak i pośrednio – w zależności od mechanizmu delegowania dostępu.

### Przykłady ról do odwzorowania za pomocą uprawnień:

| Rola / podmiot                          | Opis roli                                                                                          | Możliwe uprawnienia                                                                 |
|----------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| **Właściciel podmiotu** (`Owner`)      | Rola posiadana domyślnie z automatu przez właściciela. Aby być rozpoznanym przez system jako właściciel należy się uwierzytelnić wektorem z takim samym identyfikatorem NIP jak NIP kontekstu logowania lub powiązanym numerem PESEL           | Wszystkie uprawnienia oprócz `VatUeManage`, w tym `CredentialsManage`, `SubunitManage`, `InvoiceWrite`, `InvoiceRead`, `Introspection`. |
| **Administrator podmiotu**            | Osoba fizyczna posiadająca prawa do nadawania i odbierania uprawnień innym użytkownikom.           | `CredentialsManage`, `SubunitManage`, `Introspection`.                              |
| **Operator (księgowość / fakturowanie)** | Osoba odpowiedzialna za wystawianie lub przeglądanie faktur.                                        | `InvoiceWrite`, `InvoiceRead`.                                                      |
| **Podmiot upoważniony**                | Inny podmiot gospodarczy, któremu nadano określone uprawnienia podmiotowe do obsługi faktur – np. biuro rachunkowe.             | `InvoiceRead`, `InvoiceWrite`, `SelfInvoicing`, itp.                                |
| **Podmiot pośredniczący**              | Podmiot, który otrzymał uprawnienia z opcją delegacji (`canDelegate`) i może nadać je dalej.       | `InvoiceRead`, `InvoiceWrite` z flagą `canDelegate = true`.   
| **Administrator podmiotu unijnego**     | Osoba identyfikująca się certyfikatem  posiadająca prawa do nadawania i odbierania uprawnienń innym użytkownikom w ramach podmiotu unijnego (UE).                                     | `InvoiceWrite`, `InvoiceRead`,                                    `VatUeManage`,  `Introspection`.                      |                      |
| **Reprezentant podmiotu unijnego**     | Osoba identyfikująca się certyfikatem działająca na rzecz podmiotu unijnego (UE).                                     | `InvoiceWrite`, `InvoiceRead`.                                                      |
| **Administrator jednostki podrzędnej** | Użytkownik zarządzający dostępem w jednostkach i podmiotach podrzędnych.               | `CredentialsManage`.                                                                    |

---

### Klasyfikacja uprawnień według rodzaju:

| Typ uprawnienia           | Przykładowe wartości                                       | Możliwość nadania w sposób pośredni | Opis operacyjny                                                              |
|--------------------------|------------------------------------------------------------|-------------------------------|------------------------------------------------------------------------------|
| **Fakturowe**             | `InvoiceWrite`, `InvoiceRead`                              | ✔️ (jeśli `canDelegate=true`) | Operacje na fakturach: wysyłanie, pobieranie                     |
| **Administracyjne**       | `CredentialsManage`, `SubunitManage`,  `VatUeManage`.                       | ❌                            | Zarządzanie uprawnieniami, jednostkami                       |
| **Podmiotowe**        | `SelfInvoicing`, `RRInvoicing`, `TaxRepresentative`        | ❌                            | Specjalne typy operacji (np. fakturowanie RR lub w imieniu innych)         |
| **Techniczne**            | `Introspection`                                            | ❌                            | Dostęp do historii operacji i sesji                                         |

---

## Uprawnienia ogólne i selektywne

System KSeF umożliwia nadawanie wybranych uprawnień w sposób **ogólny (generalny)** lub **selektywny (indywidualny)**, co pozwala elastycznie zarządzać dostępem do danych wielu partnerów biznesowych.

###  Uprawnienia selektywne (indywidualne)

Uprawnienia selektywne to takie, które nadawane są przez podmiot pośredniczący (np. biuro rachunkowe) w odniesieniu do **konkretnego podmiotu docelowego (partnera)**. Pozwalają ograniczyć zakres dostępu tylko do wybranego klienta lub jednostki organizacyjnej.

**Przykład:**  
Biuro rachunkowe XYZ otrzymało od firmy ABC uprawnienie `InvoiceRead` z flagą `canDelegate = true`. Teraz może przekazać to uprawnienie swojemu pracownikowi, ale tylko w kontekście firmy ABC – inne firmy obsługiwane przez XYZ nie są objęte tym dostępem.

**Cechy selektywności:**
- Konieczne jest wskazanie `targetIdentifier` (np. `Nip` partnera).
- Odbiorca uprawnienia działa tylko w kontekście wskazanego podmiotu.
- Nie daje dostępu do danych innych partnerów podmiotu pośredniczącego.

---

###  Uprawnienia ogólne (generalne)

Uprawnienia ogólne to takie, które nadawane są bez wskazania konkretnego partnera, co oznacza, że odbiorca zyskuje dostęp do operacji w kontekście **wszystkich podmiotów, których dane przetwarza podmiot pośredniczący**.

**Przykład:**  
Podmiot A posiada uprawnienie `InvoiceRead` z `canDelegate = true` dla wielu klientów. Przekazuje pracownikowi B ogólne uprawnienie `InvoiceRead` – B może teraz działać w imieniu każdego z klientów A (np. przeglądać faktury wszystkich kontrahentów).

**Cechy generalności:**
- `targetIdentifier`  pozostaje pusty.
- Dostęp obejmuje wszystkie podmioty obsługiwane przez pośrednika.
- Stosowane w przypadku integracji masowej, dużych centrów usług wspólnych lub systemów księgowych.

---

### Uwagi techniczne i ograniczenia

- Mechanizm dotyczy tylko uprawnień `InvoiceRead` i `InvoiceWrite` nadawanych w sposób pośredni.
- W praktyce różnica polega na obecności (selektywne) lub braku (ogólne) podmiotu `targetIdentifier` w treści zapytania `POST /permissions/indirect/grants`.
- System nie pozwala połączyć w jednym wywołaniu nadania ogólnego i selektywnego – należy wykonać osobne operacje.
- Uprawnienia ogólne powinny być stosowane z ostrożnością, szczególnie w środowiskach produkcyjnych, z uwagi na ich szeroki zakres.

---

### Struktura przypisywania uprawnień:

1. **Bezpośrednie nadanie** – np. administrator przypisuje użytkownikowi uprawnienie `InvoiceWrite` w kontekście podmiotu A.
2. **Nadanie z możliwością dalszego przekazania** – np. podmiot A nadaje podmiotowi B uprawnienie `InvoiceRead` z `canDelegate=true`, co umożliwia B nadanie `InvoiceRead` podmiotowi C.
3. **Nadanie w sposób pośredni** – z użyciem dedykowanego endpointu `/permissions/indirect/grants`, gdzie podmiot pośredniczący nadaje uprawnienia w imieniu podmiotu docelowego.

---

### Przykład macierzy uprawnień:

| Użytkownik / Podmiot       | InvoiceWrite | InvoiceRead | CredentialsManage | SubunitManage | TaxRepresentative |
|----------------------------|--------------|-------------|--------------------|----------------|--------------------|
| Anna Kowalska (PESEL)      | ✅           | ✅          | ❌                 | ❌             | ❌                 |
| Biuro Rachunkowe XYZ (NIP) | ✅ (z delegacją)          | ✅ (z delegacją) | ❌                 | ❌             | ❌                 |
| Jan Nowak (Identyfikujacy sie certyfikatem)   | ✅           | ✅          | ❌                 | ❌             | ❌                 |
| Admin IT (PESEL)           | ❌           | ❌          | ✅                 | ✅             | ❌                 |
| Spółka Matka tj. owner (NIP)         | ✅           | ✅          | ✅                 | ✅             | ✅                 |

## Ograniczenia dotyczące delegowania uprawnień

| Uprawnienie           | Możliwość delegowania (`canDelegate`) | Uwagi                                                                 |
|-----------------------|----------------------------------------|-----------------------------------------------------------------------|
| `InvoiceRead`         | ✅ Tak                                 | Uprawnienie, które może być delegowane z `canDelegate = true` |
| `InvoiceWrite`        | ✅ Tak                           | Uprawnienie, które może być delegowane z `canDelegate = true` |
| `CredentialsManage`   | ❌ Nie                                  | Uprawnienie do zarządzania uprawnieniami – nie można przekazać                      |
| `SubunitManage`       | ❌ Nie                                  | Uprawnienie dla administratorów podmiotów podrzędnych i jednostek podrzędnych - nie można przekazać                                        |
| `SelfInvoicing`, `TaxRepresentative`, `RRInvoicing` | ❌ Nie                        | Tylko bezpośrednie nadanie przez właściciela lub uprawnionego         |

---

### Ograniczenia ról nadających uprawnienia

| Rodzaj operacji                            | Wymagana rola lub uprawnienie                      |
|-------------------------------------------|---------------------------------------------------|
| Nadanie uprawnienia osobie fizycznej      | `Owner` lub `CredentialsManage`                   |
| Nadanie uprawnienia podmiotowi            | `Owner` lub `CredentialsManage`                   |
| Nadanie uprawnienia podmiotowego | `Owner` lub `CredentialsManage`                   |
| Nadanie uprawnień w sposób pośredni              | `Owner` lub `CredentialsManage` w kontekscie podmiotu ktory otrzymał uprawnienie z flaga  `canDelegate = true`    |
| Nadanie uprawnień jednostce podrzędnej lub podmiotowi podrzędnemu   | `SubunitManage`                                   |
| Nadanie uprawnień administratorowi UE      | `Owner` lub `CredentialsManage`    |
| Nadanie uprawnień reprezentantowi UE      | Tylko administrator VAT UE posiadający `VatUeManage`    |
---

### Ograniczenia identyfikatorów (`subjectIdentifier`, `contextIdentifier`)

| Typ identyfikatora | Identyfikowany | Uwagi |
|--------------------|---------------------|-------|
| `Nip`              | Podmioty krajowe     | Dla firm zarejestrowanych w Polsce oraz osób fizycznych |
| `Pesel`            | Osoby fizyczne       | Wymagane m.in. przy nadaniu uprawnień pracownikom |
| `Fingerprint`      | Podmiot lub osoba fizyczna      | Wykorzystywany w przypadku gdy certyfikat nie posiada atrybutu NIP/PESEL oraz podczas nadawania uprawnień administratora podmiotu unijnego.   |
| `NipVatUe`         | Podmioty unijne      | Wymagane np. przy nadaniu administratora UE |
| `InternalId`       | Jednostki podrzędne  | - |

---

### Ograniczenia funkcjonalne API

- Nie można nadać tego samego uprawnienia dwukrotnie – API może zwrócić błąd lub zignorować duplikat.
- Nadanie uprawnienia nie skutkuje natychmiastowym dostępem – operacja musi zostać przetworzona przez system (sprawdź status operacji).
- Odebranie uprawnienia nie cofa skutków wykonanych wcześniej operacji (np. wystawione faktury pozostają w systemie).

---

### Ograniczenia czasowe

- Nadane uprawnienie pozostaje aktywne do momentu ich odebrania.
- Wdrożenie ograniczenia czasowego wymaga logiki po stronie systemu klienta (np. harmonogram odebrania uprawnienia).


## Nadawanie uprawnień


### Nadawanie uprawnień osobom fizycznym do pracy w KSeF.

W ramach organizacji korzystających z KSeF możliwe jest nadanie uprawnień konkretnym osobom fizycznym – np. pracownikom działu księgowości lub IT. Uprawnienia są przypisywane do osoby na podstawie jej identyfikatora (PESEL, NIP lub Fingerprint). Uprawnienia mogą obejmować zarówno działania operacyjne (np. wystawianie faktur), jak i administracyjne (np. zarządzanie uprawnieniami). Sekcja opisuje sposób nadania takich uprawnień za pomocą API oraz wymagania uprawnieniowe po stronie nadającego.

POST [/permissions/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1persons~1grants/post)


| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator podmiotu lub osoby fizycznej. `"Nip"`, `"Pesel"`, `"Fingerprint"`             |
| `permissions`                               | Uprawnienia do nadania. `"CredentialsManage"`, `"CredentialsRead"`, `"InvoiceWrite"`, `"InvoiceRead"`, `"Introspection"`, `"SubunitManage"`, `"EnforcementOperations"`		   |
| `description`                              | Wartość tekstowa (opis)              |
 

Lista uprawnień, które mogą zostać nadane osobie fizycznej:


| Uprawnienie | Opis |
| :------------------ | :---------------------------------- |
| `CredentialsManage` | Zarządzanie uprawnieniami |
| `CredentialsRead` | Przeglądanie uprawnień |
| `InvoiceWrite` | Wystawianie faktur |
| `InvoiceRead` | Przeglądanie faktur |
| `Introspection` | Przeglądanie historii sesji |
| `SubunitManage` | Zarządzanie jednostkami podrzędnymi |
| `EnforcementOperations` | Wykonywanie operacji egzekucyjnych |

 


Przykład w języku C#:

```csharp
 GrantPermissionsRequest request = GrantPersonPermissionsRequestBuilder
     .Create()
     .WithSubject(subjectIdentifier)
     .WithPermissions(StandardPermissionType.InvoiceRead, StandardPermissionType.InvoiceWrite)
     .WithDescription("Access for quarterly review")
     .Build();

 return await ksefClient.GrantsPermissionPersonAsync(request,  accessToken, cancellationToken);

```

Przykład w języku Java:

```java
var request = new PersonPermissionsGrantRequestBuilder()
        .withSubjectIdentifier(subjectIdentifier)
        .withPermissions(List.of(PersonPermissionType.INVOICEREAD, PersonPermissionType.INVOICEWRITE))
        .withDescription("Access for quarterly review")
        .build();

return ksefClient.grantsPermissionPerson(request);
```

Uprawnienia może nadawać ktoś kto jest:
- właścicielem
- posiada uprawnienie `CredentialsManage`
- administratorem jednostek podrzędnych, który posiada `SubunitManage`
- administratorem podmiotu unijnego, który posiada `VatUeManage`


---
### Nadanie podmiotom uprawnień do obsługi faktur

KSeF umożliwia nadanie uprawnień podmiotom, które w imieniu danej organizacji będą przetwarzać faktury – np. biurom rachunkowym, centrom usług wspólnych czy firmom outsourcingowym. Uprawnienia InvoiceRead i InvoiceWrite mogą być nadane bezpośrednio i w razie potrzeby – z możliwością dalszego przekazywania (flaga `canDelegate`). W tej sekcji omówiono mechanizm nadawania tych uprawnień, wymagane role oraz przykładowe implementacje .

POST [/permissions/entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1entities~1grants/post)


* **InvoiceWrite (Wystawianie faktur)**: To uprawnienie umożliwia przesyłanie plików faktur w formacie XML do systemu KSeF. Po pomyślnej weryfikacji i nadaniu numeru KSeF, te pliki stają się ustrukturyzowanymi fakturami.
* **InvoiceRead (Przeglądanie faktur)**: Dzięki temu uprawnieniu, podmiot może pobierać listy faktur w ramach danego kontekstu, pobierać treści faktur, faktury, zgłaszać nadużycia, a także generować i przeglądać identyfikatory zbiorczych płatności.
* Uprawnienia **InvoiceWrite** i **InvoiceRead** mogą być nadawane bezpośrednio podmiotom przez podmiot uprawniający. Klient API, który nadaje te uprawnienia bezpośrednio, musi posiadać uprawnienie **CredentialsManage** lub rolę **Owner**. W przypadku nadawania uprawnień podmiotom, możliwe jest ustawienie flagi `canDelegate` na `true` dla **InvoiceRead** oraz **InvoiceWrite**, co pozwala na dalsze, pośrednie przekazywanie tego uprawnienia.



| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator podmiotu. `"Nip"`               |
| `permissions`                               | Uprawnienia do nadania. `"InvoiceWrite"`, `"InvoiceRead"`			   |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:

```csharp
 var request = GrantEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subjectIdentifier)
    .WithPermissions(
        Permission.New(StandardPermissionType.InvoiceRead, true),
        Permission.New(StandardPermissionType.InvoiceRead, false)
        )
    .WithDescription("Access for quarterly review")
    .Build();

 return await ksefClient.GrantsPermissionEntityAsync(request, accessToken, cancellationToken);
```
Przykład w języku Java:
```java
var request = new EntityPermissionsGrantRequestBuilder()
            .withSubjectIdentifier(subjectIdentifier)
            .withPermissions(List.of(
                    new EntityPermission(EntityPermissionType.INVOICEREAD, true),
                    new EntityPermission(EntityPermissionType.INVOICEREAD, false))
            )
            .withDescription("Access for quarterly review")
            .build();

return ksefClient.grantsPermissionEntity(request);
```

---
### Nadanie uprawnień podmiotowych

Dla wybranych procesów fakturowania KSeF przewiduje tzw. uprawnienia podmiotowe, które mają zastosowanie w kontekście fakturowania w imieniu innego podmiotu (`TaxRepresentative`, `SelfInvoicing`, `RRInvoicing`). Te uprawnienia mogą być nadawane wyłącznie przez właściciela lub administratora posiadającego `CredentialsManage`. Sekcja przedstawia sposób ich nadawania, zastosowanie oraz ograniczenia techniczne.

POST [/permissions/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants/post)

Służy do nadawania uprawnień podmiotowych, takich jak `SelfInvoicing` (samofakturowanie), `RRInvoicing` (samofakturowanie RR) czy `TaxRepresentative` (operacje przedstawiciela podatkowego).

Charakter uprawnień:

Są to uprawnienia podmiotowe, co oznacza, że są istotne przy wysyłaniu plików faktur i weryfikowane w procesie ich walidacji. Mogą być modyfikowane w trakcie sesji.
Wymagane uprawnienia do nadawania uprawnień: ```CredentialsManage``` lub ```Owner```.

| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator podmiotu. `"Nip"`               |
| `permissions`                               | Uprawnienia do nadania. `"SelfInvoicing"`, `"RRInvoicing"`, `"TaxRepresentative"`			   |
| `description`                              | Wartość tekstowa (opis)              |


Przykład w języku C#:

```csharp
  var request = GrantProxyEntityPermissionsRequestBuilder
            .Create()
            .WithSubject(subjectIdentifier)
            .WithPermission(StandardPermissionType.TaxRepresentative)
            .WithDescription("Access for quarterly review")
            .Build();
 
 return await ksefClient.GrantsPermissionProxyEntityAsync(request, accessToken, cancellationToken);

```

Przykład w języku Java:
```java
GrantPermissionsProxyEntityRequest request = new GrantPermissionsProxyEntityRequest();
    request.setSubjectIdentifier(subjectIdentifier);
    request.setPermissions(List.of(ProxyEntityPermissionType.TAXREPRESENTATIVE));
    request.setDescription("Access for quarterly review");

return ksefClient.grantsPermissionsProxyEntity(request);
```
---
### Nadanie uprawnień w sposób pośredni

Mechanizm pośredniego nadawania uprawnień umożliwia działanie tzw. podmiotu pośredniczącego, który – na podstawie uprzednio uzyskanych delegacji – może przekazywać wybrane uprawnienia dalej, w kontekście innego podmiotu. Najczęściej dotyczy to biur rachunkowych, które obsługują wielu klientów. W sekcji opisano warunki, jakie muszą zostać spełnione, aby skorzystać z tej funkcjonalności oraz przedstawiono strukturę danych wymaganych do wykonania takiego nadania.

Uprawnienia `InvoiceWrite` i `InvoiceRead` to jedyne uprawnienia, które mogą być nadawane w sposób pośredni. Oznacza to, że podmiot pośredniczący może nadać te uprawnienia innemu podmiotowi (uprawnionemu), które będą obowiązywać w kontekście podmiotu docelowego (partnera). Uprawnienia te mogą być selektywne (dla konkretnego partnera) lub generalne (dla wszystkich partnerów podmiotu pośredniczącego).

POST [/permissions/indirect/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1indirect~1grants/post)



| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator osoby fizycznej. `"Nip"`, `"Pesel"`, `"Fingerprint"`               |
| `targetIdentifier`                        | Identyfikator podmiotu docelowego. `"Nip"` lub `null`              |
| `permissions`                               | Uprawnienia do nadania. `"InvoiceRead"`, `"InvoiceWrite"`			   |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:

```csharp
var request = GrantIndirectEntityPermissionsRequestBuilder
     .Create()
     .WithSubject(grantPermissionsRequest.SubjectIdentifier)
     .WithContext(grantPermissionsRequest.TargetIdentifier)
     .WithPermissions(grantPermissionsRequest.Permissions.ToArray())
     .WithDescription(grantPermissionsRequest.Description)
     .Build();

return await ksefClient.GrantsPermissionIndirectEntityAsync(request, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
IndirectPermissionsGrantRequest request = new IndirectPermissionsGrantRequest();
    request.setSubjectIdentifier(subjectIdentifier);
    request.setPermissions(List.of(IndirectPermissionType.INVOICEREAD, IndirectPermissionType.INVOICEWRITE));
    request.setDescription("Access for quarterly review");

return ksefClient.grantsPermissionIndirectEntity(request);
```
---
### Nadanie uprawnień administratora podmiotu podrzędnego

Struktura organizacyjna podmiotu może obejmować jednostki lub podmioty podrzędne – np. oddziały, działy, spółki zależne, członków grupy VAT oraz jednostki samorządu terytorialnego. KSeF umożliwia przypisanie uprawnień do zarządzania takimi jednostkami. Wymagane jest posiadanie uprawnienia `SubunitManage`. W tej sekcji przedstawiono sposób nadania uprawnień administracyjnych w kontekście jednostki podrzędnej lub podmiotu podrzędnego, z uwzględnieniem identyfikatora `InternalId` lub `Nip`.

POST [/permissions/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1subunits~1grants/post)



Wymagane uprawnienia do nadawania:

- Użytkownik, który chce nadać te uprawnienia, musi posiadać uprawnienie ```SubunitManage``` (Zarządzanie jednostkami podrzędnymi). 

| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator osoby fizycznej lub podmiotu. `"Nip"`, `"Pesel"`, `"Fingerprint"`               |
| `contextIdentifier`                        | Identyfikator podmiotu podrzędnego. `"Nip"`, `InternalId`              |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:

```csharp
var request = GrantSubUnitPermissionsRequestBuilder
    .Create()
    .WithSubject(grantPermissionsRequest.SubjectIdentifier)
    .WithContext(grantPermissionsRequest.ContextIdentifier)
    .WithDescription(grantPermissionsRequest.Description)
    .Build();

 return await ksefClient.GrantsPermissionSubUnitAsync(request, accessToken, cancellationToken);
```
Przykład w języku Java:
```java
var request = new SubunitPermissionsGrantRequestBuilder()
    .withSubjectIdentifier(subjectIdentifier)
    .withContextIdentifier(subjectIdentifier)
    .withDescription("Sub-entity access")
    .build();

return ksefClient.grantsPermissionSubUnit(request);
```
---
### Nadanie uprawnień administratora podmiotu unijnego

W przypadku organizacji zagranicznych zarejestrowanych w UE, które uczestniczą w polskim obrocie fakturowym, możliwe jest ustanowienie administratora odpowiedzialnego za zarządzanie dostępem w kontekście podmiotu złożonego, czyli pary polskiego NIPu i identyfikatora VAT UE. (`NipVatUe`). Sekcja przedstawia wymagania oraz format danych wymaganych do nadania takich uprawnień.

POST [/permissions/eu-entities/administration/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1administration~1grants/post)



| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator osoby fizycznej lub podmiotu. `"Nip"`, `"Pesel"`, `"Fingerprint"`               |
| `contextIdentifier`                        | Dwuczłonowy identyfikator składający się z numeru NIP i numeru VAT-UE `{nip}-{vat_ue}`. `"NipVatUe"`              |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:

```csharp
     var request = GrantEUEntityPermissionsRequestBuilder
         .Create()
         .WithSubject(grantPermissionsRequest.SubjectIdentifier)
         .WithContext(grantPermissionsRequest.ContextIdentifier)
         .WithDescription("Access for quarterly review")
         .Build();

 return await ksefClient.GrantsPermissionEUEntityAsync(request, accessToken, cancellationToken);
```
Przykład w języku Java:
```java
var request = new EuEntityPermissionsGrantRequestBuilder()
    .withSubject(body.subjectIdentifier())
    .withContext(body.contextIdentifier())
    .withDescription("Access for quarterly review")
    .build();

return ksefClient.grantsPermissionEUEntity(request);
```
---
### Nadanie uprawnień reprezentanta podmiotu unijnego

Reprezentant podmiotu unijnego to osoba działająca na rzecz jednostki zarejestrowanej w UE, która potrzebuje dostępu do KSeF w celu przeglądania lub wystawiania faktur. Takie uprawnienie może być nadane wyłącznie przez administratora VAT UE. Sekcja przedstawia strukturę danych oraz sposób wywołania odpowiedniego endpointu.

POST [/permissions/eu-entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1grants/post)



| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator podmiotu. `"Fingerprint"`               |
| `permissions`                               | Uprawnienia do nadania. `"InvoiceRead"`, `"InvoiceWrite"`			   |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:
```csharp
var request = GrantEUEntityPermissionsRequestBuilder
            .Create()
            .WithSubject(grantPermissionsRequest.SubjectIdentifier)
            .WithContext(grantPermissionsRequest.ContextIdentifier)
            .WithDescription("Access for quarterly review")
            .Build();

 return await ksefClient.GrantsPermissionEUEntityRepresentativeAsync(request, accessToken, cancellationToken);
```
Przykład w języku Java:
```java
var request = new EuEntityPermissionsGrantRepresentativeRequestBuilder()
    .withSubjectIdentifier(subjectIdentifier)
    .withPermissions(List.of(EuEntityPermissionType.INVOICEREAD, EuEntityPermissionType.INVOICEWRITE))
    .withDescription("Representative access")
    .build();

return ksefClient.grantsPermissionEUEntityRepresentative(request);
```

## Odbieranie uprawnień

Proces odbierania uprawnień w KSeF jest równie istotny, jak ich nadawanie – zapewnia kontrolę dostępu i umożliwia szybkie reagowanie w sytuacjach, takich jak zmiana roli pracownika, zakończenie współpracy z partnerem zewnętrznym lub naruszenie zasad bezpieczeństwa. Odebranie uprawnień może być wykonane dla każdej kategorii odbiorcy: osoby fizycznej, podmiotu, jednostki podrzędnej, przedstawiciela UE lub administratora UE. W tej sekcji omówiono metody wycofywania różnych typów uprawnień oraz wymagane identyfikatory.

### Odebranie uprawnień

Standardowa metoda odbierania uprawnień, z której można skorzystać w odniesieniu do większości przypadków: osób fizycznych, podmiotów krajowych, jednostek podrzędnych, a także reprezentantów UE lub administratorów UE. Operacja wymaga znajomości `permissionId` oraz posiadania odpowiedniego uprawnienia. 

DELETE [/permissions/common/grants/\{permissionId\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Odbieranie-uprawnien/paths/~1api~1v2~1permissions~1common~1grants~1%7BpermissionId%7D/delete)

Ta metoda służy do odbierania uprawnień takich jak:

- nadanych osobom fizycznym do pracy w KSeF,
- nadanych podmiotom do obsługi faktur,
- nadanych w sposób pośredni,
- administratora podmiotu podrzędnego,
- administratora podmiotu unijnego,
- reprezentanta podmiotu unijnego.

Przyklad w języku C#:
```csharp
await ksefClient.RevokeCommonPermissionAsync(permissionId, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
```
---
### Odebranie uprawnień podmiotowych

W przypadku uprawnień typu podmiotowego (`SelfInvoicing`, `RRInvoicing`, `TaxRepresentative`), obowiązuje osobna metoda odbierania – z użyciem endpointu dedykowanego do operacji autoryzacyjnych. Tego typu uprawnienia nie są przekazywalne, więc ich odebranie ma natychmiastowy skutek i kończy możliwość realizacji operacji fakturowych w danym trybie. Wymagana jest znajomość `permissionId`.

DELETE [/permissions/authorizations/grants/\{permissionId\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Odbieranie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants~1%7BpermissionId%7D/delete)

Ta metoda służy do odbierania uprawnień takich jak:

- samofakturowanie,
- samofakturowanie RR,
- operacje przedstawiciela podatkowego.

Przykład w języku C#:
```csharp
await ksefClient.RevokeAuthorizationsPermissionAsync(permissionId, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
```


## Wyszukiwanie nadanych uprawnień

KSeF udostępnia zestaw endpointów pozwalających na odpytywanie listy aktywnych  uprawnień nadanych użytkownikom i podmiotom. Mechanizmy te są niezbędne do audytu, przeglądu stanu dostępu, a także przy budowie interfejsów administracyjnych (np. do zarządzania strukturą dostępu w organizacji). Sekcja zawiera przegląd metod wyszukiwania z podziałem na kategorie nadanych uprawnień.

---
### Pobranie listy uprawnień do pracy w KSeF nadanych osobom fizycznym lub podmiotom

Zapytanie pozwala na pobranie listy uprawnień nadanych osobom fizycznym lub podmiotom – np. pracownikom firmy. Możliwe jest filtrowanie po rodzaju uprawnień, stanie (`Active` / `Inactive`), a także identyfikatorze nadawcy i odbiorcy. Endpoint ten bywa wykorzystywany przy onboardingu, audycie oraz monitoringu uprawnień personalnych.

POST [/permissions/query/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1persons~1grants/post)

| Pole                  | Opis                                                                 |
| :-------------------- | :------------------------------------------------------------------- |
| `authorIdentifier`    | Identyfikator podmiotu nadającego uprawnienia.   ```Nip```, ```Pesel```,```Fingerprint```                     |
| `authorizedIdentifier`| Identyfikator podmiotu, któremu nadano uprawnienia.      ```Nip```, ```Pesel```,```Fingerprint```             |
| `targetIdentifier`    | Identyfikator podmiotu docelowego (dla uprawnień pośrednich).  ```Nip```      |
| `permissionTypes`     | Typy uprawnień do filtrowania.   `"CredentialsManage"`, `"CredentialsRead"`, `"InvoiceWrite"`, `"InvoiceRead"`, `"Introspection"`, `"SubunitManage"`, `"EnforcementOperations"`  |
| `permissionState`     | Stan uprawnienia.  ```Active``` / ```Inactive```                                                  |

Przykład w języku C#:

```csharp
await ksefClient.SearchGrantedPersonPermissionsAsync(request, accessToken, pageOffset, pageSize);
```

Przykład w języku Java:
```java
ksefClient.searchGrantedPersonPermissions(request, pageOffset, pageSize);
```
---
### Pobranie listy uprawnień administratorów jednostek i podmiotów podrzędnych

Ten endpoint służy do pobrania informacji o administratorach jednostek podrzędnych lub podmiotów podrzędnych (np. oddziałów, grup VAT). Pozwala na monitorowanie, kto posiada uprawnienia zarządcze względem danej struktury podrzędnej, identyfikowanej za pomocą `InternalId` lub `Nip`.

POST [/permissions/query/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subunits~1grants/post)

| Pole                  | Opis                                                                 |
| :-------------------- | :------------------------------------------------------------------- |
| `subjectIdentifier`    | Identyfikator podmiotu podrzędnego.   ```InternalId``` lub `Nip`            |

Przykład w języku C#:
```csharp
await ksefClient.SearchSubunitAdminPermissionsAsync(accessToken, request, pageOffset, pageSize);
```

Przykład w języku Java:
```java
ksefClient.searchSubunitAdminPermissions(request, pageOffset, pageSize);
```
---
### Pobranie listy ról podmiotu

Endpoint zwraca zestaw ról przypisanych do kontekstu w ktorym jesteśmy uwierzytelnieni (czyli tego, w imieniu którego wykonywane jest zapytanie). Funkcja wykorzystywana głównie przy automatycznym sprawdzaniu dostępów przez systemy klienckie.

GET [/permissions/query/entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1entities~1roles/get)

Przykład w języku C#:
```csharp
await ksefClient.SearchEntityInvoiceRolesAsync(accessToken, pageOffset, pageSize);
```

Przykład w języku Java:
```java
ksefClient.searchEntityInvoiceRoles(pageOffset, pageSize);
```
---
### Pobranie listy podmiotów podrzędnych

Pozwala na uzyskanie informacji o powiązanych podmiotach podrzędnych dla kontekstu w którym jesteśmy uwierzytelnieni (czyli tego, w imieniu którego wykonywane jest zapytanie). Funkcja głównie wykorzystywana w celu weryfikacji struktury jednostek samorządu terytorialnego lub grup VAT.

POST [/permissions/query/subordinate-entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subordinate-entities~1roles/post)

| Pole                     | Opis                                                                                                              |
| :----------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `subordinateEntityIdentifier`   | Identyfikator podmiotu, któremu nadano uprawnienia. ```Nip```                                                     |                                               |
    

Przykład w języku C#:

```csharp
await ksefClient.SearchSubordinateEntityInvoiceRolesAsync(request, accessToken,pageOffset, pageSize)
```

Przykład w języku Java:
```java
ksefClient.searchSubordinateEntityInvoiceRoles(request, pageOffset, pageSize);
```
---
### Pobranie listy uprawnień podmiotowych do obsługi faktur

Endpoint ten służy do przeglądu wszystkich nadanych uprawnień podmiotowych nadanych przez kontekst w ktorym jestesmy uwierzytelnieni lub nadanych na kontekst w ktorym jestesmy uwierzytelnieni. Wspiera filtrowanie po typie uprawnień i odbiorcy.



POST [/permissions/query/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1authorizations~1grants/post)

| Pole                     | Opis                                                                                                              |
| :----------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `authorizingIdentifier`  | Identyfikator podmiotu nadającego uprawnienia.  ```Nip```                                                     |
| `authorizedIdentifier`   | Identyfikator podmiotu, któremu nadano uprawnienia. ```Nip```                                                     |
| `queryType`              | Typ zapytania. Określa czy odpytujemy o nadane czy otrzymane uprawnienia. ```Granted``` ```Received```            |
| `permissionTypes`        | Typy uprawnień do filtrowania.   `"SelfInvoicing"`, `"TaxRepresentative"`, `"RRInvoicing"`,                       |
 

Przyklad w języku C#:
```csharp
await ksefClient.SearchEntityAuthorizationGrantsAsync(request, accessToken, pageOffset, pageSize);
```

Przykład w języku Java:
```java
```
---
### Pobranie listy uprawnień administratorów lub reprezentantów podmiotów unijnych uprawnionych do samofakturowania

Podmioty unijne również mogą mieć przypisane uprawnienia do korzystania z KSeF. W tej sekcji możliwe jest pobranie informacji o nadanych im dostępach, z uwzględnieniem identyfikatorów VAT UE i odcisku palca certyfikatu.

POST [/permissions/query/eu-entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1eu-entities~1grants/post)

| Pole                        | Opis                                                                 |
| :-------------------------- | :------------------------------------------------------------------- |
| `vatUeIdentifier`           | Identyfikator VAT UE.                                                |
| `authorizedFingerprintIdentifier` | Odcisk palca certyfikatu uprawnionego podmiotu.                      |
| `permissionTypes`           | Typy uprawnień do filtrowania. Możliwe wartości to: `VatUeManage`, `InvoiceWrite`, `InvoiceRead`, `Introspection`. |

Przykład w języku C#:
```csharp
await ksefClient.SearchGrantedEuEntityPermissionsAsync(request, accessToken, pageOffset, pageSize)
```

Przykład w języku Java:
```java
ksefClient.searchGrantedEuEntityPermissions(request, pageOffset, pageSize);
```

## Operacje 

Krajowy System e-Faktur umożliwia śledzenie i weryfikację statusu operacji związanych z zarządzaniem uprawnieniami. Każde nadanie lub odebranie uprawnienia jest realizowane jako asynchroniczna operacja, której status można monitorować przy użyciu unikalnego identyfikatora referencyjnego (`operationReferenceNumber`). Sekcja ta prezentuje mechanizm pobierania statusu operacji i jego interpretacji w kontekście automatyzacji i kontroli poprawności działań administracyjnych w KSeF.

### Pobranie statusu operacji

Po nadaniu lub odebraniu uprawnienia, system zwraca numer referencyjny operacji (`operationReferenceNumber`). Dzięki temu identyfikatorowi możliwe jest sprawdzenie aktualnego stanu przetwarzania żądania: czy zakończyło się sukcesem, czy wystąpił błąd, lub czy nadal trwa przetwarzanie. Informacja ta może być kluczowa w systemach nadzorczych, logice automatycznego ponawiania operacji lub raportowaniu działań administracyjnych. W tej sekcji przedstawiono przykład wywołania API służącego do pobrania statusu operacji.

GET [/permissions/operations/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Operacje/paths/~1api~1v2~1permissions~1operations~1%7BoperationReferenceNumber%7D/get)

Każda operacja nadania uprawnienia zwraca identyfikator operacji, który można wykorzystać do sprawdzenia statusu tej operacji.

Przykład w języku C#:
```csharp
var operationStatus = await ksefClient.OperationsStatusAsync(referenceNumber, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
var permissionsOperationStatusResponse = ksefClient.operations(referenceNumber);
```
