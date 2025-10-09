## Certyfikaty KSeF
31.08.2025

### Wstęp
Certyfikat KSeF to cyfrowe poświadczenie tożsamości podmiotu, wydawane przez system KSeF na wniosek użytkownika.

Wniosek o wydanie certyfikatu KSeF może zostać złożony wyłącznie dla danych, które znajdują się w certyfikacie wykorzystanym do [uwierzytelnienia](uwierzytelnianie.md). Na podstawie tych danych endpoint [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get)
 zwraca dane identyfikacyjne, które muszą zostać użyte w żądaniu certyfikacyjnym.

>System nie pozwala na wystąpienie o certyfikat w imieniu innego podmiotu.

Dostępne są dwa typy certyfikatów – każdy certyfikat może mieć **tylko jeden typ** (`Authentication` albo `Offline`). Nie jest możliwe wystawienie certyfikatu łączącego obie funkcje.

| Typ              | Opis |
| ---------------- | ---- |
| `Authentication` | Certyfikat przeznaczony do uwierzytelniania w systemie KSeF.<br/>**keyUsage:** Digital Signature (80) |
| `Offline`        | Certyfikat przeznaczony wyłącznie do wystawiania faktur w trybie offline. Używany do potwierdzania autentyczności wystawcy i integralności faktury poprzez [kod QR II](kody-qr.md). Nie umożliwia uwierzytelnienia.<br/>**keyUsage:** Non-Repudiation (40) |

#### Proces uzyskania certyfikatu
Proces aplikowania o certyfikat składa się z kilku etapów:
1. Sprawdzenie dostępnych limitów,
2. Pobranie danych do wniosku certyfikacyjnego,
3. Wysłanie wniosku,
4. Pobranie wystawionego certyfikatu,


### 1. Sprawdzenie limitów

Zanim klient API złoży wniosek o wydanie nowego certyfikatu zaleca się weryfikację limitu certyfikatów.

API udostępnia informacje na temat:
* maksymalnej liczby certyfikatów, którą można dysponować,
* liczby aktualnie aktywnych certyfikatów,
* możliwości złożenia kolejnego wniosku.

GET [/certificates/limits](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1limits/get)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)
```csharp
CertificateLimitResponse certificateLimitResponse = await KsefClient
    .GetCertificateLimitsAsync(accessToken, CancellationToken);
```

Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateLimitsResponse response = ksefClient.getCertificateLimits(accessToken);
```

### 2. Pobranie danych do wniosku certyfikacyjnego

Aby rozpocząć proces aplikowania o certyfikat KSeF, należy pobrać zestaw danych identyfikacyjnych, które system zwróci w odpowiedzi na wywołanie endpointu  
GET [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get).

Dane te są odczytywane z certyfikatu użytego do uwierzytelnienia, którym może być:
- kwalifikowany certyfikat osoby fizycznej – zawierający numer PESEL albo NIP,
- kwalifikowany certyfikat organizacji (tzw. pieczęć firmowa) – zawierający numer NIP,
- Profil Zaufany (ePUAP) – wykorzystywany przez osoby fizyczne, zawiera numer PESEL,
- certyfikat wewnętrzny KSeF – wystawiany przez system KSeF, nie jest certyfikatem kwalifikowanym, ale jest honorowany w procesie uwierzytelniania.

System na tej podstawie zwraca komplet atrybutów DN (X.500 Distinguished Name), które muszą zostać użyte przy budowie żądania certyfikacyjnego (CSR). Modyfikacja tych danych spowoduje odrzucenie wniosku.

**Uwaga**: Pobranie danych certyfikacyjnych jest możliwe wyłącznie po uwierzytelnieniu z wykorzystaniem podpisu (XAdES). Uwierzytelnienie przy użyciu tokena systemowego KSeF nie pozwala na złożenie wniosku o certyfikat.


Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)
```csharp
CertificateEnrollmentsInfoResponse certificateEnrollmentsInfoResponse =
    await KsefClient.GetCertificateEnrollmentDataAsync(accessToken, CancellationToken);
```

Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateEnrollmentsInfoResponse response = ksefClient.getCertificateEnrollmentInfo(accessToken);
```

Oto pełna lista pól, które mogą być zwrócone, przedstawiona w formie tabeli zawierającej OID:

| OID      | Nazwa (ang.)          | Opis                                   | Osoba fizyczna | Pieczęć firmowa |
|----------|-----------------------|----------------------------------------|----------------|-----------------|
| 2.5.4.3  | commonName            | Nazwa powszechna                       | ✔️             | ✔️              |
| 2.5.4.4  | surname               | Nazwisko                               | ✔️             | ❌              |
| 2.5.4.5  | serialNumber          | Numer seryjny (np. PESEL, NIP)         | ✔️             | ❌              |
| 2.5.4.6  | countryName           | Kod kraju (np. PL)                     | ✔️             | ✔️              |
| 2.5.4.10 | organizationName      | Nazwa organizacji / firma              | ❌             | ✔️              |
| 2.5.4.42 | givenName             | Imię lub imiona                        | ✔️             | ❌              |
| 2.5.4.45 | uniqueIdentifier      | Unikalny identyfikator (opcjonalny)    | ✔️             | ✔️              |
| 2.5.4.97 | organizationIdentifier| Identyfikator organizacji (np. NIP)    | ❌             | ✔️              |


Atrybut `givenName` może pojawić się wielokrotnie i zwracany jest w postaci listy wartości. 

### 3. Przygotowanie CSR (Certificate Signing Request)
Aby złożyć wniosek o certyfikat KSeF, należy przygotować tzw. żądanie podpisania certyfikatu (CSR) w standardzie PKCS#10, w formacie DER, zakodowane w Base64. CSR zawiera:
* informacje identyfikujące podmiot (DN – Distinguished Name),
* klucz publiczny, który zostanie powiązany z certyfikatem.

Wymagania dotyczące klucza prywatnego użytego do podpisu CSR:
* Typy dozwolone:
  * RSA (OID: 1.2.840.113549.1.1.1), długość klucza: 2048 bitów,
  * EC (klucze eliptyczne, OID: 1.2.840.10045.2.1), krzywa NIST P-256 (secp256r1).
* Zalecane jest stosowanie kluczy EC.

* Dozwolone algorytmy podpisu:
  * RSA PKCS#1 v1.5,
  * RSA PSS,
  * ECDSA (format podpisu zgodny z RFC 3279).

* Dozwolone funkcje skrótu użyte do podpisu CSR:
  * SHA1,
  * SHA256,
  * SHA384,
  * SHA512.

Wszystkie dane identyfikacyjne (atrybuty X.509) powinny być zgodne z wartościami zwróconymi przez system w poprzednim kroku (/certificates/enrollments/data). Zmodyfikowanie tych danych spowoduje odrzucenie wniosku.

Przykład w języku C# (z użyciem ```ICryptographyService```):
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
var (csr, key) = CryptographyService.GenerateCsrWithRSA(TestFixture.EnrollmentInfo);
```


Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CsrResult csr = defaultCryptographyService.generateCsrWithRsa(enrollmentInfo);
```

* ```csrBase64Encoded``` – zawiera żądanie CSR zakodowane w formacie Base64, gotowe do wysłania do KSeF
* ```privateKeyBase64Encoded``` – zawiera klucz prywatny powiązany z wygenerowanym CSR, zakodowany w Base64. Klucz ten będzie potrzebny do operacji podpisu przy użyciu certyfikatu.

**Uwaga**: Klucz prywatny powinien być przechowywany w sposób bezpieczny i zgodny z polityką bezpieczeństwa danej organizacji.

### 4. Wysłanie wniosku certyfikacyjnego
Po przygotowaniu żądania certyfikacyjnego (CSR) należy przesłać je do systemu KSeF za pomocą wywołania 

POST [/certificates/enrollments](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post)

W przesyłanym wniosku należy podać:
* **nazwę certyfikatu** – widoczną później w metadanych certyfikatu, ułatwiającą identyfikację,
* **typ certyfikatu** – `Authentication` lub `Offline`,
* **CSR** w formacie PKCS#10 (DER), zakodowany jako ciąg Base64,
* (opcjonalnie) **validFrom** – datę rozpoczęcia ważności. Jeśli nie zostanie wskazana, certyfikat będzie ważny od chwili jego wystawienia.

Upewnij się, że CSR zawiera dokładnie te same dane, które zostały zwrócone przez endpoint /certificates/enrollments/data.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
SendCertificateEnrollmentRequest sendCertificateEnrollmentRequest = SendCertificateEnrollmentRequestBuilder
    .Create()
    .WithCertificateName(TestCertificateName)
    .WithCertificateType(CertificateType.Authentication)
    .WithCsr(csr)
    .WithValidFrom(DateTimeOffset.UtcNow.AddDays(CertificateValidityDays))
    .Build();

CertificateEnrollmentResponse certificateEnrollmentResponse = await KsefClient
    .SendCertificateEnrollmentAsync(sendCertificateEnrollmentRequest, accessToken, CancellationToken);
```

Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
SendCertificateEnrollmentRequest request = new SendCertificateEnrollmentRequestBuilder()
        .withValidFrom(OffsetDateTime.now().toString())
        .withCsr(csr.csr())
        .withCertificateName("certificate")
        .withCertificateType(CertificateType.AUTHENTICATION)
        .build();

CertificateEnrollmentResponse response = ksefClient.sendCertificateEnrollment(request, accessToken);
```

W odpowiedzi otrzymasz ```referenceNumber```, który umożliwia monitorowanie statusu wniosku oraz późniejsze pobranie wystawionego certyfikatu.

### 5. Sprawdzenie statusu wniosku

Proces wystawiania certyfikatu ma charakter asynchroniczny. Oznacza to, że system nie zwraca certyfikatu natychmiast po złożeniu wniosku, lecz umożliwia jego późniejsze pobranie po zakończeniu przetwarzania.
Status wniosku należy okresowo sprawdzać, używając numeru referencyjnego (```referenceNumber```), który został zwrócony w odpowiedzi na wysłanie wniosku (/certificates/enrollments).

GET [/certificates/enrollments/\{referenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1%7BreferenceNumber%7D/get)

Jeżeli wniosek certyfikacyjny zostanie odrzucony, w odpowiedzi otrzymamy informacje o błędzie.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateEnrollmentStatusResponse certificateEnrollmentStatusResponse = await KsefClient
    .GetCertificateEnrollmentStatusAsync(TestFixture.EnrollmentReference, accessToken, CancellationToken);
```

Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateEnrollmentStatusResponse response = ksefClient.getCertificateEnrollmentStatus(referenceNumber, accessToken);

```

Po uzyskaniu numeru seryjnego certyfikatu (```certificateSerialNumber```), możliwe jest pobranie jego zawartości i metadanych w kolejnych krokach procesu.

### 6. Pobieranie listy certyfikatów

System KSeF umożliwia pobranie treści wcześniej wystawionych certyfikatów wewnętrznych na podstawie listy numerów seryjnych. Każdy certyfikat zwracany jest w formacie DER, zakodowanym jako ciąg Base64.

POST [/certificates/retrieve](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1retrieve/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)

```csharp
CertificateListRequest certificateListRequest = new CertificateListRequest { CertificateSerialNumbers = TestFixture.SerialNumbers };

CertificateListResponse certificateListResponse = await KsefClient
    .GetCertificateListAsync(certificateListRequest, accessToken, CancellationToken);
```

Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateListResponse certificateResponse = ksefClient.getCertificateList(new CertificateListRequest(List.of(certificateSerialNumber)), accessToken);
```

Każdy element odpowiedzi zawiera:

| Pole                      | Opis    |
|---------------------------|------------------------|
| `certificateSerialNumber` | Numer seryjny certyfikatu          |
| `certificateName` | Nazwa certyfikatu nadana przy rejestracji          |
| `certificate` | Treść certyfikatu zakodowana w Base64 (format DER)          |
| `certificateType` | Typ certyfikatu (`Authentication`, `Offline`).          |

### 7. Pobieranie listy metadanych certyfikatów

Dostępna jest możliwość pobrania listy certyfikatów wewnętrznych złożonych przez dany podmiot. Dane te obejmują zarówno aktywne, jak i historyczne certyfikaty, wraz z ich statusem, zakresem ważności oraz identyfikatorami.

POST [/certificates/query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1query/post)

Parametry filtrowania (opcjonalne):
* `status` - status certyfikatu (`Active`, `Blocked`, `Revoked`, `Expired`)
* `expiresAfter` - data końca ważności certyfikatu (opcjonalna)
* `name` - nazwa certyfikatu (opcjonalny)
* `type` - typ certyfikatu (`Authentication`, `Offline`) (opcjonalny)
* `certificateSerialNumber` - numer seryjny certyfikatu (opcjonalny)
* `pageSize` - liczba elementów na stronie (domyślnie 10)
* `pageOffset` - numer strony wyników (domyślnie 0)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)
```csharp
var request = GetCertificateMetadataListRequestBuilder
    .Create()
    .WithCertificateSerialNumber(serialNumber)
    .WithName(name)
    .Build();
    CertificateMetadataListResponse certificateMetadataListResponse = await KsefClient
            .GetCertificateMetadataListAsync(accessToken, requestPayload, pageSize, pageOffset, CancellationToken);
```
Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
QueryCertificatesRequest request = new CertificateMetadataListRequestBuilder().build();

CertificateMetadataListResponse response = ksefClient.getCertificateMetadataList(request, 10, 0, accessToken);


```

W odpowiedzi otrzymamy metadane certyfikatów.



### 8. Unieważnianie certyfikatów

Certyfikat KSeF może zostać unieważniony tylko przez właściciela w przypadku kompromitacji klucza prywatnego, zakończenia jego użycia lub zmiany organizacyjnej. Po unieważnieniu certyfikat nie może być użyty do dalszego uwierzytelniania ani realizacji operacji w systemie KSeF.
Unieważnienie realizowane jest na podstawie numeru seryjnego certyfikatu (```certificateSerialNumber```) oraz opcjonalnego powodu odwołania.

POST [/certificates/\{certificateSerialNumber\}/revoke](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1%7BcertificateSerialNumber%7D~1revoke/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Certificates\CertificatesE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Certificates/CertificatesE2ETests.cs)
```csharp
CertificateRevokeRequest certificateRevokeRequest = RevokeCertificateRequestBuilder
        .Create()
        .WithRevocationReason(CertificateRevocationReason.KeyCompromise)
        .Build();

await ksefClient.RevokeCertificateAsync(request, certificateSerialNumber, accessToken, cancellationToken)
     .ConfigureAwait(false);
```

Przykład w języku Java:
[CertificateIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/CertificateIntegrationTest.java)

```java
CertificateRevokeRequest request = new CertificateRevokeRequestBuilder()
        .withRevocationReason(CertificateRevocationReason.KEYCOMPROMISE)
        .build();

ksefClient.revokeCertificate(request, serialNumber, accessToken);
```

Po unieważnieniu certyfikat nie może zostać ponownie wykorzystany. Jeśli zajdzie potrzeba jego dalszego użycia, należy wystąpić o nowy certyfikat.
