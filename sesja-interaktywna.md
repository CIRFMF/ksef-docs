## Sesja interaktywna
26.06.2025

Sesja interaktywna służy do przesyłania pojedynczych faktur ustrukturyzowanych do API KSeF. Każda faktura musi być przygotowana w formacie XML zgodnie z aktualnym schematem opublikowanym przez Ministerstwo Finansów.

### Wymagania wstępne

Aby skorzystać z wysyłki interaktywnej, należy najpierw przejść proces [uwierzytelnienia](uwierzytelnianie.md) i posiadać aktualny token dostępowy (```accessToken```), który uprawnia do korzystania z chronionych zasobów API KSeF.

Przed otwarciem sesji oraz wysłaniem faktur wymagane jest:
* wygenerowanie klucza symetrycznego o długości 256 bitów i wektora inicjującego o długości 128 bitów (IV), dołączanego jako prefiks do szyfrogramu,
* zaszyfrowanie dokumentu algorytmem AES-256-CBC z dopełnianiem PKCS#7,
* zaszyfrowanie klucza symetrycznego algorytmem RSAES-OAEP (padding OAEP z funkcją MGF1 opartą na SHA-256 oraz skrótem SHA-256), przy użyciu klucza publicznego KSeF Ministerstwa Finansów.

Operacje te można zrealizować za pomocą komponentu ```CryptographyService```, dostępnego w kliencie KSeF.

Przykład w języku C#:
```csharp
EncryptionData encryptionData = cryptographyService.GetEncryptionData();
```
Przykład w języku Java:
```java
var cryptographyService = new DefaultCryptographyService(ksefClient);
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

### 1. Otwarcie sesji

Inicjalizacja nowej sesji interaktywnej z podaniem:
* wersji schematu faktury (FA(2) lub FA(3)) <br>
określa, którą wersję XSD system będzie stosować do walidacji przesyłanych faktur.
* zaszyfrowanego klucza symetrycznego<br>
symetryczny klucz szyfrujący pliki XML, zaszyfrowany kluczem publicznym Ministerstwa Finansów; rekomendowane jest użycie nowo wygenerowanego klucza dla każdej sesji.

Otwarcie sesji jest operacją lekką i synchroniczną – można równocześnie utrzymywać wiele otwartych sesji interaktywnych w ramach jednego uwierzytelnienia.

POST [sessions/online](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/operation/onlineSession.open)

W odpowiedzi zwracany jest obiekt zawierający pole ```referenceNumber``` – unikalny identyfikator sesji interaktywnej, który należy przekazywać we wszystkich kolejnych wywołaniach API.

Przykład w języku C#:
```csharp
var request = OpenOnlineSessionRequestBuilder
         .Create()
         .WithFormCode(systemCode: "FA (2)", schemaVersion: "1-0E", value: "FA")
         .WithEncryption(
             encryptedSymmetricKey: encryptionData.EncryptionInfo.EncryptedSymmetricKey,
             initializationVector: encryptionData.EncryptionInfo.InitializationVector)
         .Build();
var sessionReferenceNumber = request.referenceNumber;
```

Przykład w języku Java:
```java
var request = new OpenOnlineSessionRequestBuilder()
        .withFormCode(new FormCode("FA (2)","FA","1-0E"))
        .withEncryptionInfo(encryptionData.encryptionInfo())
        .build();

var response = ksefClient.openOnlineSession(request);
```

### 2. Wysłanie faktury

Zaszyfrowaną fakturę należy wysłać na endpoint:

POST [sessions/online/{referenceNumber}/invoices/](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1invoices/post)

Odpowiedź zawiera ```referenceNumber``` dokumentu – używany do identyfikacji faktury w kolejnych operacjach (np. listy dokumentów).

Przykład w języku C#:
```csharp
var encryptedInvoice = cryptographyService.EncryptBytesWithAES256(invoice, encryptionData.CipherKey, encryptionData.CipherIv);

var invoiceMetadata = cryptographyService.GetMetaData(invoice);
var encryptedInvoiceMetadata = cryptographyService.GetMetaData(encryptedInvoice);
var sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA.Value, invoiceMetadata.FileSize)
    .WithEncryptedInvoiceHash(
       encryptedInvoiceMetadata.HashSHA.Value, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedInvoiceContent(Convert.ToBase64String(encryptedInvoice))
    .Build();
var sendInvoiceResponse = await ksefClient.SendOnlineSessionInvoiceAsync(sendOnlineInvoiceRequest, referenceNumber, accesToken, cancellationToken);
```

Przykład w języku Java:
```java
byte[] invoice = ...;
var encryptedInvoice = cryptographyService.encryptBytesWithAES256(invoice,
        encryptionData.cipherKey(),
        encryptionData.cipherIv());

var invoiceMetadata = cryptographyService.getMetaData(invoice);
var encryptedInvoiceMetadata = cryptographyService.getMetaData(encryptedInvoice);

var sendInvoiceRequest = new SendInvoiceRequestBuilder()
        .withInvoiceHash(invoiceMetadata.getHashSHA())
        .withInvoiceSize(invoiceMetadata.getFileSize())
        .withEncryptedInvoiceHash(encryptedInvoiceMetadata.getHashSHA())
        .withEncryptedInvoiceSize(encryptedInvoiceMetadata.getFileSize())
        .withEncryptedInvoiceContent(Base64.getEncoder().encodeToString(encryptedInvoice))
        .build();

SendInvoiceResponse sendInvoiceResponse = defaultKsefClient.onlineSessionSendInvoice(referenceNumber, sendInvoiceRequest);
```

### 3. Zamknięcie sesji
Po wysłaniu wszystkich faktur należy zamknąć sesję, co inicjuje asynchroniczne generowanie zbiorczego UPO.

POST [/sessions/online/\{referenceNumber\}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1close/post)

Zbiorcze UPO będzie dostępne po sprawdzeniu stanu sesji.

Przykład w języku C#:

```csharp
 var upo = await ksefClient.CloseOnlineSessionAsync(
            sessionReferenceNumber,
            cancellationToken
        );    
```

Przykład w języku Java:
```java
ksefClient.closeOnlineSession(referenceNumber);
```

Zobacz [Sprawdzenie stanu i pobranie UPO](wysyłka-faktur/sesja-sprawdzenie-stanu-i-pobranie-upo.md)