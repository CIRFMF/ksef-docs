## Certyfikaty KSeF
27.06.2025

### Wstęp
Certyfikat KSeF to cyfrowe poświadczenie tożsamości podmiotu, wydawane przez system KSeF na wniosek użytkownika. Certyfikat ten może być wykorzystywany do:

* uwierzytelniania się w systemie KSeF,
* realizacji operacji w trybie offline, w tym wystawiania faktur bezpośrednio w aplikacji użytkownika.


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
```csharp
var limits = await ksefClient.GetCertificateLimitsAsync();
```

Przykład w języku Java:

```java
var limits = ksefClient.getCertificateLimits();
```

### 2. Pobranie danych do wniosku certyfikacyjnego

Aby rozpocząć proces aplikowania o certyfikat KSeF, należy pobrać zestaw danych identyfikacyjnych, które zostaną automatycznie wykorzystane do wygenerowania żądania certyfikacyjnego (CSR).

GET [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get)

Dane te są zwracane na podstawie certyfikatu użytego w procesie uwierzytelnienia i odzwierciedlają tożsamość podmiotu, który składa wniosek o certyfikat.

**Uwaga**: Pobranie danych certyfikacyjnych jest możliwe wyłącznie po uwierzytelnieniu z wykorzystaniem podpisu (XAdES). Uwierzytelnienie przy użyciu tokena systemowego KSeF nie pozwala na złożenie wniosku o certyfikat.


Przykład w języku C#:
```csharp
var enrollmentData = await kSeFClient
    .GetCertificateEnrollmentDataAsync(accessToken, cancellationToken)
    .ConfigureAwait(false);
```

Przykład w języku Java:
```java
CertificateEnrollmentsInfoResponse enrollmentDaa = ksefClient.getCertificateEnrollmentInfo();
```

Oto pełna lista pól, które mogą być zwrócone, przedstawiona w formie tabeli zawierającej OID:

| OID          | Nazwa (ang.)               | Opis                                           |
|--------------|----------------------------|------------------------------------------------|
| 2.5.4.3      | commonName                 | Nazwa powszechna                               |
| 2.5.4.4      | surname                    | Nazwisko                                       |
| 2.5.4.5      | serialNumber               | Numer seryjny (np. PESEL, NIP)                 |
| 2.5.4.6      | countryName                | Kod kraju (np. PL)                             |
| 2.5.4.10     | organizationName           | Nazwa organizacji/firma                        |
| 2.5.4.42     | givenName                  | Imię lub imiona                                |
| 2.5.4.45     | uniqueIdentifier           | Unikalny identyfikator (opcjonalny)            |
| 2.5.4.97     | organizationIdentifier     | Identyfikator organizacji (np. NIP)            |

Atrybut `givenName` może pojawić się wielokrotnie i zwracany jest w postaci listy wartości. 

### 3. Przygotowanie CSR (Certificate Signing Request)
Aby złożyć wniosek o certyfikat KSeF, należy przygotować tzw. żądanie podpisania certyfikatu (CSR) w formacie PKCS#10. CSR zawiera:
* informacje identyfikujące podmiot (DN – Distinguished Name),
* klucz publiczny, który zostanie powiązany z certyfikatem.

Wszystkie dane identyfikacyjne (atrybuty X.509) powinny być zgodne z wartościami zwróconymi przez system w poprzednim kroku (/certificates/enrollment/data). Zmodyfikowanie tych danych spowoduje odrzucenie wniosku.

Przykład w języku C# (z użyciem ```ICryptographyService```):


```csharp
(var csrBase64encoded, var privateKeyBase64Encoded) = cryptographyService.GenerateCsr(enrollmentInfo);
```


Przykład w języku Java:
```java
CsrResult csr = cryptographyService.generateCsr(enrollmentInfo);
```

* ```csrBase64Encoded``` – zawiera żądanie CSR zakodowane w formacie Base64, gotowe do wysłania do KSeF
* ```privateKeyBase64Encoded``` – zawiera klucz prywatny powiązany z wygenerowanym CSR, zakodowany w Base64. Klucz ten będzie potrzebny do uwierzytelniania i operacji podpisu przy użyciu certyfikatu.

**Uwaga**: Klucz prywatny powinien być przechowywany w sposób bezpieczny i zgodny z polityką bezpieczeństwa danej organizacji.

### 4. Wysłanie wniosku certyfikacyjnego
Po przygotowaniu żądania certyfikacyjnego (CSR) należy przesłać je do systemu KSeF za pomocą wywołania 

POST [/certificates/enrollments](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post)

W przesyłanym wniosku należy podać:
* **nazwę certyfikatu** – widoczną później w metadanych certyfikatu, ułatwiającą identyfikację,
* **CSR** w formacie PKCS#10 (DER), zakodowany jako ciąg Base64,
* (opcjonalnie) **datę rozpoczęcia ważności certyfikatu** – jeśli nie zostanie wskazana, certyfikat będzie ważny od chwili jego wystawienia.

Upewnij się, że CSR zawiera dokładnie te same dane, które zostały zwrócone przez endpoint /certificates/enrollments/data.

Przykład w języku C#:
```csharp
 var request = SendCertificateEnrollmentRequestBuilder.Create()
            .WithCertificateName("Testowy certyfikat")
            .WithCsr(csrBase64encoded)
            .WithValidFrom(DateTimeOffset.UtcNow.AddDays(1)) // Certyfikat będzie ważny od jutra
            .Build();

  var certificateEnrollmentResponse = await kSeFClient.SendCertificateEnrollmentAsync(request, accessToken, cancellationToken)
```

Przykład w języku Java:
```java
var request = new SendCertificateEnrollmentRequestBuilder()
    .withCertificateName("certificateName")
    .withCsr(csr.csr())
    .withValidFrom(OffsetDateTime.now())
    .build();

return ksefClient.sendCertificateEnrollment(request);
```

W odpowiedzi otrzymasz ```referenceNumber```, który umożliwia monitorowanie statusu wniosku oraz późniejsze pobranie wystawionego certyfikatu.

### 5. Sprawdzenie statusu wniosku

Proces wystawiania certyfikatu ma charakter asynchroniczny. Oznacza to, że system nie zwraca certyfikatu natychmiast po złożeniu wniosku, lecz umożliwia jego późniejsze pobranie po zakończeniu przetwarzania.
Status wniosku należy okresowo sprawdzać, używając numeru referencyjnego (```referenceNumber```), który został zwrócony w odpowiedzi na wysłanie wniosku (/certificates/enrollments).

GET [/certificates/enrollments/\{referenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1%7BreferenceNumber%7D/get)

Jeżeli wniosek certyfikacyjny zostanie odrzucony, w odpowiedzi otrzymamy informacje o błędzie.

Przykład w języku C#:

```csharp
await kSeFClient.GetCertificateEnrollmentStatusAsync(certificateEnrollmentResponse.ReferenceNumber, accessToken, cancellationToken)
```

Przykład w języku Java:

```java
ksefClient.getCertificateEnrollmentStatus(referenceNumber);
```

Po uzyskaniu numeru seryjnego certyfikatu (```certificateSerialNumber```), możliwe jest pobranie jego zawartości i metadanych w kolejnych krokach procesu.

## 6. Pobieranie listy certyfikatów

System KSeF umożliwia pobranie treści wcześniej wystawionych certyfikatów wewnętrznych na podstawie listy numerów seryjnych. Każdy certyfikat zwracany jest w formacie DER, zakodowanym jako ciąg Base64.

POST [/certificates/retrieve](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1retrieve/post)

Przykład w języku C#:

```csharp
var certificates = await kSeFClient.GetCertificateListAsync(accessToken, new CertificateListRequest { CertificateSerialNumbers = certificateSerialNumber}, cancellationToken)
    .ConfigureAwait(false);
```

Przykład w języku Java:

```java
var certificates = ksefClient.getCertificateList(new CertificateListRequest(List.of(certificateSerialNumber)))
        .getCertificates();
```

Struktura odpowiedzi:
Każdy element odpowiedzi zawiera:

| Pole                      | Opis    |
|---------------------------|------------------------|
| `certificateSerialNumber` | `Numer seryjny certyfikatu`          |
| `certificateName` | `Nazwa certyfikatu nadana przy rejestracji`          |
| `certificateBase64` | `Treść certyfikatu zakodowana w Base64 (format DER)`          |

Otrzymane certyfikaty możemy między innymi wykorzystać do [uwierzytelniania](uwierzytelnianie.md) w systemie KSeF.

## 7. Pobieranie listy metadanych certyfikatów

Dostępna jest możliwość pobrania listy certyfikatów wewnętrznych złożonych przez dany podmiot. Dane te obejmują zarówno aktywne, jak i historyczne certyfikaty, wraz z ich statusem, zakresem ważności oraz identyfikatorami.

GET [/certificates/query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1query/post)

#### Parametry filtrowania (opcjonalne): ####
* `status` - status certyfikatu (`Active`, `Blocked`, `Revoked`, `Expired`)
* `expiresAfter` - data końca ważności certyfikatu (opcjonalny)
* `name` - nazwa certyfikatu (opcjonalny)
* `certificateSerialNumber` - numer seryjny certyfikatu (opcjonalny)
* `pageSize` - numer strony (domyślnie 10)
* `pageOffset` - offset strony (domyślnie 0)`

Przykład w języku C#:
```csharp
var request = GetCertificateMetadataListRequestBuilder.Create()
            .WithCertificateSerialNumber("<certificateSerialNumber>") // Optional filter
            .WithName("<name>") // Optional filter
            .WithStatus(CertificateStatusEnum.Active) // Optional filter
            .WithPageSize(10)
            .Build();

var metadataList = await kSeFClient.GetCertificateMetadataListAsync(request, accessToken, cancellationToken)
            .ConfigureAwait(false);
```
Przykład w języku Java:

```java
var request = new CertificateMetadataListRequestBuilder()
        .withName("name")
        .withCertificateSerialNumber("certificateSerialNumber")
        .withStatus(CertificateListItemStatus.ACTIVE)
        .build();
ksefClient.getCertificateMetadataList(request, 10, 0)
        .getCertificates();
```

W odpowiedzi otrzymamy metadane certyfikatów:


| Pole                      | Przykładowa wartość    |
|---------------------------|------------------------|
| `certificateSerialNumber` | `00035434535`          |
| `name`                    | `Certyfikat 1`         |
| `commonName`              | `Jan Kowalski`         |
| `status`                  | `Active`               |
| `subjectIdentifier`       | `1234445678`           |
| `subjectIdentifierType`   | `Nip`                  |
| `validFrom`               | `2019-08-24T14:15:22Z` |
| `validTo`                 | `2019-08-24T14:15:22Z` |
| `lastUseDate`             | `2019-08-24T14:15:22Z` |



## 8. Unieważnianie certyfikatu

Certyfikat KSeF może zostać unieważniony tylko przez właściciela w przypadku kompromitacji klucza prywatnego, zakończenia jego użycia lub zmiany organizacyjnej. Po unieważnieniu certyfikat nie może być użyty do dalszego uwierzytelniania ani realizacji operacji w systemie KSeF.
Unieważnienie realizowane jest na podstawie numeru seryjnego certyfikatu (```certificateSerialNumber```) oraz opcjonalnego powodu odwołania.

POST [/certificates/\{certificateSerialNumber\}/revoke](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1%7BcertificateSerialNumber%7D~1revoke/post)

Przykład w języku C#:

```csharp
var request = RevokeCertificateRequestBuilder.Create()
    .WithRevocationReason(CertificateRevocationReason.KeyCompromise) // optional
    .Build();

await kSeFClient.RevokeCertificateAsync(request, certificateSerialNumber, accessToken, cancellationToken)
     .ConfigureAwait(false);
```

Przykład w języku Java:

```java
var request = new RevokeCertificateRequestBuilder()
        .withRevocationReason(CertificateRevocationReason.KEYCOMPROMISE)
        .build();
ksefClient.revokeCertificate(request, certificateSerialNumber);
```

Po unieważnieniu certyfikat nie może zostać ponownie wykorzystany. Jeśli zajdzie potrzeba jego dalszego użycia, należy wystąpić o nowy certyfikat.
