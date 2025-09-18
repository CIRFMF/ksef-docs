## Sesja – sprawdzenie stanu i pobranie UPO
10.07.2025

Niniejszy dokument opisuje operacje służące do monitorowania stanu sesji (interaktywnej lub wsadowej) oraz pobierania UPO dla faktur i całej sesji.

### 1. Pobranie listy sesji
Zwraca listę sesji spełniających podane kryteria wyszukiwania.

GET [sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions/get)

Zwraca bieżący status sesji wraz z zagregowanymi danymi o liczbie przesłanych, poprawnie i niepoprawnie przetworzonych faktur; po zamknięciu sesji udostępnia dodatkowo listę referencji do zbiorczego UPO.

Przykład w języku C#:
```csharp
// Pobieranie sesji wsadowych
 var sessions = new List<Session>();
 const int pageSize = 20;
 string? continuationToken = null;
 do
 {
     var response = await ksefClient.GetSessionsAsync(SessionType.Batch, accessToken, pageSize, continuationToken, sessionsFilter, cancellationToken);
     continuationToken = response.ContinuationToken;
     sessions.AddRange(response.Sessions);
 } while (!string.IsNullOrEmpty(continuationToken));

// Pobieranie sesji interaktywnych
 var sessions = new List<Session>();
 const int pageSize = 20;
 string? continuationToken = null;
 do
 {
     var response = await ksefClient.GetSessionsAsync(SessionType.Online, accessToken, pageSize, continuationToken, sessionsFilter, cancellationToken);
     continuationToken = response.ContinuationToken;
     sessions.AddRange(response.Sessions);
 } while (!string.IsNullOrEmpty(continuationToken)); 
```

Przykład w języku Java:
```java
int pageSize = 10;
String continuationToken = null;
SessionsQueryRequest request = new SessionsQueryRequest();
request.setSessionType(SessionType.ONLINE); // sesja interaktywna
//request.setSessionType(SessionType.BATCH); // sesja batchowa
SessionsQueryResponse sessionsQueryResponse = defaultKsefClient.getSessions(request, pageSize, continuationToken);
```

### 2. Sprawdzenie stanu sesji
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


### 3. Pobranie informacji na temat przesłanych faktur

GET [sessions/\{referenceNumber\}/invoices](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices/get)

Zwraca listę metadanych wszystkich przesłanych faktur wraz z ich statusami oraz łączną liczbę tych faktur w sesji.

Przykład w języku C#:
```csharp
const int pageSize = 50;
string continuationtoken = null;

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

    continuationtoken = sessionInvoices.ContinuationToken;
}
while (continuationtoken != null);

```

Przykład w języku Java:
```java
var sessionInvoices = ksefClient.getSessionInvoices(referenceNumber, 10, 0);
sessionInvoices.getInvoices().forEach(invoice -> {
    log.info(invoice.getInvoiceNumber() + " " + invoice.getKsefNumber() + " " + invoice.getStatus());
});
```
### 4. Pobranie informcji o pojedynczej fakturze

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

### 5. Pobranie UPO dla faktury

Umożliwia pobranie UPO dla pojedynczej, poprawnie przyjętej faktury.

#### 5.1 Na podstawie numery referencyjnego faktury

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

#### 5.2 Na podstawie numeru KSeF faktury

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
* zgodny ze schematem [XSD](/faktury/upo/schemy/upo-faktura-v3-1.xsd) dla pojedynczej faktury.

### 6. Pobranie listy niepoprawnie przyjętych faktur

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

### 7. Pobranie UPO sesji

UPO sesji stanowi zbiorcze poświadczenie przyjęcia wszystkich faktur poprawnie przesłanych w ramach danej sesji.

Po zamknięciu sesji, w odpowiedzi na sprawdzenie jej [stanu](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get) (krok 2 – Sprawdzenie stanu sesji), zwracane są nie tylko informacje o liczbie poprawnie i błędnie przetworzonych faktur, lecz także lista referencji do zbiorczych UPO.

Każdy element tablicy `upo.pages[]` zawiera numer referencyjny UPO (`referenceNumber`) oraz link (`downloadUrl`) umożliwiający jego pobranie:

```json
"upo": {
    "pages": [
        {
            "referenceNumber": "20250901-EU-47FDBE3000-5961A5D232-BF",
            "downloadUrl": "/api/v2/sessions/20250901-SB-47FA636000-5960B49115-9D/upo/20250901-EU-47FDBE3000-5961A5D232-BF"
        },
        {
            "referenceNumber": "20250901-EU-48D8488000-59667BB54C-C8",
            "downloadUrl": "/api/v2/sessions/20250901-SB-47FA636000-5960B49115-9D/upo/20250901-EU-48D8488000-59667BB54C-C8"
        }        
    ]
}

```

Dysponując tą listą, klient API może pobrać UPO pojedynczo, wywołując endpoint wskazany w polu `downloadUrl`, tj.  
GET [/sessions/\{referenceNumber\}/upo/\{upoReferenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1upo~1%7BupoReferenceNumber%7D/get)

Otrzymany dokument XML jest zgodny ze schematem [XSD](/faktury/upo/schemy/upo-sesja-v4-2.xsd) i może zawierać maksymalnie 10 000 pozycji faktur.

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

## Powiązane dokumenty
- [Numer KSeF – struktura i walidacja](numer-ksef.md)