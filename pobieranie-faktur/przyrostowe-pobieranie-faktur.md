# Przyrostowe pobieranie faktur
21.11.2025

## Wprowadzenie

Przyrostowe pobieranie faktur, oparte na eksporcie paczek (POST [`/invoices/exports`](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1invoices~1exports/post)), jest rekomendowanym mechanizmem synchronizacji między centralnym repozytorium KSeF a lokalnymi bazami danych systemów zewnętrznych. 

Kluczową rolę odgrywa tu mechanizm **[High Water Mark (HWM)](hwm.md)** - stabilny punkt w czasie, do którego system gwarantuje kompletność danych.

## Architektura rozwiązania

Przyrostowe pobieranie opiera się na trzech kluczowych komponentach:

1. **Synchronizacja w oknach czasowych** - wykorzystanie przylegających okien czasowych z uwzględnieniem HWM co zapewnia ciągłość i brak pominięć
2. **Obsługa limitów API** - sterowanie tempem wywołań, obsługa HTTP 429 oraz Retry-After.
3. **Deduplikacja** - eliminacja duplikatów na podstawie metadanych z plików `_metadata.json`.

Metoda bazowa: POST [`/invoices/exports`](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1invoices~1exports/post) inicjuje asynchroniczny eksport. Po zakończeniu przetwarzania status operacji udostępnia unikalne adresy URL do pobrania części paczki.

## Synchronizacja w oknach czasowych (Windowing)

### Koncepcja

Pobieranie faktur odbywa się w przylegających oknach czasowych z wykorzystaniem daty `PermanentStorageHwmDate`. Aby włączyć mechanizm HWM, należy ustawić parametr `restrictToPermanentStorageHwmDate = true` w żądaniu eksportu. Każde kolejne okno rozpoczyna się dokładnie w momencie zakończenia poprzedniego z uwzględnieniem HWM (z wyjątkiem sytuacji opisanej w sekcji [Mechanizm High Water Mark (HWM) i obsługa obciętych paczek](#mechanizm-high-water-mark-hwm-i-obsługa-obciętych-paczek-istruncated)). Przez „moment zakończenia" rozumie się:

- wartość `dateRange.to`, gdy została podana, lub
- `PermanentStorageHwmDate` gdy `dateRange.to` pominięto.  

Takie podejście zapewnia ciągłość zakresów i eliminuje ryzyko pominięcia jakiejkolwiek faktury. Faktury powinny być pobierane oddzielnie dla każdego typu podmiotu (`Podmiot 1`, `Podmiot 2`, `Podmiot 3`, `Podmiot upoważniony`) występującego w dokumencie. Iteracja przez podmioty zapewnia kompletność danych - firma może występować w różnych rolach na fakturach.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
// Słownik do śledzenia punktu kontynuacji dla każdego SubjectType
Dictionary<InvoiceSubjectType, DateTime?> continuationPoints = new();
IReadOnlyList<(DateTime From, DateTime To)> windows = BuildIncrementalWindows(batchCreationStart, batchCreationCompleted);

// Tworzenie planu eksportu - krotki (okno czasowe, typ podmiotu)
IEnumerable<InvoiceSubjectType> subjectTypes = Enum.GetValues<InvoiceSubjectType>().Where(x => x != InvoiceSubjectType.SubjectAuthorized);
IOrderedEnumerable<ExportTask> exportTasks = windows
    .SelectMany(window => subjectTypes, (window, subjectType) => new ExportTask(window.From, window.To, subjectType))
    .OrderBy(task => task.From)
    .ThenBy(task => task.SubjectType);


foreach (ExportTask task in exportTasks)
{
    DateTime effectiveFrom = GetEffectiveStartDate(continuationPoints, task.SubjectType, task.From);

    OperationResponse? exportResponse = await InitiateInvoiceExportAsync(effectiveFrom, task.To, task.SubjectType);

    // Dalsza obsługa eksportu...
```

Przykład w języku ```java```:
[IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
Map<InvoiceQuerySubjectType, OffsetDateTime> continuationPoints = new HashMap<>();

List<TimeWindows> timeWindows = buildIncrementalWindows(batchCreationStart, batchCreationCompleted);
List<InvoiceQuerySubjectType> subjectTypes = Arrays.stream(InvoiceQuerySubjectType.values())
        .filter(x -> x != InvoiceQuerySubjectType.SUBJECTAUTHORIZED)
        .toList();

List<ExportTask> exportTasks = timeWindows.stream()
        .flatMap(window -> subjectTypes.stream()
                .map(subjectType -> new ExportTask(window.getFrom(), window.getTo(), subjectType)))
        .sorted(Comparator.comparing(ExportTask::getFrom)
                .thenComparing(ExportTask::getSubjectType))
        .toList();
exportTasks.forEach(task -> {
        EncryptionData encryptionData = defaultCryptographyService.getEncryptionData();
        OffsetDateTime effectiveFrom = getEffectiveStartDate(continuationPoints, task.getSubjectType(), task.getFrom());
        String operationReferenceNumber = initiateInvoiceExportAsync(effectiveFrom, task.getTo(),
            task.getSubjectType(), accessToken, encryptionData.encryptionInfo());

// Dalsza obsługa eksportu...
```

### Zalecane wielkości okien

- **Częstotliwość i limity**  
    POST `/invoice/exports` wymaga wskazania typu podmiotu (`Podmiot 1`, `Podmiot 2`, `Podmiot 3`, `Podmiot upoważniony`). Zgodnie z [limitami API](../limity/limity-api.md) można zainicjować maksymalnie 20 eksportów na godzinę; harmonogram powinien dzielić tę pulę między wybrane typy podmiotów.
- **Strategia harmonogramu**  
    W trybie ciągłej synchronizacji można przyjąć 4 eksporty/h na każdy typ podmiotu. W praktyce role `Podmiot 3` i `Podmiot upoważniony` zwykle występują rzadziej i mogą być uruchamiane sporadycznie, np. raz na dobę w oknie nocnym.
- **Minimalny interwał**  
    Interwał cykliczny nie powinien być krótszy niż 15 minut dla każdego typu podmiotu (zgodnie z zaleceniami w limitach API).
- **Wielkość okna**
    W scenariuszu ciągłej synchronizacji zalecane jest wywołanie eksportu bez określonej daty końcowej (`DateRange.To` pominięte). W takim przypadku system KSeF przygotowuje możliwie duży, spójny pakiet w granicach limitów algorytmu (liczba faktur, rozmiar danych po kompresji), co ogranicza liczbę wywołań i obciążenie po obu stronach. Gdy `IsTruncated = true`, kolejne wywołanie należy rozpocząć od `LastPermanentStorageDate`, gdy `IsTruncated = false` , kolejne wywołanie należy rozpocząć od zwróconego `PermanentStorageHwmDate`.
- **Brak nakładania**
    Zakresy muszą być przylegające; koniec jednego okna jest początkiem następnego.
- **Punkt kontrolny**
    Punkt kontynuacji wyznaczony przez HWM - `PermanentStorageHwmDate` lub `LastPermanentStorageDate` dla obciętych paczek stanowi początek kolejnego okna.

>Datą otrzymania faktury jest data nadania numeru KSeF. Numer nadawany jest podczas przetwarzania faktury po stronie KSeF i nie zależy od momentu pobrania do systemu zewnętrznego.

## Obsługa limitów API

### Limity według typu endpointów

Wszystkie endpointy związane z pobieraniem faktur podlegają ścisłym limitom API określonym w dokumentacji [Limity API](../limity/limity-api.md). Limity te są wiążące i muszą być respektowane przez każdą implementację przyrostowego pobierania.

W przypadku przekroczenia limitów system zwraca kod HTTP `429` (Too Many Requests) wraz z nagłówkiem `Retry-After` wskazującym czas oczekiwania przed kolejną próbą.

## Inicjalizacja eksportu faktur

### Kluczowe znaczenie daty PermanentStorage

Dla przyrostowego pobierania faktur **konieczne** jest użycie daty typu `PermanentStorage`, które zapewnia wiarygodność danych. Oznacza moment trwałej materializacji rekordu, jest odporna na asynchroniczne opóźnienia procesu przyjmowania danych i pozwala bezpiecznie wyznaczać okna przyrostu.
Tym samym inne typy dat (jak `Issue` czy `Invoicing`) mogą prowadzić do nieprzewidywalnych zachowań w synchronizacji przyrostowej.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
EncryptionData exportEncryption = CryptographyService.GetEncryptionData();

InvoiceQueryFilters filters = new()
{
    SubjectType = subjectType,
    DateRange = new DateRange
    {
        DateType = DateType.PermanentStorage,
        From = windowFromUtc,
        To = windowToUtc,
        RestrictToPermanentStorageHwmDate = true
    }
};

InvoiceExportRequest request = new()
{
    Filters = filters,
    Encryption = exportEncryption.EncryptionInfo
};

OperationResponse response = awat KsefClient.ExportInvoicesAsync(request, _accessToken, ct, includeMetadata: true);
```

Przykład w języku ```java```:
[IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
EncryptionData encryptionData = defaultCryptographyService.getEncryptionData();
InvoiceExportFilters filters = new InvoiceExportFilters();
filters.setSubjectType(subjectType);
filters.setDateRange(new InvoiceQueryDateRange(
        InvoiceQueryDateType.PERMANENTSTORAGE, windowFrom, windowTo)
);

InvoiceExportRequest request = new InvoiceExportRequest();
request.setFilters(filters);
request.setEncryption(encryptionInfo);

InitAsyncInvoicesQueryResponse response = ksefClient.initAsyncQueryInvoice(request, accessToken);
```

## Pobieranie i przetwarzanie paczek

Po zakończeniu eksportu paczka faktur jest dostępna do pobrania jako zaszyfrowane archiwum ZIP dzielone na części. Proces pobierania i przetwarzania obejmuje:

1. **Pobranie części** - każda część pobierana osobno z adresów URL zwróconych w statusie operacji.
2. **Deszyfrowanie AES-256** - każda część jest deszyfrowana przy użyciu klucza i IV wygenerowanych podczas inicjalizacji eksportu.
3. **Składanie paczki** - odszyfrowane części łączone w jeden strumień danych.
4. **Rozpakowanie ZIP** - archiwum zawiera pliki XML faktur oraz plik `_metadata.json`.

### Plik _metadata.json

Zawartość pliku _metadata.json to obiekt JSON z właściwością `invoices` (tablica elementów typu `InvoiceMetadata`, jak zwracany przez POST `/invoices/query/metadata`).
Plik ten jest kluczowy dla mechanizmu deduplikacji, ponieważ zawiera numery KSeF wszystkich faktur w paczce.

**Włączenie metadanych (do 27.10.2025)**  
Aby dołączyć plik `_metadata.json`, należy dodać nagłówek do żądania eksportu:

```http
X-KSeF-Feature: include-metadata
```

**Od 27.10.2025** paczka eksportu będzie zawsze zawierać plik `_metadata.json` bez konieczności dodawania nagłówka.

Przykład w języku ```C#```:

[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

[KSeF.Client.Tests.Utils\BatchSessionUtils.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Utils/BatchSessionUtils.cs)

```csharp
List<InvoiceSummary> metadataSummaries = new();
Dictionary<string, string> invoiceXmlFiles = new(StringComparer.OrdinalIgnoreCase);

// Pobranie, odszyfrowanie i połączenie wszystkich części w jeden strumień
using MemoryStream decryptedArchiveStream = await BatchUtils.DownloadAndDecryptPackagePartsAsync(
    package.Parts, 
    encryptionData, 
    CryptographyService, 
    cancellationToken: CancellationToken)
    .ConfigureAwait(false);

// Rozpakowanie ZIP
Dictionary<string, string> unzippedFiles = await BatchUtils.UnzipAsync(decryptedArchiveStream, CancellationToken).ConfigureAwait(false);

foreach ((string fileName, string content) in unzippedFiles)
{
    if (fileName.Equals(MetadataEntryName, StringComparison.OrdinalIgnoreCase))
    {
        InvoicePackageMetadata? metadata = JsonSerializer.Deserialize<InvoicePackageMetadata>(content, MetadataSerializerOptions);
        if (metadata?.Invoices != null)
        {
            metadataSummaries.AddRange(metadata.Invoices);
        }
    }
    else if (fileName.EndsWith(XmlFileExtension, StringComparison.OrdinalIgnoreCase))
    {
        invoiceXmlFiles[fileName] = content;
    }
}
```

Przykład w języku ```java```:
[IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
 List<InvoicePackagePart> parts = invoiceExportStatus.getPackageParts().getParts();
byte[] mergedZip = FilesUtil.mergeZipParts(
        encryptionData,
        parts,
        part -> ksefClient.downloadPackagePart(part),
        (encryptedPackagePart, key, iv) -> defaultCryptographyService.decryptBytesWithAes256(encryptedPackagePart, key, iv)
);
Map<String, String> downloadedFiles = FilesUtil.unzip(mergedZip);

String metadataJson = downloadedFiles.keySet()
        .stream()
        .filter(fileName -> fileName.endsWith(".json"))
        .findFirst()
        .map(downloadedFiles::get)
        .orElse(null);
InvoicePackageMetadata invoicePackageMetadata = objectMapper.readValue(metadataJson, InvoicePackageMetadata.class);
```

## Mechanizm High Water Mark (HWM) i obsługa obciętych paczek (IsTruncated)

### Koncepcja HWM

High Water Mark (HWM) to mechanizm zapewniający optymalne zarządzanie punktami startowymi dla kolejnych eksportów w przyrostowym pobieraniu faktur. HWM składa się z dwóch komplementarnych składników:

- **`PermanentStorageHwmDate`** - stabilna górna granica danych uwzględnionych w paczce, reprezentująca moment, do którego system gwarantuje kompletność danych.
- **`LastPermanentStorageDate`** - data ostatniej faktury w paczce, używana gdy paczka została obcięta (`IsTruncated = true`).

#### Korzyści mechanizmu HWM

- **Minimalizacja duplikatów** - HWM znacząco redukuje liczbę duplikatów między kolejnymi paczkami
- **Optymalizacja wydajności** - zmniejsza obciążenie mechanizmu deduplikacji  
- **Zachowanie kompletności** - zapewnia, że żadne faktury nie zostaną pominięte
- **Stabilność synchronizacji** - dostarcza niezawodne punkty kontynuacji dla długotrwałych procesów

### Strategia kontynuacji paczek

Flaga `IsTruncated = true` jest ustawiana, gdy podczas budowy paczki osiągnięto limity algorytmu (liczba faktur lub rozmiar danych po kompresji). W takim przypadku w statusie operacji dostępne są obydwie właściwości HWM.
Mechanizm HWM wykorzystuje następującą hierarchię priorytetów dla wyznaczania punktu kontynuacji, Aby zachować ciągłość pobierania i nie pominąć żadnego dokumentu, następne wywołanie eksportu należy rozpocząć od:

1. **Paczka obcięta** (`IsTruncated = true`) - następne wywołanie rozpoczyna się od `LastPermanentStorageDate`
2. **Stabilny HWM** - wykorzystanie `PermanentStorageHwmDate` jako punktu startowego dla następnego okna

- kolejne okno rozpoczyna się w tym samym punkcie co zakończone (przyległość); ewentualne duplikaty zostaną usunięte w etapie deduplikacji na podstawie numerów KSeF z _metadata.json.
Poniżej przykład utrzymywania punktu kontynuacji:

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
private static void UpdateContinuationPointIfNeeded(
    Dictionary<InvoiceSubjectType, DateTime?> continuationPoints,
    InvoiceSubjectType subjectType,
    InvoiceExportPackage package)
{
    // Priorytet 1: Paczka obcięta - LastPermanentStorageDate
    if (package.IsTruncated && package.LastPermanentStorageDate.HasValue)
    {
        continuationPoints[subjectType] = package.LastPermanentStorageDate.Value.UtcDateTime;
    }
    // Priorytet 2: Stabilny HWM jako granica kolejnego okna
    else if (package.PermanentStorageHwmDate.HasValue)
    {
        continuationPoints[subjectType] = package.PermanentStorageHwmDate.Value.UtcDateTime;
    }
    else
    {
        // Zakres w pełni przetworzony - usunięcie punktu kontynuacji
        continuationPoints.Remove(subjectType);
    }
}
```

Przykład w języku ```java```:
[IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
private void updateContinuationPointIfNeeded(Map<InvoiceQuerySubjectType, OffsetDateTime> continuationPoints,
                                                 InvoiceQuerySubjectType subjectType,
                                                 InvoiceExportPackage invoiceExportPackage) {
    if (Boolean.TRUE.equals(invoiceExportPackage.getIsTruncated()) && Objects.nonNull(invoiceExportPackage.getLastPermanentStorageDate())) {
        continuationPoints.put(subjectType, invoiceExportPackage.getLastPermanentStorageDate());
    } else {
        continuationPoints.remove(subjectType);
    }
}
```

## Deduplikacja faktur

### Strategia deduplikacji

Deduplikacja odbywa się na podstawie numerów KSeF zawartych w pliku `_metadata.json`:

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
Dictionary<string, InvoiceSummary> uniqueInvoices = new(StringComparer.OrdinalIgnoreCase);
bool hasDuplicates = false;

// Przetwarzanie metadanych z paczki - dodawanie unikalnych faktur i wykrywanie duplikatów
hasDuplicates = packageResult.MetadataSummaries
    .DistinctBy(s => s.KsefNumber, StringComparer.OrdinalIgnoreCase)
    .Any(summary => !uniqueInvoices.TryAdd(summary.KsefNumber, summary));
```

Przykład w języku ```java```:
[IncrementalInvoiceRetrieveIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/IncrementalInvoiceRetrieveIntegrationTest.java)

```java
hasDuplicates.set(packageProcessingResult.getInvoiceMetadataList()
        .stream()
        .anyMatch(summary -> uniqueInvoices.containsKey(summary.getKsefNumber())));

packageProcessingResult.getInvoiceMetadataList()
        .stream()
        .distinct()
        .forEach(summary -> uniqueInvoices.put(summary.getKsefNumber(), summary));
```

## Powiązane dokumenty

- [High Water Mark](hwm.md)
- [Limity API](../limity/limity-api.md)
- [Pobieranie faktur](pobieranie-faktur.md)
