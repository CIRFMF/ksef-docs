## Sesja wsadowa
10.07.2025

Wysyłka wsadowa umożliwia jednorazowe przesłanie wielu faktur w pojedynczym pliku ZIP, zamiast wysyłać każdą fakturę oddzielnie.

To rozwiązanie przyspiesza i ułatwia przetwarzanie dużej liczby dokumentów — szczególnie dla firm, które generują wiele faktur dziennie.

Każda faktura musi być przygotowana w formacie XML zgodnie z aktualnym schematem opublikowanym przez Ministerstwo Finansów:
* Paczka ZIP powinna być podzielona na części nie większe niż 100 MB (przed zaszyfrowaniem), które są szyfrowane i przesyłane osobno.
* Należy podać informacje o każdej części paczki w obiekcie ```fileParts```.


### Wymagania wstępne
Aby skorzystać z wysyłki wsadowej, należy najpierw przejść proces [uwierzytelnienia](uwierzytelnianie.md) i posiadać aktualny token dostępu (```accessToken```), który uprawnia do korzystania z chronionych zasobów API KSeF.

Przed otwarciem sesji oraz wysłaniem faktur wymagane jest:
* wygenerowanie klucza symetrycznego o długości 256 bitów i wektora inicjującego o długości 128 bitów (IV), dołączanego jako prefiks do szyfrogramu,
* zaszyfrowanie dokumentu algorytmem AES-256-CBC z dopełnianiem PKCS#7,
* zaszyfrowanie klucza symetrycznego algorytmem RSAES-OAEP (padding OAEP z funkcją MGF1 opartą na SHA-256 oraz skrótem SHA-256), przy użyciu klucza publicznego KSeF Ministerstwa Finansów.

Operacje te można zrealizować za pomocą komponentu ```CryptographyService```, dostępnego w oficjalnym kliencie KSeF. Biblioteka ta udostępnia gotowe metody do generowania i szyfrowania kluczy, zgodnie z wymaganiami systemu.

Przykład w języku C#:
```csharp
EncryptionData encryptionData = cryptographyService.GetEncryptionData();
```
Przykład w języku Java:
```java
var cryptographyService = new DefaultCryptographyService(ksefClient);
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

Wygenerowane dane służą do szyfrowania faktur.

### 1. Przygotowanie paczki ZIP
Należy utworzyć paczkę ZIP zawierającą wszystkie faktury, które zostaną przesłane w ramach jednej sesji.  

Przykład w języku C#:
```csharp
// Wczytaj pliki faktur ustrukturyzowanych (xml) do pamięci
    var files = invoices.Select(f => new { FileName = Path.GetFileName(f), Content = System.IO.File.ReadAllBytes(f) }).ToList();

    // Stwórz ZIP w pamięci
    byte[] zipBytes;
    using (var zipStream = new MemoryStream())
    {
        using (var archive = new ZipArchive(zipStream, ZipArchiveMode.Create, true))
        {
            foreach (var file in files)
            {
                var entry = archive.CreateEntry(file.FileName, CompressionLevel.Optimal);
                using var entryStream = entry.Open();
                entryStream.Write(file.Content, 0, file.Content.Length);
            }
        }
        zipBytes = zipStream.ToArray();
    }
```

Przykład w języku Java:
```java
    List<Path> invoices = ....;
    byte[] zipBytes;
    try (ByteArrayOutputStream zipStream = new ByteArrayOutputStream();
        ZipOutputStream archive = new ZipOutputStream(zipStream)) {

        for (Path file : invoices) {
            archive.putNextEntry(new ZipEntry(file.getFileName().toString()));
            byte[] fileContent = Files.readAllBytes(file);
            archive.write(fileContent);
            archive.closeEntry();
        }
        archive.finish();
        zipBytes = zipStream.toByteArray();
    }
```

### 2. Podział binarny paczki ZIP na części

Ze względu na ograniczenia rozmiaru przesyłanych plików, paczka ZIP powinna być podzielona binarnie na mniejsze części, które będą przesyłane osobno. Każda część powinna mieć unikalną nazwę i numer porządkowy.

Przykład w języku C#:
```csharp

 // Pobierz metadane ZIP-a (przed szyfrowaniem)
 var zipMetadata = cryptographyService.GetMetaData(zipBytes);

 // Podziel ZIP na 11 partów
 int partCount = 11;
 int partSize = (int)Math.Ceiling((double)zipBytes.Length / partCount);
 var zipParts = new List<byte[]>();
 for (int i = 0; i < partCount; i++)
 {
     int start = i * partSize;
     int size = Math.Min(partSize, zipBytes.Length - start);
     if (size <= 0) break;
     var part = new byte[size];
     Array.Copy(zipBytes, start, part, 0, size);
     zipParts.Add(part);
 }

```

Przykład w języku Java:
```java
    int numberOfParts = 11;
    var zipMetadata = cryptographyService.getMetaData(zipBytes);
    int partSize = (int) Math.ceil((double) zipBytes.length / numberOfParts);

    List<byte[]> zipParts = new ArrayList<>();
    for (int i = 0; i < numberOfParts; i++) {
        int start = i * partSize;
        int size = Math.min(partSize, zipBytes.length - start);
        if (size <= 0) break;
        byte[] part = Arrays.copyOfRange(zipBytes, start, start + size);
        zipParts.add(part);
    }
```

### 3. Zaszyfrowanie części paczki
Każdą część należy zaszyfrować nowo wygenerowanym kluczem AES‑256‑CBC z dopełnianiem PKCS#7.

Przykład w języku C#:
```csharp
    //  Szyfruj każdy part i pobierz metadane
        var encryptedParts = new List<(byte[] Data, FileHash Metadata)>();
        for (int i = 0; i < zipParts.Count; i++)
        {
            var encrypted = cryptographyService.EncryptBytesWithAES256(zipParts[i], encryptionData.CipherKey, encryptionData.CipherIv);
            var metadata = cryptographyService.GetMetaData(encrypted);
            encryptedParts.Add((encrypted, metadata));

            // Zapisz part na dysku
            var partFileName = Path.Combine(BatchPartsDirectory, $"faktura_part{i + 1}.zip.aes");
            System.IO.File.WriteAllBytes(partFileName, encrypted);
        }
```

Przykład w języku Java:
```java
    List<BatchPartSendingInfo> encryptedZipParts = new ArrayList<>();
    for (int i = 0; i < zipParts.size(); i++) {
        byte[] encryptedZipPart = cryptographyService.encryptBytesWithAES256(
                zipParts.get(i),
                encryptionData.cipherKey(),
                encryptionData.cipherIv()
        );
        FileMetadata zipPartMetadata = cryptographyService.getMetaData(encryptedZipPart);
        encryptedZipParts.add(new BatchPartSendingInfo(encryptedZipPart, zipPartMetadata, (i + 1)));
    }
```

### 4. Otwarcie sesji wsadowej

Inicjalizacja nowej sesji wsadowej z podaniem:
* wersji schematu faktury: [FA(2)](faktury/schemat-FA(2)-v1-0E.xsd), [FA(3)](faktury/schemat-FA(3)-v1-0E.xsd) <br>
określa, którą wersję XSD system będzie stosować do walidacji przesyłanych faktur.
* zaszyfrowanego klucza symetrycznego<br>
symetryczny klucz szyfrujący pliki XML, zaszyfrowany kluczem publicznym Ministerstwa Finansów; rekomendowane jest użycie nowo wygenerowanego klucza dla każdej sesji.
* metadane paczki ZIP i jej części: nazwa pliku, hash, rozmiar oraz lista części (jeśli paczka jest dzielona)

POST [/sessions/batch](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/operation/batch.open)

W odpowiedzi na otwarcie sesji otrzymamy obiekt zawierający `referenceNumber`, który będzie używany w kolejnych krokach do identyfikacji sesji wsadowej.

Przykład w języku C#:
```csharp
//  Buduj request
        var batchFileInfoBuilder = OpenBatchSessionRequestBuilder
            .Create()
            .WithFormCode(systemCode: "FA (2)", schemaVersion: "1-0E", value: "FA")
            .WithBatchFile(
                fileName: "faktury.zip",
                fileSize: zipMetadata.FileSize,
                fileHash: zipMetadata.HashSHA.Value);

        for (int i = 0; i < encryptedParts.Count; i++)
        {
            batchFileInfoBuilder = batchFileInfoBuilder.AddBatchFilePart(
                ordinalNumber: i + 1,
                fileName: $"faktura_part{i + 1}.zip.aes",
                fileSize: encryptedParts[i].Metadata.FileSize,
                fileHash: encryptedParts[i].Metadata.HashSHA.Value);
        }

        var openBatchRequest = batchFileInfoBuilder.EndBatchFile()
            .WithEncryption(
                encryptedSymmetricKey: encryptionData.EncryptionInfo.EncryptedSymmetricKey,
                initializationVector: encryptionData.EncryptionInfo.InitializationVector)
            .Build();

        // Otwórz sesję wsadową
        var openBatchSessionResponse = await ksefClient.OpenBatchSessionAsync(openBatchRequest, accessToken, cancellationToken);
```

Przykład w języku Java:
```java
var builder = OpenBatchSessionRequestBuilder.create()
        .withFormCode("FA (2)", "1-0E", "FA")
        .withOfflineMode(false)
        .withBatchFile(zipMetadata.getFileSize(), zipMetadata.getHashSHA());

for (int i = 0; i < encryptedZipParts.size(); i++) {
    var part = encryptedZipParts.get(i);
    builder = builder.addBatchFilePart(i + 1, "faktura_part" + (i + 1) + ".zip.aes",
            part.getMetadata().getFileSize(), part.getMetadata().getHashSHA());
}

OpenBatchSessionRequest request = builder.endBatchFile()
        .withEncryption(
                encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
                encryptionData.encryptionInfo().getInitializationVector()
        )
        .build();

var openBatchSessionResponse = ksefClient.openBatchSession(request);
```

Metoda zwraca listę części paczki; dla każdej części podaje adres uploadu (URL), wymaganą metodę HTTP oraz komplet nagłówków, które należy przesłać razem z daną częścią.

### 5. Przesłanie zadeklarowanych części paczki

Na podstawie danych zwróconych przy otwarciu sesji (unikalny URL, metoda HTTP, wymagane nagłówki) należy przesłać każdą zadeklarowaną część paczki pod wskazany adres, stosując dokładnie podaną metodę i nagłówki dla danej części.

Przykład w języku C#:
```csharp
await ksefClient.SendBatchPartsAsync(openBatchSessionResponse, BatchPartsDirectory);
```

Przykład w języku Java:
```java
ksefClient.sendBatchParts(openBatchSessionResponse, encryptedZipParts);
```

### 6. Zamknięcie sesji wsadowej
Po przesłaniu wszystkich części paczki należy zamknąć sesję wsadową, co inicjuje asynchronicznie przetwarzanie paczki faktur ([szczegóły weryfikacji](faktury\weryfikacja-faktury.md)), oraz generowanie zbiorczego UPO.

POST [/sessions/batch/\{referenceNumber\}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/paths/~1api~1v2~1sessions~1batch~1%7BreferenceNumber%7D~1close/post)}]

Przykład w języku C#:

```csharp

var closeBatchSessionRequest = CloseBatchSessionRequestBuilder
    .Create()
    .WithReferenceNumber(openBatchSessionResponse.ReferenceNumber)
    .Build();
    var closeBatchSessionResponse = await ksefClient.CloseBatchSessionAsync(closeBatchSessionRequest, accessToken);
```
Przykład w języku Java:
```java
ksefClient.closeBatchSession(referenceNumber);
```

Zobacz 
- [Sprawdzenie stanu i pobranie UPO](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md)
- [Weryfikacja faktury](faktury/weryfikacja-faktury.md)
- [Numer KSeF – struktura i walidacja](numer-ksef.md)
