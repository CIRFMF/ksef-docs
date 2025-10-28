## Sesja interaktywna
10.07.2025

Sesja interaktywna służy do przesyłania pojedynczych faktur ustrukturyzowanych do API KSeF. Każda faktura musi być przygotowana w formacie XML zgodnie z aktualnym schematem opublikowanym przez Ministerstwo Finansów.

### Wymagania wstępne

Aby skorzystać z wysyłki interaktywnej, należy najpierw przejść proces [uwierzytelnienia](uwierzytelnianie.md) i posiadać aktualny token dostępowy (```accessToken```), który uprawnia do korzystania z chronionych zasobów API KSeF.

Przed otwarciem sesji oraz wysłaniem faktur wymagane jest:
* wygenerowanie klucza symetrycznego o długości 256 bitów i wektora inicjującego o długości 128 bitów (IV), dołączanego jako prefiks do szyfrogramu,
* zaszyfrowanie dokumentu algorytmem AES-256-CBC z dopełnianiem PKCS#7,
* zaszyfrowanie klucza symetrycznego algorytmem RSAES-OAEP (padding OAEP z funkcją MGF1 opartą na SHA-256 oraz skrótem SHA-256), przy użyciu klucza publicznego KSeF Ministerstwa Finansów.

Operacje te można zrealizować za pomocą komponentu ```CryptographyService```, dostępnego w kliencie KSeF.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
EncryptionData encryptionData = CryptographyService.GetEncryptionData();
```
Przykład w języku Java:
[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

### 1. Otwarcie sesji

Inicjalizacja nowej sesji interaktywnej z podaniem:
* wersji schematu faktury: [FA(2)](faktury/schemat-FA(2)-v1-0E.xsd), [FA(3)](faktury/schemat-FA(3)-v1-0E.xsd) <br>
określa, którą wersję XSD system będzie stosować do walidacji przesyłanych faktur.
* zaszyfrowanego klucza symetrycznego<br>
symetryczny klucz szyfrujący pliki XML, zaszyfrowany kluczem publicznym Ministerstwa Finansów; rekomendowane jest użycie nowo wygenerowanego klucza dla każdej sesji.

Otwarcie sesji jest operacją lekką i synchroniczną – można równocześnie utrzymywać wiele otwartych sesji interaktywnych w ramach jednego uwierzytelnienia.

POST [sessions/online](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/operation/onlineSession.open)

W odpowiedzi zwracany jest obiekt zawierający: 
 - ```referenceNumber``` – unikalny identyfikator sesji interaktywnej, który należy przekazywać we wszystkich kolejnych wywołaniach API.
 - ```validUntil``` – Termin ważności sesji. Po jego upływie sesja zostanie automatycznie zamknięta. Czas życia sesji interaktywnej wynosi 12 godzin od momentu jej utworzenia.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
OpenOnlineSessionRequest openOnlineSessionRequest = OpenOnlineSessionRequestBuilder
    .Create()
    .WithFormCode(systemCode: SystemCodeHelper.GetValue(systemCode), schemaVersion: DefaultSchemaVersion, value: DefaultFormCodeValue)
    .WithEncryption(
        encryptedSymmetricKey: encryptionData.EncryptionInfo.EncryptedSymmetricKey,
        initializationVector: encryptionData.EncryptionInfo.InitializationVector)
    .Build();

OpenOnlineSessionResponse openOnlineSessionResponse = await KsefClient.OpenOnlineSessionAsync(openOnlineSessionRequest, accessToken, CancellationToken);
```

Przykład w języku Java:
[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
OpenOnlineSessionRequest request = new OpenOnlineSessionRequestBuilder()
        .withFormCode(new FormCode(systemCode, schemaVersion, value))
        .withEncryptionInfo(encryptionData.encryptionInfo())
        .build();

OpenOnlineSessionResponse openOnlineSessionResponse = ksefClient.openOnlineSession(request, accessToken);
```

### 2. Wysłanie faktury

Zaszyfrowaną fakturę należy wysłać na endpoint:

POST [sessions/online/{referenceNumber}/invoices/](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1invoices/post)

Odpowiedź zawiera ```referenceNumber``` dokumentu – używany do identyfikacji faktury w kolejnych operacjach (np. listy dokumentów).

Po prawidłowym przesłaniu faktury rozpoczyna się asynchroniczna weryfikacja faktury ([szczegóły weryfikacji](faktury/weryfikacja-faktury.md)).

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
byte[] encryptedInvoice = cryptographyService.EncryptBytesWithAES256(invoice, encryptionData.CipherKey, encryptionData.CipherIv);
FileMetadata invoiceMetadata = cryptographyService.GetMetaData(invoice);
FileMetadata encryptedInvoiceMetadata = cryptographyService.GetMetaData(encryptedInvoice);

SendInvoiceRequest sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA, invoiceMetadata.FileSize)
    .WithEncryptedDocumentHash(encryptedInvoiceMetadata.HashSHA, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedDocumentContent(Convert.ToBase64String(encryptedInvoice))
    .Build();

SendInvoiceResponse sendInvoiceResponse = await KsefClient.SendOnlineSessionInvoiceAsync(sendOnlineInvoiceRequest, referenceNumber, accessToken);
```

Przykład w języku Java:
[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
String invoiceTemplate = ""; // xml z fakturą

byte[] invoice = invoiceTemplate.getBytes(StandardCharsets.UTF_8);

byte[] encryptedInvoice = cryptographyService.encryptBytesWithAES256(invoice,
    encryptionData.cipherKey(),
    encryptionData.cipherIv());

FileMetadata invoiceMetadata = cryptographyService.getMetaData(invoice);
FileMetadata encryptedInvoiceMetadata = cryptographyService.getMetaData(encryptedInvoice);

SendInvoiceOnlineSessionRequest sendInvoiceOnlineSessionRequest = new SendInvoiceOnlineSessionRequestBuilder()
    .withInvoiceHash(invoiceMetadata.getHashSHA())
    .withInvoiceSize(invoiceMetadata.getFileSize())
    .withEncryptedInvoiceHash(encryptedInvoiceMetadata.getHashSHA())
    .withEncryptedInvoiceSize(encryptedInvoiceMetadata.getFileSize())
    .withEncryptedInvoiceContent(Base64.getEncoder().encodeToString(encryptedInvoice))
    .build();

SendInvoiceResponse sendInvoiceResponse = ksefClient.onlineSessionSendInvoice(sessionReferenceNumber, sendInvoiceOnlineSessionRequest, accessToken);
```

### 3. Zamknięcie sesji
Po wysłaniu wszystkich faktur należy zamknąć sesję, co inicjuje asynchroniczne generowanie zbiorczego UPO.

POST [/sessions/online/\{referenceNumber\}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1close/post)

Zbiorcze UPO będzie dostępne po sprawdzeniu stanu sesji.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\OnlineSession\OnlineSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/OnlineSession/OnlineSessionE2ETests.cs)

```csharp
await KsefClient.CloseOnlineSessionAsync(referenceNumber, accessToken, CancellationToken);
```

Przykład w języku Java:
[OnlineSessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/OnlineSessionIntegrationTest.java)

```java
ksefClient.closeOnlineSession(sessionReferenceNumber, accessToken);
```

Powiązane dokumenty: 
- [Sprawdzenie stanu i pobranie UPO](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md)
- [Weryfikacja faktury](faktury/weryfikacja-faktury.md)
- [Numer KSeF – struktura i walidacja](faktury/numer-ksef.md)