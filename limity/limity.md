# Limity
21.10.2025

## Wstęp

W systemie KSeF 2.0 zastosowano mechanizmy limitujące liczbę i wielkość operacji API oraz parametry związane z przesyłanymi danymi. Celem tych limitów jest:
- ochrona stabilności systemu przy dużej skali działania,
- przeciwdziałanie nadużyciom i nieefektywnym integracjom,
- zapobieganie nadużyciom i potencjalnym zagrożeniom cyberbezpieczeństwa,
- zapewnienie równych warunków dostępu dla wszystkich użytkowników.

Limity zostały zaprojektowane z możliwością elastycznego dostosowania do potrzeb konkretnych podmiotów wymagających większej intensywności operacji.

## Limity żądań API
System KSeF ogranicza liczbę zapytań, jakie można wysyłać w krótkim czasie, aby zapewnić stabilne działanie systemu i równy dostęp dla wszystkich użytkowników.
Więcej informacji znajduje się w [Limity żądań API](limity-api.md).

## Limity na kontekst

| Parametr                                                    | Wartość domyślna                       |
| ----------------------------------------------------------- | -------------------------------------- |
| Maksymalny rozmiar faktury bez załącznika                | 1 MB                                  |
| Maksymalny rozmiar faktury z załącznikiem                 | 3 MB                                  |
| Maksymalna liczba faktur w sesji interaktywnej/wsadowej | 10 000                                 |

## Limity na uwierzytelniony podmiot

### Wnioski i aktywne certyfikaty

| Identyfikator z certyfikatu            | Wnioski o certyfikat KSeF | Aktywne certyfikaty KSeF |
| -------------------------------------- | ------------------------- | ------------------------ |
| NIP                                    | 300                       | 100                      |
| PESEL                                  | 6                         | 2                        |
| Odcisk palca certyfikatu (fingerprint) | 6                         | 2                        |



## Dostosowanie limitów

System KSeF umożliwia indywidualne dostosowanie wybranych limitów technicznych dla:
- limitów API - np. zwiększenie liczby żądań dla wybranego endpointu,
- kontekstu - np. zwiększenie maksymalnego rozmiaru faktury,
- podmiotu uwierzytelniającego - np. zwiększenie limitów aktywnych certyfikatów KSeF dla osoby fizycznej (PESEL).

Na **środowisku produkcyjnym** zwiększenie limitów możliwe jest wyłącznie na podstawie uzasadnionego wniosku, popartego realną potrzebą operacyjną.
Wniosek składa się za pośrednictwem [formularza kontaktowego](https://ksef.podatki.gov.pl/formularz/), wraz ze szczegółowym opisem zastosowania.

## Sprawdzanie indywidualnych limitów
System KSeF udostępnia endpointy pozwalające na sprawdzenie aktualnych wartości limitów dla bieżącego kontekstu lub podmiotu:

### Pobranie limitów dla bieżącego kontekstu

GET [/limits/context](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1limits~1context/get)

Zwraca wartości obowiązujących limitów sesji interaktywnych i wsadowych dla bieżącego kontekstu.

Przykład w języku C#:
[KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs)
```csharp
Client.Core.Models.TestData.SessionLimitsInCurrentContextResponse limitsForContext =
    await LimitsClient.GetLimitsForCurrentContextAsync(
        accessToken,
        CancellationToken);
```
Przykład w języku Java:

[ContextLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ContextLimitIntegrationTest.java)

```java
GetContextLimitResponse response = ksefClient.getContextSessionLimit(accessToken);
```

### Pobranie limitów dla bieżącego podmiotu

GET [/limits/subject](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1limits~1subject/get)

Zwraca obowiązujące limity certyfikatów i wniosków certyfikacyjnych dla bieżącego podmiotu uwierzytelnionego.

Przykład w języku C#:
[KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs)
```csharp
Client.Core.Models.TestData.CertificatesLimitInCurrentSubjectResponse limitsForSubject =
        await LimitsClient.GetLimitsForCurrentSubjectAsync(
            accessToken,
            CancellationToken);
```

Przykład w języku Java:

[SubjectLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubjectLimitIntegrationTest.java)

```java
GetSubjectLimitResponse response = ksefClient.getSubjectCertificateLimit(accessToken);
```

## Modyfikacja limitów na środowisku testowym

Na **środowisku testowym** udostępniono zestaw metod umożliwiających zmianę oraz przywracanie limitów do wartości domyślnych.
Operacje te dostępne są wyłącznie dla uwierzytelnionych podmiotów i nie mają wpływu na środowisko produkcyjne.

### Zmiana limitów sesji dla bieżącego kontekstu

POST [/testdata/limits/context/session](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1testdata~1limits~1context~1session/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs)

```csharp
Client.Core.Models.TestData.ChangeSessionLimitsInCurrentContextRequest newLimits =
    new()
    {
        OnlineSession = new Client.Core.Models.TestData.SessionLimits
        {
            MaxInvoices = newMaxInvoices,
            MaxInvoiceSizeInMB = newMaxInvoiceSizeInMB
            MaxInvoiceWithAttachmentSizeInMB = newMaxInvoiceWithAttachmentSizeInMB
        },

        BatchSession = new Client.Core.Models.TestData.SessionLimits
        {
            MaxInvoices = newBatchSessionMaxInvoices
            MaxInvoiceSizeInMB = newBatchSessionMaxInvoiceSizeInMB,
            MaxInvoiceWithAttachmentSizeInMB = newBatchSessionMaxInvoiceWithAttachmentSizeInMB,
        }
    };

await TestDataClient.ChangeSessionLimitsInCurrentContextAsync(
    newLimits,
    accessToken);
```

Przykład w języku Java:

[ContextLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ContextLimitIntegrationTest.java)

```java
ChangeContextLimitRequest request = new ChangeContextLimitRequest();
OnlineSessionLimit onlineSessionLimit = new OnlineSessionLimit();
onlineSessionLimit.setMaxInvoiceSizeInMB(4);
onlineSessionLimit.setMaxInvoiceWithAttachmentSizeInMB(5);
onlineSessionLimit.setMaxInvoices(6);

BatchSessionLimit batchSessionLimit = new BatchSessionLimit();
batchSessionLimit.setMaxInvoiceSizeInMB(4);
batchSessionLimit.setMaxInvoiceWithAttachmentSizeInMB(5);
batchSessionLimit.setMaxInvoices(6);

request.setOnlineSession(onlineSessionLimit);
request.setBatchSession(batchSessionLimit);

ksefClient.changeContextLimitTest(request, accessToken);
```

### Przywrócenie limitów sesji dla kontekstu do wartości domyślnych

DELETE [/testdata/limits/context/session](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1testdata~1limits~1context~1session/delete)

Przykład w języku C#:
[KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs)

```csharp
await TestDataClient.RestoreDefaultSessionLimitsInCurrentContextAsync(accessToken);
```

Przykład w języku Java:
[ContextLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/ContextLimitIntegrationTest.java)

```java
ksefClient.resetContextLimitTest(accessToken);
```

### Zmiana limitów certyfikatów dla bieżącego podmiotu

POST [/testdata/limits/subject/certificate](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1testdata~1limits~1subject~1certificate/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs)

```csharp
Client.Core.Models.TestData.ChangeCertificatesLimitInCurrentSubjectRequest newCertificateLimitsForSubject = new()
{
    SubjectIdentifierType = Client.Core.Models.TestData.TestDataSubjectIdentifierType.Nip,
    Certificate = new Client.Core.Models.TestData.TestDataCertificate
    {
        MaxCertificates = newMaxCertificatesValue
    },
    Enrollment = new Client.Core.Models.TestData.TestDataEnrollment
    {
        MaxEnrollments = newMaxEnrollmentsValue
    }
};

await TestDataClient.ChangeCertificatesLimitInCurrentSubjectAsync(
    newCertificateLimitsForSubject,
    accessToken);
```

Przykład w języku Java:
[SubjectLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubjectLimitIntegrationTest.java)

```java
ChangeSubjectCertificateLimitRequest request = new ChangeSubjectCertificateLimitRequest();
request.setCertificate(new CertificateLimit(15));
request.setEnrollment(new EnrollmentLimit(15));
request.setSubjectIdentifierType(ChangeSubjectCertificateLimitRequest.SubjectType.NIP);

ksefClient.changeSubjectLimitTest(request, accessToken);
```

### Przywrócenie limitów certyfikatów dla podmiotu do wartości domyślnych ###

DELETE [/testdata/limits/subject/certificate](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1testdata~1limits~1subject~1certificate/delete)

Przykład w języku C#:
[KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Limits/LimitsE2ETests.cs)

```csharp
await TestDataClient.RestoreDefaultCertificatesLimitInCurrentSubjectAsync(accessToken);
```

Przykład w języku Java:
[SubjectLimitIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SubjectLimitIntegrationTest.java)

```java
ksefClient.resetSubjectCertificateLimit(accessToken);
```

Powiązane dokumenty: 
- [Limity żądań api](limity-api.md)
- [Weryfikacja faktury](../faktury/weryfikacja-faktury.md)
- [Certyfikaty KSeF](../certyfikaty-KSeF.md)