## Pobieranie faktur
### Pobranie faktur po numerze KSeF
21.08.2025

Zwraca fakturę o podanym numerze KSeF.

GET [/invoices/ksef/\{ksefReferenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1ksef~1%7BksefNumber%7D/get)

Przykład w języku C#:

```csharp
string invoice = await ksefClient.GetInvoiceAsync(ksefReferenceNumber, accessToken, cancellationToken);
```

Przykład w języku Java:

[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] invoice = createKSeFClient().getInvoice(ksefNumber, accessToken);
```

### Pobranie listy metadanych faktur
Zwraca listę metadanych faktur spełniające podane kryteria wyszukiwania.

POST [/invoices/query/metadata](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/docs/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
InvoiceQueryFilters invoiceQueryFilters = new InvoiceQueryFilters
{
    SubjectType = SubjectType.Subject1,
    DateRange = new DateRange
    {
        From = DateTime.UtcNow.AddDays(-30),
        To = DateTime.UtcNow,
        DateType = DateType.Issue
    }
};

PagedInvoiceResponse pagedInvoicesResponse = await ksefClient.QueryInvoiceMetadataAsync(
    invoiceQueryFilters, 
    accessToken, 
    pageOffset: 0, 
    pageSize: 10, 
    cancellationToken);
```

Przykład w języku Java:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoiceMetadataQueryRequest request = new InvoiceMetadataQueryRequestBuilder()
        .withSubjectType(InvoiceQuerySubjectType.SUBJECT1)
        .withDateRange(new InvoiceQueryDateRange(InvoiceQueryDateType.ISSUE,
        OffsetDateTime.now().minusDays(10), OffsetDateTime.now().plusDays(10)))
        .build();

QueryInvoiceMetadataResponse response = createKSeFClient().queryInvoiceMetadata(0, 10, request, accessToken);

```

### Inicjalizuje asynchroniczne zapytanie o pobranie faktur

Rozpoczyna asynchroniczny proces wyszukiwania faktur w systemie KSeF na podstawie przekazanych filtrów. Wymagane jest przekazanie informacji o szyfrowaniu w polu `encryption`, które służą wykorzystywane do zaszyfrowania wygenerowanych paczek z fakturami.

POST [/invoices/exports](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post)

Przykład w języku C#:
```csharp
InvoiceQueryFilters query = new InvoiceQueryFilters
{
    DateRange = new DateRange
    {
        From = DateTime.Now.AddDays(-1),
        To = DateTime.Now.AddDays(1),
        DateType = DateType.Issue
    },
    SubjectType = SubjectType.Subject1
};

ExportInvoicesResponse exportInvoicesResponse = await KsefClient.ExportInvoicesAsync(
    query,
    accessToken);
```
Dostępne wartości `DateType` oraz `SubjectType` są opisane [tutaj](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post) 

Faktury w paczce są sortowane rosnąco według typu daty wskazanego w `DateRange` podczas inicjalizacji eksportu.

Przykład w języku Java:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoicesAsyncQueryRequest request = new InvoicesAsyncQueryRequestBuilder()
        .withEncryption(new EncryptionInfo(encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
        encryptionData.encryptionInfo().getInitializationVector()))
        .withSubjectType(InvoiceQuerySubjectType.SUBJECT1)
        .withDateRange(new InvoiceQueryDateRange(InvoiceQueryDateType.ISSUE,
        OffsetDateTime.now().minusDays(10), OffsetDateTime.now().plusDays(10)))
        .build();

InitAsyncInvoicesQueryResponse response = createKSeFClient().initAsyncQueryInvoice(request, accessToken);

```

### Sprawdza status asynchronicznego zapytania o pobranie faktur

Pobiera status wcześniej zainicjalizowanego zapytania asynchronicznego na podstawie identyfikatora operacji. Umożliwia śledzenie postępu przetwarzania zapytania oraz pobranie gotowych paczek z wynikami, jeśli są już dostępne.

GET [/invoices/exports/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports~1%7BoperationReferenceNumber%7D/get)

Przykład w języku C#:
```csharp
InvoiceExportStatusResponse exportStatus = await KsefClient.GetInvoiceExportStatusAsync(
    exportInvoicesResponse.OperationReferenceNumber,
    accessToken);
```
Przykład w języku Java:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
AsyncInvoicesQueryStatus response = createKSeFClient().checkStatusAsyncQueryInvoice(operationReferenceNumber, accessToken);

```
