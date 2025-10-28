# Przyrostowe pobieranie faktur

21.10.2025

## Wprowadzenie

Przyrostowe pobieranie faktur, oparte na eksporcie paczek (POST [`/invoice/exports`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post)), jest rekomendowanym mechanizmem synchronizacji między centralnym repozytorium KSeF a lokalnymi bazami danych systemów zewnętrznych. 

## Architektura rozwiązania

Przyrostowe pobieranie opiera się na trzech kluczowych komponentach:

1. **Synchronizacja w oknach czasowych** - wykorzystanie przylegających okien czasowych wyznaczanych względem daty `PermanentStorage` co zapewnia ciągłość i brak pominięć.
2. **Obsługa limitów API** - sterowanie tempem wywołań, obsługa HTTP 429 oraz Retry-After.
3. **Deduplikacja** - eliminacja duplikatów na podstawie metadanych z plików `_metadata.json`.

Metoda bazowa: POST [`/invoice/exports`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post) inicjuje asynchroniczny eksport. Po zakończeniu przetwarzania status operacji udostępnia unikalne adresy URL do pobrania części paczki.

## Synchronizacja w oknach czasowych (Windowing)

### Koncepcja

Pobieranie faktur odbywa się w przylegających oknach czasowych z wykorzystaniem daty `PermanentStorage`, gdzie każde kolejne okno rozpoczyna się dokładnie w momencie zakończenia poprzedniego (z wyjątkiem sytuacji opisanej w sekcji [Obsługa obciętych paczek (IsTruncated)](#obsługa-obciętych-paczek-istruncated)). Przez „moment zakończenia” rozumie się:
- wartość `dateRange.to`, gdy została podana, lub
- `PermanentStorageDate` ostatniej faktury ujętej w paczce, gdy `dateRange.to` pominięto.  

Takie podejście zapewnia ciągłość zakresów i eliminuje ryzyko pominięcia jakiejkolwiek faktury. Faktury powinny być pobierane oddzielnie dla każdego typu podmiotu (`Podmiot 1`, `Podmiot 2`, `Podmiot 3`, `Podmiot upoważniony`) występującego w dokumencie. Iteracja przez podmioty zapewnia kompletność danych - firma może występować w różnych rolach na fakturach.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
// Słownik do śledzenia punktu kontynuacji dla każdego SubjectType
Dictionary<SubjectType, DateTime?> continuationPoints = new();
IReadOnlyList<(DateTime From, DateTime To)> windows = BuildIncrementalWindows(batchCreationStart, batchCreationCompleted);

// Tworzenie planu eksportu - krotki (okno czasowe, typ podmiotu)
IEnumerable<SubjectType> subjectTypes = Enum.GetValues<SubjectType>().Where(x => x != SubjectType.SubjectAuthorized);
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

### Zalecane wielkości okien

- **Częstotliwość i limity**  
    POST `/invoice/exports` wymaga wskazania typu podmiotu (`Podmiot 1`, `Podmiot 2`, `Podmiot 3`, `Podmiot upoważniony`). Zgodnie z [limitami API](../limity/limity-api.md) można zainicjować maksymalnie 20 eksportów na godzinę; harmonogram powinien dzielić tę pulę między wybrane typy podmiotów.
- **Strategia harmonogramu**  
    W trybie ciągłej synchronizacji można przyjąć 4 eksporty/h na każdy typ podmiotu. W praktyce role `Podmiot 3` i `Podmiot upoważniony` zwykle występują rzadziej i mogą być uruchamiane sporadycznie, np. raz na dobę w oknie nocnym.
- **Minimalny interwał**  
    Interwał cykliczny nie powinien być krótszy niż 15 minut dla każdego typu podmiotu (zgodnie z zaleceniami w limitach API).
- **Wielkość okna**
    W scenariuszu ciągłej synchronizacji zalecane jest wywołanie eksportu bez określej daty końcowej (`DateRange.To` pominięte). W takim przypadku system KSeF przygotowuje możliwie duży, spójny pakiet w granicach limitów algorytmu (liczba faktur, rozmiar danych po kompresji), co ogranicza liczbę wywołań i obciążenie po obu stronach. Gdy `IsTruncated = true`, kolejne wywołanie należy rozpocząć od `LastPermanentStorageDate`.
- **Brak nakładania**
    Zakresy muszą być przylegające; koniec jednego okna jest początkiem następnego.
- **Punkt kontrolny**
    Ostatnia wartość `PermanentStorageDate` z poprawnie pobranej paczki stanowi początek kolejnego okna.

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
        DateType = DateType.PermanentStorage, // WAŻNE: Użyj PermanentStorage dla przyrostowego pobierania
        From = windowFromUtc,
        To = windowToUtc
    }
};

InvoiceExportRequest request = new()
{
    Filters = filters,
    Encryption = exportEncryption.EncryptionInfo
};

OperationResponse response = await KsefRateLimitWrapper.ExecuteWithRetryAsync(
    ksefApiCall: ct => KsefClient.ExportInvoicesAsync(request, _accessToken, ct, includeMetadata: true),
    endpoint: KsefApiEndpoint.InvoiceExport,
    cancellationToken: CancellationToken);
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
    cancellationToken: CancellationToken);

// Rozpakowanie ZIP
Dictionary<string, string> unzippedFiles = await BatchUtils.UnzipAsync(decryptedArchiveStream, CancellationToken);

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

## Obsługa obciętych paczek (IsTruncated)

Flaga `IsTruncated = true` jest ustawiana, gdy podczas budowy paczki osiągnięto limity algorytmu (liczba faktur lub rozmiar danych po kompresji). W takim przypadku w statusie operacji dostępna jest właściwość `LastPermanentStorageDate` - data ostatniej faktury ujętej w paczce.

Aby zachować ciągłość pobierania i nie pominąć żadnego dokumentu:
- następne wywołanie eksportu należy rozpocząć od `LastPermanentStorageDate` (ustawić `DateRange.From` = `LastPermanentStorageDate`, a `DateRange.To` można pominąć),
- kolejne okno rozpoczyna się w tym samym punkcie co zakończone (przyległość); ewentualne duplikaty zostaną usunięte w etapie deduplikacji na podstawie numerów KSeF z _metadata.json.

Poniżej przykład utrzymywania punktu kontynuacji:

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\Invoice\IncrementalInvoiceRetrievalE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Invoice/IncrementalInvoiceRetrievalE2ETests.cs)

```csharp
private static void UpdateContinuationPointIfNeeded(
    Dictionary<SubjectType, DateTime?> continuationPoints,
    SubjectType subjectType,
    InvoiceExportPackage package)
{
    if (package.IsTruncated && package.LastPermanentStorageDate.HasValue)
    {
        // Ustaw continuation point na datę ostatniej faktury w obciętym pakiecie
        continuationPoints[subjectType] = package.LastPermanentStorageDate.Value.UtcDateTime;
    }
    else
    {
        // Jeśli pakiet nie jest obcięty, usuń continuation point - zakres został w pełni przetworzony
        continuationPoints.Remove(subjectType);
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

## Powiązane dokumenty

- [Limity API](../limity/limity-api.md)
- [Pobieranie faktur](pobieranie-faktur.md)
