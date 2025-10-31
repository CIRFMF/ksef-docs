## Pobieranie faktur
### Pobranie faktur po numerze KSeF
21.08.2025

Zwraca fakturę o podanym numerze KSeF.

GET [/invoices/ksef/\{ksefReferenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1ksef~1%7BksefNumber%7D/get)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
string invoice = await ksefClient.GetInvoiceAsync(ksefReferenceNumber, accessToken, cancellationToken);
```

Przykład w języku Java:

[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
byte[] invoice = ksefClient.getInvoice(ksefNumber, accessToken);
```

### Pobranie listy metadanych faktur
Zwraca listę metadanych faktur spełniające podane kryteria wyszukiwania.

**Plik _metadata.json w paczce eksportu**  
W paczce eksportu znajduje się plik `_metadata.json` zawierający tablicę obiektów `InvoiceMetadata` (model zwracany przez POST `/invoices/query/metadata` - "Pobieranie metadanych faktur").

POST [/invoices/query/metadata](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

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
InvoiceQueryFilters request = new InvoiceQueryFiltersBuilder()
        .withSubjectType(InvoiceQuerySubjectType.SUBJECT1)
        .withDateRange(
                new InvoiceQueryDateRange(InvoiceQueryDateType.INVOICING, OffsetDateTime.now().minusYears(1),
                        OffsetDateTime.now()))
        .build();

QueryInvoiceMetadataResponse response = ksefClient.queryInvoiceMetadata(0, 10, request, accessToken);


```

### Inicjalizuje asynchroniczne zapytanie o pobranie faktur

Rozpoczyna asynchroniczny proces wyszukiwania faktur w systemie KSeF na podstawie przekazanych filtrów. Wymagane jest przekazanie informacji o szyfrowaniu w polu `encryption`, które służą wykorzystywane do zaszyfrowania wygenerowanych paczek z fakturami.

POST [/invoices/exports](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
EncryptionData encryptionData = CryptographyService.GetEncryptionData();

InvoiceQueryFilters query = new InvoiceQueryFilters
{
    DateRange = new DateRange
    {
        From = DateTime.Now.AddDays(-1),
        To = DateTime.Now.AddDays(1),
        DateType = DateType.Invoicing
    },
    SubjectType = SubjectType.Subject1
};

InvoiceExportRequest invoiceExportRequest = new InvoiceExportRequest
{
    Encryption = encryptionData.EncryptionInfo,
    Filters = query
};

OperationResponse invoicesForSellerResponse = await KsefClient.ExportInvoicesAsync(
    invoiceExportRequest,
    _accessToken,
    CancellationToken);
```

Dostępne wartości `DateType` oraz `SubjectType` są opisane [tutaj](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post).

Faktury w paczce są sortowane rosnąco według typu daty wskazanego w `DateRange` podczas inicjalizacji eksportu.

Przykład w języku Java:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoiceExportFilters filters = new InvoicesAsyncQueryFiltersBuilder()
        .withSubjectType(InvoiceQuerySubjectType.SUBJECT1)
        .withDateRange(
                new InvoiceQueryDateRange(InvoiceQueryDateType.INVOICING, OffsetDateTime.now().minusDays(10), OffsetDateTime.now().plusDays(10)))
        .build();

InvoiceExportRequest request = new InvoiceExportRequest(
        new EncryptionInfo(encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
                encryptionData.encryptionInfo().getInitializationVector()), filters);

InitAsyncInvoicesQueryResponse response = ksefClient.initAsyncQueryInvoice(request, accessToken);

```

### Sprawdza status asynchronicznego zapytania o pobranie faktur

Pobiera status wcześniej zainicjalizowanego zapytania asynchronicznego na podstawie identyfikatora operacji. Umożliwia śledzenie postępu przetwarzania zapytania oraz pobranie gotowych paczek z wynikami, jeśli są już dostępne.

GET [/invoices/exports/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports~1%7BoperationReferenceNumber%7D/get)

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Invoice\InvoiceE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/InvoiceE2ETests.cs)

```csharp
InvoiceExportStatusResponse exportStatus = await KsefClient.GetInvoiceExportStatusAsync(
    exportInvoicesResponse.OperationReferenceNumber,
    accessToken);
```

Przykład w języku Java:

[QueryInvoiceIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QueryInvoiceIntegrationTest.java)

```java
InvoiceExportStatus response = ksefClient.checkStatusAsyncQueryInvoice(operationReferenceNumber, accessToken);

```

## Powiązane dokumenty

- [Przyrostowe pobieranie faktur](przyrostowe-pobieranie-faktur.md)