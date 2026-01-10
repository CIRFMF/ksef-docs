## Sesja wsadowa
10.07.2025

Wysyłka wsadowa umożliwia jednorazowe przesłanie wielu faktur w pojedynczym pliku ZIP, zamiast wysyłać każdą fakturę oddzielnie.

To rozwiązanie przyspiesza i ułatwia przetwarzanie dużej liczby dokumentów — szczególnie dla firm, które generują wiele faktur dziennie.

Każda faktura musi być przygotowana w formacie XML zgodnie z aktualnym schematem opublikowanym przez Ministerstwo Finansów:
* Paczka ZIP powinna być podzielona na części nie większe niż 100 MB (przed zaszyfrowaniem), które są szyfrowane i przesyłane osobno.
* Należy podać informacje o każdej części paczki w obiekcie ```fileParts```.


### Wymagania wstępne
Aby skorzystać z wysyłki wsadowej, należy najpierw przejść proces [uwierzytelnienia](uwierzytelnianie.md) i posiadać aktualny token dostępu (```accessToken```), który uprawnia do korzystania z chronionych zasobów API KSeF.

**Zalecenie (korelacja statusów po `invoiceHash`)**  
Przed utworzeniem paczki do wysyłki wsadowej zaleca się obliczyć skrót SHA-256 dla każdego pliku XML faktury (oryginał, przed szyfrowaniem) oraz zapisać lokalne mapowanie. Umożliwia to jednoznaczne powiązanie statusów przetwarzania po stronie KSeF z lokalnymi dokumentami źródłowymi (XML) przygotowanymi do wysyłki.

Przed otwarciem sesji oraz wysłaniem faktur wymagane jest:
* wygenerowanie klucza symetrycznego o długości 256 bitów i wektora inicjującego o długości 128 bitów (IV), dołączanego jako prefiks do szyfrogramu,
* przygotowanie paczki ZIP,
* (opcjonalnie, jeśli paczka przekracza dopuszczalny rozmiar) podział paczki ZIP na części,
* zaszyfrowanie części algorytmem AES-256-CBC z dopełnianiem PKCS#7,
* zaszyfrowanie klucza symetrycznego algorytmem RSAES-OAEP (padding OAEP z funkcją MGF1 opartą na SHA-256 oraz skrótem SHA-256), przy użyciu klucza publicznego KSeF Ministerstwa Finansów.

Operacje te można zrealizować za pomocą komponentu ```CryptographyService```, dostępnego w oficjalnym kliencie KSeF. Biblioteka ta udostępnia gotowe metody do generowania i szyfrowania kluczy, zgodnie z wymaganiami systemu.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
EncryptionData encryptionData = cryptographyService.GetEncryptionData();
```
Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

Wygenerowane dane służą do szyfrowania faktur.

### 1. Przygotowanie paczki ZIP
Należy utworzyć paczkę ZIP zawierającą wszystkie faktury, które zostaną przesłane w ramach jednej sesji.  

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
(byte[] zipBytes, Client.Core.Models.Sessions.FileMetadata zipMeta) =
    BatchUtils.BuildZip(invoices, cryptographyService);

//BatchUtils.BuildZip
public static (byte[] ZipBytes, FileMetadata Meta) BuildZip(
    IEnumerable<(string FileName, byte[] Content)> files,
    ICryptographyService crypto)
{
    using MemoryStream zipStream = new MemoryStream();
    using ZipArchive archive = new ZipArchive(zipStream, ZipArchiveMode.Create, leaveOpen: true);
    
    foreach ((string fileName, byte[] content) in files)
    {
        ZipArchiveEntry entry = archive.CreateEntry(fileName, CompressionLevel.Optimal);
        using Stream entryStream = entry.Open();
        entryStream.Write(content);
    }
    
    archive.Dispose();
    
    byte[] zipBytes = zipStream.ToArray();
    List<byte[]> meta = crypto.GetMetaData(zipBytes);

    return (zipBytes, meta);
}
```

Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
byte[] zipBytes = FilesUtil.createZip(invoicesInMemory);

// get ZIP metadata (before crypto)
FileMetadata zipMetadata = defaultCryptographyService.getMetaData(zipBytes);
```

### 2. Podział binarny paczki ZIP na części

Ze względu na ograniczenia rozmiaru przesyłanych plików, paczka ZIP powinna być podzielona binarnie na mniejsze części, które będą przesyłane osobno. Każda część powinna mieć unikalną nazwę i numer porządkowy.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp

 // Pobierz metadane ZIP-a (przed szyfrowaniem)
FileMetadata zipMetadata = cryptographyService.GetMetaData(zipBytes);
int maxPartSize = 100 * 1000 * 1000; // 100 MB
int partCount = (int)Math.Ceiling((double)zipBytes.Length / maxPartSize);
int partSize = (int)Math.Ceiling((double)zipBytes.Length / partCount);
List<byte[]> zipParts = new List<byte[]>();
for (int i = 0; i < partCount; i++)
{
    int start = i * partSize;
    int size = Math.Min(partSize, zipBytes.Length - start);
    if (size <= 0) break;
    byte[] part = new byte[size];
    Array.Copy(zipBytes, start, part, 0, size);
    zipParts.Add(part);
}

```

Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
List<byte[]> zipParts = FilesUtil.splitZip(partsCount, zipBytes);
```

### 3. Zaszyfrowanie części paczki
Każdą część należy zaszyfrować nowo wygenerowanym kluczem AES‑256‑CBC z dopełnianiem PKCS#7.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionStreamE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionStreamE2ETests.cs)
```csharp
List<BatchPartStreamSendingInfo> encryptedParts = new(rawParts.Count);
for (int i = 0; i < rawParts.Count; i++)
{
    using MemoryStream partInput = new(rawParts[i], writable: false);
    MemoryStream encryptedOutput = new();
    await cryptographyService.EncryptStreamWithAES256Async(partInput, encryptedOutput, encryptionData.CipherKey, encryptionData.CipherIv, CancellationToken).ConfigureAwait(false);

    if (encryptedOutput.CanSeek)
    {
        encryptedOutput.Position = 0;
    }

    FileMetadata partMeta = await cryptographyService.GetMetaDataAsync(encryptedOutput, CancellationToken).ConfigureAwait(false);
    if (encryptedOutput.CanSeek)
    {
        encryptedOutput.Position = 0; // reset po odczycie do metadanych
    }

    encryptedParts.Add(new BatchPartStreamSendingInfo
    {
        DataStream = encryptedOutput,
        OrdinalNumber = i + 1,
        Metadata = partMeta
    });
}
```

Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
 List<BatchPartSendingInfo> encryptedZipParts = new ArrayList<>();
 for (int i = 0; i < zipParts.size(); i++) {
     byte[] encryptedZipPart = defaultCryptographyService.encryptBytesWithAES256(
             zipParts.get(i),
             cipherKey,
             cipherIv
     );
     FileMetadata zipPartMetadata = defaultCryptographyService.getMetaData(encryptedZipPart);
     encryptedZipParts.add(new BatchPartSendingInfo(encryptedZipPart, zipPartMetadata, (i + 1)));
 }

```

### 4. Otwarcie sesji wsadowej

Inicjalizacja nowej sesji wsadowej z podaniem:
* wersji schematu faktury: [FA(2)](faktury/schemy/FA/schemat_FA(2)_v1-0E.xsd), [FA(3)](faktury/schemy/FA/schemat_FA(3)_v1-0E.xsd) <br>
określa, którą wersję XSD system będzie stosować do walidacji przesyłanych faktur.
* zaszyfrowanego klucza symetrycznego<br>
symetryczny klucz szyfrujący pliki XML, zaszyfrowany kluczem publicznym Ministerstwa Finansów; rekomendowane jest użycie nowo wygenerowanego klucza dla każdej sesji.
* metadane paczki ZIP i jej części: nazwa pliku, hash, rozmiar oraz lista części (jeśli paczka jest dzielona)

POST [/sessions/batch](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/paths/~1sessions~1batch/post)

W odpowiedzi na otwarcie sesji otrzymamy obiekt zawierający `referenceNumber`, który będzie używany w kolejnych krokach do identyfikacji sesji wsadowej.

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
Client.Core.Models.Sessions.BatchSession.OpenBatchSessionRequest openBatchRequest =
    BatchUtils.BuildOpenBatchRequest(zipMeta, encryptionData, encryptedParts, systemCode);

Client.Core.Models.Sessions.BatchSession.OpenBatchSessionResponse openBatchSessionResponse =
    await BatchUtils.OpenBatchAsync(KsefClient, openBatchRequest, accessToken).ConfigureAwait(false);

//BatchUtils.BuildOpenBatchRequest
public static OpenBatchSessionRequest BuildOpenBatchRequest(
    FileMetadata zipMeta,
    EncryptionData encryption,
    IEnumerable<BatchPartSendingInfo> encryptedParts,
    SystemCode systemCode = DefaultSystemCode,
    string schemaVersion = DefaultSchemaVersion,
    string value = DefaultValue)
{
    IOpenBatchSessionRequestBuilderBatchFile builder = OpenBatchSessionRequestBuilder
        .Create()
        .WithFormCode(systemCode: SystemCodeHelper.GetValue(systemCode), schemaVersion: schemaVersion, value: value)
        .WithBatchFile(fileSize: zipMeta.FileSize, fileHash: zipMeta.HashSHA);

    foreach (BatchPartSendingInfo p in encryptedParts)
    {
        builder = builder.AddBatchFilePart(
            ordinalNumber: p.OrdinalNumber,
            fileName: $"part_{p.OrdinalNumber}.zip.aes",
            fileSize: p.Metadata.FileSize,
            fileHash: p.Metadata.HashSHA);
    }

    return builder
        .EndBatchFile()
        .WithEncryption(
            encryptedSymmetricKey: encryption.EncryptionInfo.EncryptedSymmetricKey,
            initializationVector: encryption.EncryptionInfo.InitializationVector)
        .Build();
}

//BatchUtils.OpenBatchAsync
public static async Task<OpenBatchSessionResponse> OpenBatchAsync(
    IKSeFClient client,
    OpenBatchSessionRequest openReq,
    string accessToken)
    => await client.OpenBatchSessionAsync(openReq, accessToken).ConfigureAwait(false);
```

Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
OpenBatchSessionRequestBuilder builder = OpenBatchSessionRequestBuilder.create()
        .withFormCode(SystemCode.FA_2, SchemaVersion.VERSION_1_0E, SessionValue.FA)
        .withOfflineMode(false)
        .withBatchFile(zipMetadata.getFileSize(), zipMetadata.getHashSHA());

for (int i = 0; i < encryptedZipParts.size(); i++) {
        BatchPartSendingInfo part = encryptedZipParts.get(i);
        builder = builder.addBatchFilePart(i + 1, "faktura_part" + (i + 1) + ".zip.aes",part.getMetadata().getFileSize(), part.getMetadata().getHashSHA());
}

OpenBatchSessionRequest request = builder.endBatchFile()
        .withEncryption(
                        encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
                        encryptionData.encryptionInfo().getInitializationVector()
                )
        .build();

OpenBatchSessionResponse response = ksefClient.openBatchSession(request, accessToken);
```

Metoda zwraca listę części paczki; dla każdej części podaje adres uploadu (URL), wymaganą metodę HTTP oraz komplet nagłówków, które należy przesłać razem z daną częścią.

### 5. Przesłanie zadeklarowanych części paczki

Korzystając z danych zwróconych przy otwarciu sesji w `partUploadRequests`, tj. unikalnego adresu url z kluczem dostępu, metody HTTP (method) oraz wymaganych nagłówków (headers), należy przesłać każdą zadeklarowaną część paczki (`fileParts`) pod wskazany adres, stosując dokładnie te wartości dla danej części. Łącznikiem pomiędzy deklaracją a instrukcją wysyłki jest właściwość `ordinalNumber`.

W treści żądania (body) należy umieścić bajty odpowiedniej części pliku (bez opakowania w JSON).

> Uwaga: nie należy dodawać do nagłówków token dostępu (`accessToken`).

Każdą część przesyła się oddzielnym żądaniem HTTP. Zwracane kody odpowiedzi:
* `201` - poprawne przyjęcie pliku,
* `400` - błędne dane,
* `401` - nieprawidłowe uwierzytelnienie,
* `403` - brak uprawnień do zapisu (np. upłynął czas na zapis).

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionE2ETests.cs)

```csharp
await KsefClient.SendBatchPartsAsync(openBatchSessionResponse, encryptedParts);
```

Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
ksefClient.sendBatchParts(response, encryptedZipParts);
```

**Limit czasu na przesłanie partów w sesji wsadowej**  
Wysyłka plików w sesji wsadowej jest ograniczona czasowo. Czas ten zależy wyłącznie od liczby zadeklarowanych partów i wynosi 20 minut na każdy part. Każdy dodatkowy part proporcjonalnie zwiększa limit czasu **dla każdego parta** w paczce.

Łączny czas na wysyłkę każdego parta = liczba partów × 20 minut.  
Przykład. Paczka zawiera 2 party – każdy part ma 40 minut na wysyłkę.

Wielkość parta nie ma znaczenia dla ustalenia limitu czasu – jedynym kryterium jest liczba partów zadeklarowana przy otwarciu sesji.  

Autoryzacja jest weryfikowana na początku każdego żądania HTTP. Jeżeli w momencie przyjęcia żądania adres jest ważny, operacja przesłania zostaje zrealizowana w całości. Wygaśnięcie ważności w trakcie trwania przesyłania nie przerywa rozpoczętej operacji.

### 6. Zamknięcie sesji wsadowej
Po przesłaniu wszystkich części paczki należy zamknąć sesję wsadową, co inicjuje asynchronicznie przetwarzanie paczki faktur ([szczegóły weryfikacji](faktury/weryfikacja-faktury.md)), oraz generowanie zbiorczego UPO.

POST [/sessions/batch/\{referenceNumber\}/close](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/paths/~1sessions~1batch~1%7BreferenceNumber%7D~1close/post)}]

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\BatchSession\BatchSessionStreamE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/BatchSession/BatchSessionStreamE2ETests.cs)
```csharp
await KsefClient.CloseBatchSessionAsync(referenceNumber, accessToken);
```
Przykład w języku Java:
[BatchIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/BatchIntegrationTest.java)

```java
ksefClient.closeBatchSession(referenceNumber, accessToken);
```

Zobacz 
- [Sprawdzenie stanu i pobranie UPO](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md)
- [Weryfikacja faktury](faktury/weryfikacja-faktury.md)
- [Numer KSeF – struktura i walidacja](faktury/numer-ksef.md)
