## Sesja – sprawdzenie stanu i pobranie UPO
26.06.2025

Niniejszy rozdział opisuje operacje służące do monitorowania stanu sesji (interaktywnej lub wsadowej) oraz pobierania UPO dla faktur i całej sesji.

### 1. Sprawdzenie stanu sesji
Sprawdza bieżący stan sesji.

GET [sessions/\{referenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get)

Zwraca bieżący status sesji wraz z zagregowanymi danymi o liczbie przesłanych, poprawnie i niepoprawnie przetworzonych faktur; po zamknięciu sesji udostępnia dodatkowo listę referencji do zbiorczego UPO.

Przykład w języku C#:
```csharp
var openSessionResult = await kSeFClient.GetSessionStatusAsync(referenceNumber, accessToken, cancellationToken);
var documentCount = openSessionResult.InvoiceCount;
var successfulInvoiceCount = openSessionResult.SuccessfulInvoiceCount;
var failedInvoiceCount = openSessionResult.FailedInvoiceCount;
```

Przykład w języku Java:
```java
var openSessionResult = ksefClient.getSessionStatus(referenceNumber);
var invoiceCount = openSessionResult.getInvoiceCount();
var successfulInvoiceCount = openSessionResult.getSuccessfulInvoiceCount();
var failedInvoiceCount = openSessionResult.getFailedInvoiceCount();
```


### 2. Pobranie informacji na temat przesłanych faktur

GET [sessions/\{referenceNumber\}/invoices](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices/get)

Zwraca listę metadanych wszystkich przesłanych faktur wraz z ich statusami oraz łączną liczbę tych faktur w sesji.

Przykład w języku C#:
```csharp
const int pageSize = 50;
int pageOffset = 0;
bool morePages;

do
{
    var sessionInvoices = await ksefClient
                                .GetSessionInvoicesAsync(
                                referenceNumber,
                                accessToken,
                                pageOffset,
                                pageSize,
                                cancellationToken);

    foreach (var doc in getInvoicesResult.Invoices)
    {
        Console.WriteLine($"#{doc.InvoiceNumber}. Status: {doc.Status.Code}");
    }

    morePages = listResult.Invoices.Count == pageSize;
    pageOffset++;
}
while (morePages);

```

Przykład w języku Java:
```java
var sessionInvoices = ksefClient.getSessionInvoices(referenceNumber, 10, 0);
sessionInvoices.getInvoices().forEach(invoice -> {
    log.info(invoice.getInvoiceNumber() + " " + invoice.getKsefNumber() + " " + invoice.getStatus());
});
```
### 3. Pobranie informacji o pojedynczej fakturze

Umożliwia pobranie szczegółowych informacji o pojedynczej fakturze w sesji, w tym jej statusu i metadanych.

Należey podać numer referencyjny sesji `referenceNumber` oraz numer referencyjny faktury `invoiceReferenceNumber`.

GET [sessions/\{referenceNumber\}/invoices/\{invoiceReferenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1%7BinvoiceReferenceNumber%7D/get)

Przyklad w języku C#:
```csharp
var invoice = await ksefClient
                .GetSessionInvoiceAsync(
                referenceNumber,
                invoiceReferenceNumber,
                accessToken,
                cancellationToken);
```

Przykład w języku Java:
```java
SessionInvoice sessionInvoiceStatus = ksefClient.getSessionInvoiceStatus(referenceNumber, invoiceReferenceNumber);
```

### 4. Pobranie UPO dla faktury

Umożliwia pobranie UPO dla pojedynczej, poprawnie przyjętej faktury.

#### 4.1 Na podstawie numery referencyjnego faktury

GET [sessions/\{referenceNumber\}/invoices/\{invoiceReferenceNumber\}/upo](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1%7BinvoiceReferenceNumber%7D~1upo/get)

Przykład w języku C#:
```csharp
var upo = await ksefClient
                .GetSessionInvoiceUpoByReferenceNumberAsync(
                referenceNumber,
                invoiceReferenceNumber,
                accessToken,
                cancellationToken)
```

Przykład w języku Java:
```java
var upo = ksefClient.getSessionInvoiceUpoByReferenceNumber(referenceNumber, invoiceReferenceNumber);
```

#### 4.2 Na podstawie numeru KSeF faktury

GET [sessions/\{referenceNumber\}/invoices/\{ksefNumber\}/upo](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1ksef~1%7BksefNumber%7D~1upo/get)

Przykład w języku C#:
```csharp
var upo = await ksefClient
                .GetSessionInvoiceUpoByKsefNumberAsync(
                referenceNumber,
                ksefNumber,
                accessToken,
                cancellationToken)
```

Przykład w języku Java:
```java
var upo = ksefClient.getSessionInvoiceUpoByKsefNumber(referenceNumber, ksefNumber);
```

Otrzymany dokument XML jest: 
* podpisany w formacie XADES przez Ministerstwo Finansów
* zgodny ze schematem [XSD](/wysyłka-faktur/upo-faktura.xsd) dla pojedynczej faktury.

### 5. Pobranie listy niepoprawnie przyjętych faktur

GET [sessions/\{referenceNumber\}/invoices/failed](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1failed/get)

Zwraca łączną liczbę odrzuconych faktur w sesji oraz szczegółowe informacje (status i szczegóły błędów) dla każdej niepoprawnie przetworzonej faktury.

Przykład w języku C#:
```csharp
const int pageSize = 50;
string continuationToken = "";

do
{
    var sessionInvoices = await ksefClient
                                .GetSessionFailedInvoicesAsync(
                                referenceNumber,
                                accessToken,
                                pageSize,
                                continuationToken,
                                cancellationToken);

    continuationToken = failedResult.Invoices.ContinuationToken

}
while (!string.IsNullOrEmpty(continuationToken));
```

Przykład w języku Java:
```java
SessionInvoicesResponse sessionFailedInvoices = ksefClient.getSessionFailedInvoices(referenceNumber, pageSize);
sessionFailedInvoices.getInvoices().forEach(invoice->{
    log.info(invoice.getInvoiceNumber() + " " + invoice.getStatus());
});
```

Endpoint umożliwia selektywne pobranie wyłącznie odrzuconych faktur, co ułatwia analizę błędów w sesjach zawierających dużą liczbę faktur.

### 6. Pobranie UPO sesji

Udostępnia zbiorcze UPO potwierdzające przyjęcie wszystkich faktur przesłanych w danej sesji.

GET [/sessions/\{referenceNumber\}/upo/\{upoReferenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1upo~1%7BupoReferenceNumber%7D/get)

Otrzymany dokument XML jest zgodny ze schematem [XSD](/wysyłka-faktur/upo-sesja.xsd).

Przyklad w języku C#:

```csharp
 var upo = await ksefClient.GetSessionUpoAsync(
            sessionReferenceNumber,
            upoReferenceNumber,
            accesToken,
            cancellationToken
        );
```

Przykład w języku Java:
```java
var upo = ksefClient.getSessionUpo(referenceNumber, upoReferenceNumber);
```