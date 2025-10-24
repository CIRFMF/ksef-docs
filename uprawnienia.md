## Uprawnienia
10.07.2025

### Wprowadzenie – kontekst biznesowy
System KSeF wprowadza zaawansowany mechanizm zarządzania uprawnieniami, który stanowi fundament bezpiecznego i zgodnego z przepisami korzystania z systemu przez różne podmioty. Uprawnienia decydują o tym, kto może wykonywać określone operacje w KSeF – takie jak wystawianie faktur, przeglądanie dokumentów, nadawanie dalszych uprawnień czy zarządzanie jednostkami podrzędnymi.

### Cele zarządzania uprawnieniami:
- Bezpieczeństwo danych – ograniczenie dostępu do informacji tylko do osób i podmiotów, które są do tego formalnie uprawnione.
- Zgodność z przepisami – zapewnienie, że operacje są wykonywane przez właściwe jednostki zgodnie z wymogami ustawowymi (np. Ustawa VAT).
- Audytowalność – każda operacja związana z nadaniem lub odebraniem uprawnień jest rejestrowana i może być poddana analizie.

### Kto nadaje uprawnienia?
Uprawnienia mogą być nadawane przez:

- właściciela podmiotu - rola (Owner),
- administratora podmiotu podrzędnego,
- administratora jednostki podrzędnej,
- administratora podmiotu unijnego,
- administratora podmiotu, czyli inny podmiot lub osoba posiadająca uprawnienie CredentialsManage.

W praktyce oznacza to, że każda organizacja musi zarządzać uprawnieniami swoich pracowników np. nadając uprawnienia pracownikowi działu księgowości podczas przyjmowaniu nowego pracownika lub też odbierając uprawnienia gdy taki pracownik kończy stosunek pracy.

### Kiedy nadaje się uprawnienia?
#### Przykłady:
- podczas rozpoczęcia współpracy z nowym pracownikiem,
- w przypadku gdy firma wchodzi we współpracę np. z biurem rachunkowym powinna nadać uprawnienia do odczytu faktur na to biuro rachunkowe, aby to biuro mogło rozliczać faktury tej firmy,
- w zwiazku ze zmianami relacji pomiedzy podmiotami.

### Struktura nadanych uprawnień:
Uprawnienia są nadawane:
1) Osobom fizycznym identyfikowanym przez PESEL, NIP lub odcisk palca certyfikatu – do pracy w KSeF:
    - w kontekście podmiotu nadającego uprawnienie (uprawnienia nadawane bezpośrednio) lub
    - w kontekście innego podmiotu lub innych podmiotów:
        - w kontekście podmiotu podrzędnego identyfikowanego przez NIP (podrzędnej jednostki samorządu terytorialnego lub członka grupy VAT),
        - w kontekście jednostki podrzędnej identyfikowanej identyfikatorem wewnętrznym,
        - w kontekście złożonym NIP- VAT UE łączącym podmiot polski z podmiotem unijnym uprawnionym do samofakturowania w imieniu tego podmiotu polskiego,
        - w kontekście wskazanego podmiotu identyfikowanego przez NIP – klienta podmiotu nadającego uprawnienia (uprawnienia selektywne nadawane w sposób pośredni),
        - w kontekście wszystkich podmiotów – klientów podmiotu nadającego uprawnienia (uprawnienia generalne nadawane w sposób pośredni).
2) Innym podmiotom – identyfikowanym przez NIP:
    - jako końcowym odbiorcom uprawnień do wystawiania lub przeglądania faktur,
    - jako pośrednikom - z włączoną opcją zezwolenia na dalsze przekazywanie uprawnień, aby uprawniony podmiot miał możliwość nadawania uprawnień w sposób pośredni (patrz p. IV i V powyzej).

3) Innym podmiotom do działania w swoim kontekście w imieniu podmiotu uprawniającego (uprawnienia podmiotowe):
    - przedstawicielom podatkowym,
    - podmiotom uprawnionym do samofakturowania,
    - podmiotom uprawnionym do wystawiania faktur VAT RR.

Dostęp do funkcji systemu zależy kontekstu, w którym nastąpiło uwierzytelnienie oraz od zakresu uprawnień, jakie nadano uwierzytelnionemu podmiotowi/osobie w tym kontekście.

##  Słowniczek pojęć (w zakresie uprawnień KSeF)

| Termin                          | Definicja |
|---------------------------------|-----------|
| **Uprawnienie**                 | Zezwolenie na wykonanie określonych operacji w KSeF, np. `InvoiceWrite`, `CredentialsManage`. |
| **Właściciel**                       | Właściciel podmiotu – osoba mająca domyślnie pełen dostęp do operacji w kontekście podmiotu mającego taki sam identyfikator NIP, jaki jest zapisany w użytym środku uwierzytelnienia; dla właściciela obowiązuje również powiązanie NIP-PESEL, zatem może uwierzytelnić się również środkiem zawierającym powiązany nr PESEL zachowując wszystkie uprawnienia właściciela. |
| **Administrator podmiotu podrzędnego**              | Osoba z uprawnieniami do zarządzania uprawnieniami (`CredentialsManage`) w kontekście podmiotu podrzędnego. Może nadawać uprawnienia (np. `InvoiceWrite`). Podmiotem podrzędnym może być np. członek grupy VAT. |
| **Administrator jednostki podrzędnej**              | Osoba z uprawnieniami do zarządzania uprawnieniami  (`CredentialsManage`) w jednostce podrzędnej. Może nadawać uprawnienia (np. `InvoiceWrite`). |
| **Administrator podmiotu unijnego**              | Osoba z uprawnieniami do zarządzania uprawnieniami (`CredentialsManage`) w kontekście złożonym identyfikowanym za pomocą NipVatUe. Może nadawać uprawnienia (np. `InvoiceRead`). |
| **Podmiot pośredniczący**   | Podmiot, który otrzymał uprawnienie z flagą `canDelegate = true` i może przekazać to uprawnienie dalej, czyli nadawać uprawnienie w sposób pośredni. Mogą to być tylko uprawnienia `InvoiceWrite` i `InvoiceRead`. |
| **Podmiot docelowy**  | Podmiot, w którego kontekście obowiązuje dane uprawnienie – np. firma, obsługiwana przez biuro rachunkowe. |
| **Nadane w sposób bezpośredni**       | Uprawnienie nadane wprost danemu użytkownikowi lub podmiotowi przez właściciela lub administratora. |
| **Nadanie w sposób pośredni**          | Uprawnienie nadane przez pośrednika do obsługi innego podmiotu – tylko dla `InvoiceRead` i `InvoiceWrite`. |
| **`canDelegate`**              | Flaga techniczna (`true` / `false`) pozwalająca na delegowanie uprawnień. Tylko `InvoiceRead` oraz `InvoiceWrite` mogą mieć `canDelegate = true`. Może być wykorzystana tylko podczas nadawania uprawnienia podmiotowi do obsługi faktur |
| **`subjectIdentifier`**        | Dane identyfikujące odbiorcę uprawnień (osobę lub podmiot): `Nip`, `Pesel`, `Fingerprint`. |
| **`targetIdentifier` / `contextIdentifier`** | Dane identyfikujące kontekst, w którym działa nadane uprawnienie – np. NIP klienta, Identyfikator wewnętrzny jednostki organizacyjnej. |
| **Fingerprint**                | Wynik obliczenia funkcji skrótu SHA-256 na certyfikacie kwalifikowanym. Pozwala na rozpoznanie certyfikatu podmiotu posiadającego uprawnienie nadane na odcisk palca certyfikatu Używany m.in. w identyfikacji osób lub podmiotów zagranicznych. |
| **InternalId**                 | Wewnętrzny identyfikator jednostki podrzędnej w systemie KSeF - dwuczłonowy identyfikator składający się z numeru NIP oraz pięciu cyfr `nip-5_cyfr`.  |
| **NipVatUe**                   | Identyfikator złożony, czyli dwuczłonowy identyfikator składający się z nr NIP podmiotu polskiego oraz numeru VAT UE podmiotu unijnego, które są oddzielone za pomocą separatora `nip-vat_ue`. |
| **Odbieranie**                     | Operacja odebrania wcześniej nadanego uprawnienia. |
| **`permissionId`**             | Techniczny identyfikator nadanego uprawnienia – wymagany m.in. przy operacjach odbierania. |
| **`operationReferenceNumber`** | Identyfikator operacji (np. nadania lub odebrania uprawnień), zwracany przez API, wykorzystywany do sprawdzenia statusu. |
| **Status operacji**            | Bieżący stan procesu nadania/odebrania uprawnień: `100`, `200`, `400` itp. |

## Model ról i uprawnień (macierz uprawnień)

System KSeF umożliwia przypisywanie uprawnień w sposób precyzyjny, z uwzględnieniem rodzajów czynności wykonywanych przez użytkowników. Uprawnienia mogą być nadawane zarówno bezpośrednio, jak i pośrednio – w zależności od mechanizmu delegowania dostępu.

### Przykłady ról do odwzorowania za pomocą uprawnień:

| Rola / podmiot                          | Opis roli                                                                                          | Możliwe uprawnienia                                                                 |
|----------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| **Właściciel podmiotu**      | Rola posiadana domyślnie z automatu przez właściciela. Aby być rozpoznanym przez system jako właściciel należy się uwierzytelnić wektorem z takim samym identyfikatorem NIP jak NIP kontekstu logowania lub powiązanym numerem PESEL           | Rola `Owner` obejmująca wszystkie uprawnienia fakturowe i administracyjne poza `VatUeManage`. |
| **Administrator podmiotu**            | Osoba fizyczna posiadająca prawa do nadawania i odbierania uprawnień innym użytkownikom i/lub powoływania administratorów jednostek/podmiotów podrzędnych.           | `CredentialsManage`, `SubunitManage`, `Introspection`.                              |
| **Operator (księgowość / fakturowanie)** | Osoba odpowiedzialna za wystawianie lub przeglądanie faktur.                                        | `InvoiceWrite`, `InvoiceRead`.                                                      |
| **Podmiot upoważniony**                | Inny podmiot gospodarczy, któremu nadano określone uprawnienia do wystawiania faktur w imieniu podmiotu, np. Przedstawiciel podatkowy.             | `SelfInvoicing`, `RRInvoicing`, `TaxRepresentative`                             |
| **Podmiot pośredniczący**              | Podmiot, który otrzymał uprawnienia z opcją delegacji (`canDelegate`) i może nadać je dalej.       | `InvoiceRead`, `InvoiceWrite` z flagą `canDelegate = true`.   
| **Administrator podmiotu unijnego**     | Osoba identyfikująca się certyfikatem  posiadająca prawa do nadawania i odbierania uprawnień innym użytkownikom w ramach podmiotu unijnego powiązanego z danym podmiotem polskim.                                     | `InvoiceWrite`, `InvoiceRead`,                                    `VatUeManage`,  `Introspection`.                      |                      |
| **Reprezentant podmiotu unijnego**     | Osoba identyfikująca się certyfikatem działająca na rzecz podmiotu unijnego powiązanego z danym podmiotem polskim.                                     | `InvoiceWrite`, `InvoiceRead`.                                                      |
| **Administrator jednostki podrzędnej** | Użytkownik mający możliwość powoływania administratorów w jednostkach lub podmiotach podrzędnych.               | `CredentialsManage`.                                                                    |

---

### Klasyfikacja uprawnień według rodzaju:

| Typ uprawnienia           | Przykładowe wartości                                       | Możliwość nadania w sposób pośredni | Opis operacyjny                                                              |
|--------------------------|------------------------------------------------------------|-------------------------------|------------------------------------------------------------------------------|
| **Fakturowe**             | `InvoiceWrite`, `InvoiceRead`                              | ✔️ (jeśli `canDelegate=true`) | Operacje na fakturach: wysyłanie, pobieranie                     |
| **Administracyjne**       | `CredentialsManage`, `SubunitManage`,  `VatUeManage`.                       | ❌                            | Zarządzanie uprawnieniami, jednostkami podrzędnymi                      |
| **Podmiotowe**        | `SelfInvoicing`, `RRInvoicing`, `TaxRepresentative`        | ❌                            | Upoważnienie innych podmiotów do działania (wystawiania faktur) we własnym kontekście w imieniu podmiotu uprawniającego         |
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
- Typ identyfikatora podmiotu docelowego `targetIdentifier` to `AllPartners`.
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

1. **Nadanie bezpośrednie** – np. administrator podmiotu A przypisuje użytkownikowi uprawnienie `InvoiceWrite` osobie fizycznej w kontekście podmiotu A.
2. **Nadanie z możliwością dalszego przekazania** – np. administrator podmiotu A nadaje podmiotowi B (pośrednikowi) uprawnienie `InvoiceRead` z `canDelegate=true`, co umożliwia administratorowi podmiotu B nadanie `InvoiceRead` podmiotowi/osobie C.
3. **Nadanie w sposób pośredni** – z użyciem dedykowanego endpointu /permissions/indirect/grants, gdzie administrator podmiotu pośrednika B, który otrzymał od podmiotu A uprawnienie z delegacją, nadaje uprawnienia w imieniu do obsługi podmiotu docelowego A podmiotowi/osobie C.

---

### Przykład macierzy uprawnień:

| Użytkownik / Podmiot       | InvoiceWrite | InvoiceRead | CredentialsManage | SubunitManage | TaxRepresentative |
|----------------------------|--------------|-------------|--------------------|----------------|--------------------|
| Anna Kowalska (PESEL)      | ✅           | ✅          | ❌                 | ❌             | ❌                 |
| Biuro Rachunkowe XYZ (NIP) | ✅ (z delegacją)          | ✅ (z delegacją) | ❌                 | ❌             | ❌                 |
| Jan Nowak (Identyfikujacy sie certyfikatem)   | ✅           | ✅          | ❌                 | ❌             | ❌                 |
| Admin działu księgowości (PESEL)           | ❌           | ❌          | ✅                 | ✅             | ❌                 |
| Spółka Matka tj. owner (NIP)         | ✅           | ✅          | ✅                 | ✅             | ✅                 |
| Admin grupy VAT (PESEL)          | ❌           | ❌          | ❌                 | ✅             | ❌                 |
| Przedstawiciel podatkowy (NIP)          | ❌           | ❌          | ❌                 | ❌             | ✅                 |

---

### Role lub uprawnienia wymagane do nadawania uprawnień 

| Nadanie uprawnień:                        | Wymagana rola lub uprawnienie                      |
|-------------------------------------------|---------------------------------------------------|
| osobie fizycznej do pracy w KSeF      | `Owner` lub `CredentialsManage`                   |
| podmiotowi do obsługi faktur           | `Owner` lub `CredentialsManage`                   |
| podmiotowych | `Owner` lub `CredentialsManage`                   |
| do obsługi faktur  – w sposób pośredni              | `Owner` lub `CredentialsManage`    |
| administratorowi jednostki podrzędnej   | `SubunitManage`                                   |
| administratorowi podmiotu unijnego      | `Owner` lub `CredentialsManage`    |
| reprezentantowi podmiotu unijnego     | `VatUeManage`    |
---

### Ograniczenia identyfikatorów (`subjectIdentifier`, `contextIdentifier`)

| Typ identyfikatora | Identyfikowany | Uwagi |
|--------------------|---------------------|-------|
| `Nip`              | Podmiot krajowy     | Dla podmiotów zarejestrowanych w Polsce oraz osób fizycznych |
| `Pesel`            | Osoba fizyczna       | Wymagane m.in. przy nadaniu uprawnień pracownikom posługującym się profilem zaufanym lub certyfikatem kwalifikowanym z numerem PESEL  |
| `Fingerprint`      | Właściciel certyfikatu      | Wykorzystywane w sytuacji, gdy certyfikat kwalifikowany nie zawiera identyfikatora NIP ani PESEL oraz przy identyfikowaniu administratorów lub reprezentantów podmiotów unijnych   |
| `NipVatUe`         | Podmioty unijne powiązane z podmiotami polskimi       | Wymagane przy nadawaniu uprawnień administratorom i przedstawicielom podmiotów unijnych |
| `InternalId`       | Jednostki podrzędne  | Wykorzystywane w podmiotach o strukturze złożonej z jednostek podrzędnych |

---

### Ograniczenia funkcjonalne API

- Nie można nadać tego samego uprawnienia dwukrotnie – API może zwrócić błąd lub zignorować duplikat.
- Wykonanie operacji nadania uprawnienia nie skutkuje natychmiastowym dostępem – operacja jest asynchroniczna musi zostać poprawnie przetworzona przez system (należy sprawdzić status operacji).

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
[KSeF.Client.Tests.Core\E2E\Permissions\PersonPermission\PersonPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/PersonPermission/PersonPermissionE2ETests.cs)

```csharp
GrantPermissionsPersonRequest request = GrantPersonPermissionsRequestBuilder
    .Create()
    .WithSubject(subject)
    .WithPermissions(
        StandardPermissionType.InvoiceRead,
        StandardPermissionType.InvoiceWrite)
    .WithDescription(description)
    .Build();

OperationResponse response =
    await KsefClient.GrantsPermissionPersonAsync(request, accessToken, CancellationToken);
```

Przykład w języku Java:
[PersonPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/PersonPermissionIntegrationTest.java)

```java

GrantPersonPermissionsRequest request = new GrantPersonPermissionsRequestBuilder()
        .withSubjectIdentifier(new PersonPermissionsSubjectIdentifier(PersonPermissionsSubjectIdentifier.IdentifierType.PESEL, personValue))
        .withPermissions(List.of(PersonPermissionType.INVOICEWRITE, PersonPermissionType.INVOICEREAD))
        .withDescription("e2e test grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionPerson(request, accessToken);
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
[KSeF.Client.Tests.Core\E2E\Permissions\EntityPermission\EntityPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EntityPermission/EntityPermissionE2ETests.cs)

```csharp
GrantPermissionsEntityRequest request = GrantEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subject)
    .WithPermissions(
        Permission.New(StandardPermissionType.InvoiceRead, true),
        Permission.New(StandardPermissionType.InvoiceWrite, false)
    )
    .WithDescription(description)
    .Build();

OperationResponse response = await KsefClient.GrantsPermissionEntityAsync(request, accessToken, CancellationToken);
```
Przykład w języku Java:
[EntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EntityPermissionIntegrationTest.java)

```java
GrantEntityPermissionsRequest request = new GrantEntityPermissionsRequestBuilder()
        .withPermissions(List.of(
                new EntityPermission(EntityPermissionType.INVOICE_READ, true),
                new EntityPermission(EntityPermissionType.INVOICE_WRITE, false)))
        .withDescription(DESCRIPTION)
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, targetNip))
        .build();

OperationResponse response = ksefClient.grantsPermissionEntity(request, accessToken);
```

---
### Nadanie uprawnień podmiotowych

Dla wybranych procesów fakturowania KSeF przewiduje tzw. uprawnienia podmiotowe, które mają zastosowanie w kontekście fakturowania w imieniu innego podmiotu (`TaxRepresentative`, `SelfInvoicing`, `RRInvoicing`). Te uprawnienia mogą być nadawane wyłącznie przez właściciela lub administratora posiadającego `CredentialsManage`. Sekcja przedstawia sposób ich nadawania, zastosowanie oraz ograniczenia techniczne.

POST [/permissions/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants/post)

Służy do nadawania tzw. uprawnień podmiotowych, takich jak `SelfInvoicing` (samofakturowanie), `RRInvoicing` (samofakturowanie RR) czy `TaxRepresentative` (operacje przedstawiciela podatkowego).

Charakter uprawnień:

Są to uprawnienia podmiotowe, co oznacza, że są istotne przy wysyłaniu przez podmiot plików faktur i weryfikowane w procesie ich walidacji. Weryfikowana jest zależność pomiędzy podmiotem, a danymi na fakturach. Mogą być zmieniane w trakcie sesji. 

Wymagane uprawnienia do nadawania uprawnień: ```CredentialsManage``` lub ```Owner```.

| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator podmiotu. `"Nip"`               |
| `permissions`                               | Uprawnienia do nadania. `"SelfInvoicing"`, `"RRInvoicing"`, `"TaxRepresentative"`			   |
| `description`                              | Wartość tekstowa (opis)              |


Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\ProxyPermission\AuthorizationPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/ProxyPermission/AuthorizationPermissionsE2ETests.cs)

```csharp
GrantAuthorizationPermissionsRequest grantPermissionAuthorizationRequest =
    GrantAuthorizationPermissionsRequestBuilder
    .Create()
    .WithSubject(Fixture.SubjectIdentifier)
    .WithPermission(StandardPermissionType.SelfInvoicing)
    .WithDescription("E2E test grant")
    .Build();

OperationResponse operationResponse = await KsefClient
    .GrantsAuthorizationPermissionAsync(grantPermissionAuthorizationRequest,
    accessToken, CancellationToken);
```

Przykład w języku Java:
[ProxyPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ProxyPermissionIntegrationTest.java)

```java
GrantAuthorizationPermissionsRequest request = new GrantAuthorizationPermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withPermission(InvoicePermissionType.SELF_INVOICING)
        .withDescription("e2e test grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionsProxyEntity(request, accessToken);
```
---
### Nadanie uprawnień w sposób pośredni

Mechanizm pośredniego nadawania uprawnień umożliwia działanie tzw. podmiotu pośredniczącego, który – na podstawie uprzednio uzyskanych delegacji – może przekazywać wybrane uprawnienia dalej, w kontekście innego podmiotu. Najczęściej dotyczy to biur rachunkowych, które obsługują wielu klientów. W sekcji opisano warunki, jakie muszą zostać spełnione, aby skorzystać z tej funkcjonalności oraz przedstawiono strukturę danych wymaganych do wykonania takiego nadania.

Uprawnienia `InvoiceWrite` i `InvoiceRead` to jedyne uprawnienia, które mogą być nadawane w sposób pośredni. Oznacza to, że podmiot pośredniczący może nadać te uprawnienia innemu podmiotowi (uprawnionemu), które będą obowiązywać w kontekście podmiotu docelowego (partnera). Uprawnienia te mogą być selektywne (dla konkretnego partnera) lub generalne (dla wszystkich partnerów podmiotu pośredniczącego). W przypadku selektywnego nadania w identyfikatorze podmiotu docelowego należy podać typ `"Nip"` i wartość konkretnego numeru nip. Natomiast w przypadku uprawnień generalnych w identyfikatorze podmiotu docelowego należy podać typ `"AllPartners"`, bez uzupełnionego pola `value`.

POST [/permissions/indirect/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1indirect~1grants/post)



| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator osoby fizycznej. `"Nip"`, `"Pesel"`, `"Fingerprint"`               |
| `targetIdentifier`                        | Identyfikator podmiotu docelowego. `"Nip"` lub `null`              |
| `permissions`                               | Uprawnienia do nadania. `"InvoiceRead"`, `"InvoiceWrite"`			   |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\IndirectPermission\IndirectPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/IndirectPermission/IndirectPermissionE2ETests.cs)

```csharp
GrantPermissionsIndirectEntityRequest request = GrantIndirectEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subject)
    .WithContext(new TargetIdentifier { Type = TargetIdentifierType.Nip, Value = targetNip })
    .WithPermissions(StandardPermissionType.InvoiceRead, StandardPermissionType.InvoiceWrite)
    .WithDescription(description)
    .Build();

OperationResponse grantOperationResponse = await KsefClient.GrantsPermissionIndirectEntityAsync(request, accessToken, CancellationToken);
```

Przykład w języku Java:
[IndirectPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IndirectPermissionIntegrationTest.java)

```java
GrantIndirectEntityPermissionsRequest request = new GrantIndirectEntityPermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withTargetIdentifier(new TargetIdentifier(TargetIdentifier.IdentifierType.NIP, targetNip))
        .withPermissions(List.of(INVOICE_WRITE))
        .withDescription("E2E indirect grantE2E indirect grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionIndirectEntity(request, accessToken);

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
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
GrantPermissionsSubUnitRequest grantPermissionsSubUnitRequest =
    GrantSubUnitPermissionsRequestBuilder
    .Create()
    .WithSubject(Fixture.SubUnit)
    .WithContext(Fixture.Unit)
    .WithDescription("E2E test grant sub-unit")
    .Build();

OperationResponse operationResponse = await KsefClient
    .GrantsPermissionSubUnitAsync(grantPermissionsSubUnitRequest, accessToken, CancellationToken);
```
Przykład w języku Java:

[SubUnitPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubUnitPermissionIntegrationTest.java)

```java
SubunitPermissionsGrantRequest request = new SubunitPermissionsGrantRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withContextIdentifier(new ContextIdentifier(ContextIdentifier.IdentifierType.INTERNALID, contextNip))
        .withDescription("e2e subunit test")
        .withSubunitName("test")
        .build();

OperationResponse response = ksefClient.grantsPermissionSubUnit(request, accessToken);

```
---
### Nadanie uprawnień administratora podmiotu unijnego

Nadanie uprawnień administratora podmiotu unijnego w KSeF pozwala na uprawnienie podmiotu lub osoby wyznaczonej przez podmiot unijny mający prawo do samofakturowania w imieniu podmiotu polskiego nadającego uprawnienie. Wykonanie tej operacji powoduje, że uprawniona w ten sposób osoba uzyskuje możliwość logowania się w kontekście złożonym: `NipVatUe`, wiążącym podmiot polski nadający uprawnienie z podmiotem unijnym mającym prawo do samofakturowania. Po nadaniu uprawnień administratora podmiotu unijnego osoba taka będzie mogła wykonywać operacje na fakturach, a także zarządzać uprawnieniami innych osób (tzw. reprezentantów podmiotu unijnego) w ramach tego kontekstu złożonego.

POST [/permissions/eu-entities/administration/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1administration~1grants/post)



| Pole                                       | Wartość                                         |
| :----------------------------------------- | :---------------------------------------------- |
| `subjectIdentifier`                        | Identyfikator osoby fizycznej lub podmiotu. `"Nip"`, `"Pesel"`, `"Fingerprint"`               |
| `contextIdentifier`                        | Dwuczłonowy identyfikator składający się z numeru NIP i numeru VAT-UE `{nip}-{vat_ue}`. `"NipVatUe"`              |
| `description`                              | Wartość tekstowa (opis)              |

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\EuEntityPermission\EuEntityPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EuEntityPermission/EuEntityPermissionE2ETests.cs)

```csharp
GrantPermissionsRequest grantPermissionsRequest = GrantEUEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(TestFixture.EuEntity)
    .WithSubjectName(EuEntitySubjectName)
    .WithContext(contextIdentifier)
    .WithDescription(EuEntityDescription)
    .Build();

OperationResponse operationResponse = await KsefClient
    .GrantsPermissionEUEntityAsync(grantPermissionsRequest, accessToken, CancellationToken);
```
Przykład w języku Java:
[EuEntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityPermissionIntegrationTest.java)

```java
EuEntityPermissionsGrantRequest request = new GrantEUEntityPermissionsRequestBuilder()
        .withSubject(new SubjectIdentifier(SubjectIdentifier.IdentifierType.FINGERPRINT, euEntity))
        .withEuEntityName("Sample Subject Name")
        .withContext(new ContextIdentifier(ContextIdentifier.IdentifierType.NIP_VAT_UE, nipVatUe))
        .withDescription("E2E EU Entity Permission Test")
        .build();

OperationResponse response = ksefClient.grantsPermissionEUEntity(request, accessToken);

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
[KSeF.Client.Tests.Core\E2E\Permissions\EuAdministrationPermission\EuRepresentativePermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EuAdministrationPermission/EuRepresentativePermissionE2ETests.cs)

```csharp
GrantPermissionsEUEntityRepresentativeRequest grantRepresentativePermissionsRequest = GrantEUEntityRepresentativePermissionsRequestBuilder
    .Create()
    .WithSubject(new Client.Core.Models.Permissions.EUEntityRepresentative.SubjectIdentifier
    {
        Type = Client.Core.Models.Permissions.EUEntityRepresentative.SubjectIdentifierType.Fingerprint,
        Value = euRepresentativeEntityCertificateFingerprint
    })
    .WithPermissions(
        StandardPermissionType.InvoiceWrite,
        StandardPermissionType.InvoiceRead
        )
    .WithDescription("Representative for EU Entity")
    .Build();

OperationResponse grantRepresentativePermissionResponse = await KsefClient.GrantsPermissionEUEntityRepresentativeAsync(grantRepresentativePermissionsRequest,
    euAuthInfo.AccessToken.Token,
    CancellationToken.None);
```
Przykład w języku Java:
[EuEntityRepresentativePermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityRepresentativePermissionIntegrationTest.java)

```java
GrantEUEntityRepresentativePermissionsRequest request = new GrantEUEntityRepresentativePermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.FINGERPRINT, fingerprint))
        .withPermissions(List.of(EuEntityPermissionType.INVOICE_WRITE, EuEntityPermissionType.INVOICE_READ))
        .withDescription("Representative for EU Entity")
        .build();

OperationResponse response = ksefClient.grantsPermissionEUEntityRepresentative(request, accessToken);

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

Przykład w języku C#:
```csharp
OperationResponse operationResponse = await KsefClient.RevokeCommonPermissionAsync(permission.Id, accessToken, CancellationToken);
```

Przykład w języku Java:
[EntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EntityPermissionIntegrationTest.java)

```java
ksefClient.revokeCommonPermission(operationId, accessToken);
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
[KSeF.Client.Tests.Core\E2E\Permissions\EuEntityPermission\EuEntityPermissionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/EuEntityPermission/EuEntityPermissionE2ETests.cs)

```csharp
await ksefClient.RevokeAuthorizationsPermissionAsync(permissionId, accessToken, cancellationToken);
```

Przykład w języku Java:
[ProxyPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ProxyPermissionIntegrationTest.java)

```java
ksefClient.revokeAuthorizationsPermission(operationId, accessToken);
```


## Wyszukiwanie nadanych uprawnień

KSeF udostępnia zestaw endpointów pozwalających na odpytywanie listy aktywnych uprawnień nadanych użytkownikom i podmiotom. Mechanizmy te są niezbędne do audytu, przeglądu stanu dostępu, a także przy budowie interfejsów administracyjnych (np. do zarządzania strukturą dostępu w organizacji). Sekcja zawiera przegląd metod wyszukiwania z podziałem na kategorie nadanych uprawnień.

---
### Pobranie listy własnych uprawnień

Zapytanie pozwala na pobranie listy uprawnień posiadanych przez uwierzytelniony podmiot.
 Na tej liście znajdują się uprawnienia:
- nadane w sposób bezpośredni w bieżącym kontekście
- nadane przez podmiot nadrzędny
- nadane w sposób pośredni, gdzie kontekstem jest pośrednik lub podmiot docelowy
- nadane podmiotowi do obsługi faktur (`"InvoiceRead"` i `"InvoiceWrite"`) przez inny podmiot, jeśli uwierzytelniony podmiot ma uprawnienia właścicielskie (`"Owner"`) 

POST [/permissions/query/personal/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1personal~1grants/post)

Przykład w języku Java:
[SearchPersonalGrantPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SearchPersonalGrantPermissionIntegrationTest.java)

```java
QueryPersonalGrantRequest request = new QueryPersonalGrantRequest();
QueryPersonalGrantResponse response = ksefClient.searchPersonalGrantPermission(request, 0, 10, token.accessToken());

```
---
### Pobranie listy uprawnień do pracy w KSeF nadanych osobom fizycznym lub podmiotom

Zapytanie pozwala na pobranie listy uprawnień nadanych osobom fizycznym lub podmiotom – np. pracownikom firmy. Możliwe jest filtrowanie po rodzaju uprawnień, stanie (`Active` / `Inactive`), a także identyfikatorze nadawcy i odbiorcy. Endpoint ten bywa wykorzystywany przy onboardingu, audycie oraz monitoringu uprawnień personalnych.

POST [/permissions/query/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1persons~1grants/post)

| Pole                  | Opis                                                                 |
| :-------------------- | :------------------------------------------------------------------- |
| `authorIdentifier`    | Identyfikator podmiotu nadającego uprawnienia.   ```Nip```, ```Pesel```, ```Fingerprint```, ```System```                      |
| `authorizedIdentifier`| Identyfikator podmiotu, któremu nadano uprawnienia.      ```Nip```, ```Pesel```,```Fingerprint```             |
| `targetIdentifier`    | Identyfikator podmiotu docelowego (dla uprawnień pośrednich).  ```Nip```, ```AllPartners```      |
| `permissionTypes`     | Typy uprawnień do filtrowania.   `"CredentialsManage"`, `"CredentialsRead"`, `"InvoiceWrite"`, `"InvoiceRead"`, `"Introspection"`, `"SubunitManage"`, `"EnforcementOperations"`  |
| `permissionState`     | Stan uprawnienia.  ```Active``` / ```Inactive```                                                  |

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedPermissionsResponse<Client.Core.Models.Permissions.PersonPermission> response =
    await KsefClient
    .SearchGrantedPersonPermissionsAsync(
        personPermissionsQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Przykład w języku Java:
[PersonPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/PersonPermissionIntegrationTest.java)

```java
PersonPermissionsQueryRequest request = new PersonPermissionsQueryRequestBuilder()
        .withAuthorizedIdentifier(new PersonPermissionsAuthorizedIdentifier(PersonPermissionsAuthorizedIdentifier.IdentifierType.PESEL, personValue))
        .withQueryType(PersonPermissionQueryType.PERMISSION_GRANTED_IN_CURRENT_CONTEXT)
        .withPermissionTypes(List.of(PersonPermissionType.INVOICEWRITE, PersonPermissionType.INVOICEREAD))
        .build();

QueryPersonPermissionsResponse response = ksefClient.searchGrantedPersonPermissions(request, 0, 10, accessToken);

```
---
### Pobranie listy uprawnień administratorów jednostek i podmiotów podrzędnych

Ten endpoint służy do pobrania informacji o administratorach jednostek podrzędnych lub podmiotów podrzędnych (np. oddziałów, grup VAT). Pozwala na monitorowanie, kto posiada uprawnienia zarządcze względem danej struktury podrzędnej, identyfikowanej za pomocą `InternalId` lub `Nip`.

POST [/permissions/query/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subunits~1grants/post)

| Pole                  | Opis                                                                 |
| :-------------------- | :------------------------------------------------------------------- |
| `subjectIdentifier`    | Identyfikator podmiotu podrzędnego.   ```InternalId``` lub `Nip`            |

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
SubunitPermissionsQueryRequest subunitPermissionsQueryRequest = new SubunitPermissionsQueryRequest();
PagedPermissionsResponse<Client.Core.Models.Permissions.SubunitPermission> response =
    await KsefClient
    .SearchSubunitAdminPermissionsAsync(
        subunitPermissionsQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Przykład w języku Java:
[SubUnitPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubUnitPermissionIntegrationTest.java)

```java
SubunitPermissionsQueryRequest request = new SubunitPermissionsQueryRequestBuilder()
        .withSubunitIdentifier(new SubunitPermissionsSubunitIdentifier(SubunitPermissionsSubunitIdentifier.IdentifierType.INTERNALID, subUnitNip))
        .build();

QuerySubunitPermissionsResponse response = ksefClient.searchSubunitAdminPermissions(request, 0, 10, accessToken);


```
---
### Pobranie listy ról podmiotu

Endpoint zwraca zestaw ról przypisanych do kontekstu w ktorym jesteśmy uwierzytelnieni (czyli tego, w imieniu którego wykonywane jest zapytanie). Funkcja wykorzystywana głównie przy automatycznym sprawdzaniu dostępów przez systemy klienckie.

GET [/permissions/query/entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1entities~1roles/get)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedRolesResponse<EntityRole> response =
    await KsefClient
    .SearchEntityInvoiceRolesAsync(
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Przykład w języku Java:
[SearchEntityInvoiceRoleIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SearchEntityInvoiceRoleIntegrationTest.java)

```java
QueryEntityRolesResponse response = ksefClient.searchEntityInvoiceRoles(0, 10, token);
```
---
### Pobranie listy podmiotów podrzędnych

Pozwala na uzyskanie informacji o powiązanych podmiotach podrzędnych dla kontekstu w którym jesteśmy uwierzytelnieni (czyli tego, w imieniu którego wykonywane jest zapytanie). Funkcja głównie wykorzystywana w celu weryfikacji struktury jednostek samorządu terytorialnego lub grup VAT.

POST [/permissions/query/subordinate-entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subordinate-entities~1roles/post)

| Pole                     | Opis                                                                                                              |
| :----------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `subordinateEntityIdentifier`   | Identyfikator podmiotu, któremu nadano uprawnienia. ```Nip```                                                     |                                               |
    

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
SubordinateEntityRolesQueryRequest subordinateEntityRolesQueryRequest = new SubordinateEntityRolesQueryRequest();
PagedRolesResponse<SubordinateEntityRole> response =
    await KsefClient
    .SearchSubordinateEntityInvoiceRolesAsync(
        subordinateEntityRolesQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Przykład w języku Java:
[SearchSubordinateQueryIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SearchSubordinateQueryIntegrationTest.java)

```java
SubordinateEntityRolesQueryResponse response = ksefClient.searchSubordinateEntityInvoiceRoles(queryRequest, 0, 10,accessToken);
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
 

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedAuthorizationsResponse<AuthorizationGrant> response =
        await KsefClient
        .SearchEntityAuthorizationGrantsAsync(
            entityAuthorizationsQueryRequest,
            accessToken,
            pageOffset: 0,
            pageSize: 10,
            CancellationToken);
```

Przykład w języku Java:
[ProxyPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ProxyPermissionIntegrationTest.java)

```java
GrantAuthorizationPermissionsRequest request = new GrantAuthorizationPermissionsRequestBuilder()
        .withSubjectIdentifier(new SubjectIdentifier(SubjectIdentifier.IdentifierType.NIP, subjectNip))
        .withPermission(InvoicePermissionType.SELF_INVOICING)
        .withDescription("e2e test grant")
        .build();

OperationResponse response = ksefClient.grantsPermissionsProxyEntity(request, accessToken);

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
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
PagedPermissionsResponse<Client.Core.Models.Permissions.EuEntityPermission> response =
    await KsefClient
    .SearchGrantedEuEntityPermissionsAsync(
        euEntityPermissionsQueryRequest,
        accessToken,
        pageOffset: 0,
        pageSize: 10,
        CancellationToken);
```

Przykład w języku Java:
[EuEntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityPermissionIntegrationTest.java)

```java
EuEntityPermissionsQueryRequest request = new EuEntityPermissionsQueryRequestBuilder()
   .withAuthorizedFingerprintIdentifier(subjectContext)
   .build();

QueryEuEntityPermissionsResponse response = createKSeFClient().searchGrantedEuEntityPermissions(request, 0, 10, accessToken);
```

## Operacje 

Krajowy System e-Faktur umożliwia śledzenie i weryfikację statusu operacji związanych z zarządzaniem uprawnieniami. Każde nadanie lub odebranie uprawnienia jest realizowane jako asynchroniczna operacja, której status można monitorować przy użyciu unikalnego identyfikatora referencyjnego (`operationReferenceNumber`). Sekcja ta prezentuje mechanizm pobierania statusu operacji i jego interpretacji w kontekście automatyzacji i kontroli poprawności działań administracyjnych w KSeF.

### Pobranie statusu operacji

Po nadaniu lub odebraniu uprawnienia, system zwraca numer referencyjny operacji (`operationReferenceNumber`). Dzięki temu identyfikatorowi możliwe jest sprawdzenie aktualnego stanu przetwarzania żądania: czy zakończyło się sukcesem, czy wystąpił błąd, lub czy nadal trwa przetwarzanie. Informacja ta może być kluczowa w systemach nadzorczych, logice automatycznego ponawiania operacji lub raportowaniu działań administracyjnych. W tej sekcji przedstawiono przykład wywołania API służącego do pobrania statusu operacji.

GET [/permissions/operations/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Operacje/paths/~1api~1v2~1permissions~1operations~1%7BoperationReferenceNumber%7D/get)

Każda operacja nadania uprawnienia zwraca identyfikator operacji, który należy wykorzystać do sprawdzenia statusu tej operacji.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Permissions\SubunitPermission\SubunitPermissionsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Permissions/SubunitPermission/SubunitPermissionsE2ETests.cs)

```csharp
var operationStatus = await ksefClient.OperationsStatusAsync(referenceNumber, accessToken, cancellationToken);
```

Przykład w języku Java:
[EuEntityPermissionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/EuEntityPermissionIntegrationTest.java)

```java
PermissionStatusInfo operations = createKSeFClient().permissionOperationStatus(referenceNumber, accessToken);
```

### Sprawdzenie statusu zgody na wystawianie faktur z załącznikiem

Zgoda jest wymagana do wystawiania faktur zawierających załączniki i obowiązuje w obrębie bieżącego kontekstu (`ContextIdentifier`) użytego przy uwierzytelnieniu. Zgoda jest nadawana poza API, wyłącznie w usłudze e-Urząd Skarbowy, a zgłoszenia można składać od 1 stycznia 2026 r. API nie udostępnia operacji złożenia zgody

GET [/permissions/attachments/status](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Operacje/paths/~1api~1v2~1permissions~1attachments~1status/get)

Zwraca status zgody dla bieżącego kontekstu. Jeżeli zgoda nie jest aktywna, faktura z załącznikiem wysłana do API KSeF zostanie odrzucona.

**Środowisko testowe**  
Na środowisku testowym dostępny jest endpoint POST `/testdata/attachment`, który nadaje możliwość wysyłania faktur z załącznikiem przez wskazany podmiot. Endpoint służy wyłącznie do zasymulowania nadania zgody w testach i działa w zakresie bieżącego kontekstu.