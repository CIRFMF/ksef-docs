## Pobieranie faktur
### Pobranie faktur po numerze KSeF
21.08.2025

Zwraca fakturę o podanym numerze KSeF.

GET [/invoices/ksef/\{ksefReferenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1ksef~1%7BksefNumber%7D/get)

Przykład w języku C#:

```csharp
var invoice = await ksefClient.GetInvoiceAsync(ksefReferenceNumber, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
var document = ksefClient.getInvoice(ksefReferenceNumber);
```

### Pobranie listy metadanych faktur
Zwraca listę metadanych faktur spełniające podane kryteria wyszukiwania.

POST [/invoices/query/metadata](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post)

Przykład w języku C#:
```csharp
var pagedInvoicesResponse = await ksefClient.QueryInvoiceMetadataAsync(body, accessToken, pageOffset, pageSize, cancellationToken);
```

Przykład w języku Java:
```java
var queryInvoicesReponse = ksefClient.queryInvoices(pageOffset, pageSize, invoicesQueryRequest);
```

### Inicjalizuje asynchroniczne zapytanie o pobranie faktur

Rozpoczyna asynchroniczny proces wyszukiwania faktur w systemie KSeF na podstawie przekazanych filtrów. Wymagane jest przekazanie informacji o szyfrowaniu w polu Encryption, które służą do zaszyfrowania wygenerowanych paczek z fakturami.

POST [/invoices/async-query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1async-query/post)

Przykład w języku C#:
```csharp
var operstonStatus =await ksefClient.AsyncQueryInvoicesAsync(body, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
var operationStatus = ksefClient.initAsyncQueryInvoice(invoicesAsynqQueryRequest);
```

### Sprawdza status asynchronicznego zapytania o pobranie faktur

Pobiera status wcześniej zainicjalizowanego zapytania asynchronicznego na podstawie identyfikatora operacji. Umożliwia śledzenie postępu przetwarzania zapytania oraz pobranie gotowych paczek z wynikami, jeśli są już dostępne.

GET [/invoices/async-query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1async-query~1%7BoperationReferenceNumber%7D/get)

Przykład w języku C#:
```csharp
var invoicesQueryStatus = await ksefClient.GetAsyncQueryInvoicesStatusAsync(operationReferenceNumber, accessToken, cancellationToken);
```
Przykład w języku Java:
```java
var invoicesQueryStatus = ksefClient.checkStatusAsyncQueryInvoice(operationReferenceNumber);
```