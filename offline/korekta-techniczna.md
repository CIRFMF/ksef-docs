# Korekta techniczna faktury offline  
20.08.2025  

## Opis funkcjonalności  
Korekta techniczna umożliwia ponowne przesłanie faktury wystawionej w [trybie offline](../tryby-offline.md), która po przesłaniu do systemu KSeF została **odrzucona** z powodu błędów technicznych, np.:  
- niezgodność ze schematem,  
- przekroczenie dopuszczalnego rozmiaru pliku,
- duplikat faktury,
- inne **błędy walidacji technicznej** uniemożliwiające nadanie ```numeru KSeF```.


> **Uwaga**!
1. Korekta techniczna **nie dotyczy** sytuacji związanych z brakiem uprawnień podmiotów występujących na fakturze (np. samofakturowanie, walidacja relacji dla JST lub grup VAT).
2. W tym trybie **nie jest dozwolona** korygowanie treści faktury – korekta techniczna dotyczy wyłącznie problemów technicznych uniemożliwiających jej przyjęcie w systemie KSeF.
3. Korekta techniczna może być przesyłana wyłącznie w [sesji interaktywnej](../sesja-interaktywna.md), natomiast może dotyczyć faktur offline odrzuconych zarówno w [sesji interaktywnej](../sesja-interaktywna.md), jak i w [sesji wsadowej](../sesja-wsadowa.md).
4. Niedozwolone jest korygowanie technicznie faktury offline, dla której została już przyjęta inna prawidłowa korekta.

## Przykładowy przebieg korekty technicznej faktury offline  

1. **Sprzedawca wystawia fakturę w trybie offline.**  
   - Faktura zawiera dwa kody QR:  
     - **Kod QR I** – umożliwia weryfikację faktury w systemie KSeF,  
     - **Kod QR II** – umożliwia potwierdzenie autentyczności wystawcy na podstawie [certyfikatu KSeF](../certyfikaty-KSeF.md).  

2. **Klient otrzymuje wizualizację faktury (np. w postaci wydruku).**  
   - Po zeskanowaniu **kodu QR I** klient uzyskuje informację, że faktura **nie została jeszcze przesłana do systemu KSeF**.  
   - Po zeskanowaniu **kodu QR II** klient uzyskuje informację o certyfikacie KSeF, który potwierdza autentyczność wystawcy.  

3. **Sprzedawca przesyła fakturę offline do systemu KSeF.**  
   - System KSeF weryfikuje dokument.  
   - Faktura zostaje **odrzucona** z powodu błędu technicznego (np. niepoprawna zgodność ze schematem XSD).  

4. **Sprzedawca aktualizuje swoje oprogramowanie** i ponownie generuje fakturę o tej samej treści, ale zgodną ze  schematem.  
   - Ponieważ zawartość XML różni się od wersji pierwotnej, **skrót SHA-256 pliku faktury jest inny**.

5. **Sprzedawca wysyła poprawioną fakturę jako korektę techniczną.**  
   - Wskazuje w polu `hashOfCorrectedInvoice` skrót SHA-256 pierwotnej, odrzuconej faktury offline.  
   - Parametr `offlineMode` ustawiony jest na `true`.  

6. **System KSeF poprawnie przyjmuje fakturę.**  
   - Dokument otrzymuje numer KSeF.  
   - Faktura zostaje **powiązana z pierwotną fakturą offline**, której skrót został wskazany w polu `hashOfCorrectedInvoice`.  
   - Dzięki temu możliwe jest przekierowanie klienta ze „starego” kodu QR I do poprawionej faktury.

7. **Klient korzysta z kodu QR I umieszczonego na pierwotnej fakturze.**  
   - System KSeF informuje, że **pierwotna faktura została technicznie poprawiona**.  
   - Klient otrzymuje metadane nowej, poprawnie przetworzonej faktury i ma możliwość jej pobrania z systemu.  

## Wysłanie korekty  

Korekta przesyłana jest zgodnie z zasadami opisanymi w dokumencie [sesja interaktywna](../sesja-interaktywna.md), z dodatkowym ustawieniem:  
- `offlineMode: true`,  
- `hashOfCorrectedInvoice` – skrót faktury pierwotnej.  

Przykład w języku C#:
```csharp
var sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA, invoiceMetadata.FileSize)
    .WithEncryptedDocumentHash(
        encryptedInvoiceMetadata.HashSHA, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedDocumentContent(Convert.ToBase64String(encryptedInvoice))
    .WithOfflineMode(true)
    .WithHashOfCorrectedInvoice(hashOfCorrectedInvoice)    
    .Build();
```

Przykład w języku Java:
[OnlineSessionController#sendTechnicalCorrection.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/main/java/pl/akmf/ksef/sdk/api/OnlineSessionController.java#L120)
```java
SendInvoiceOnlineSessionRequest sendInvoiceOnlineSessionRequest = new SendInvoiceOnlineSessionRequestBuilder()
           .withInvoiceHash(invoiceMetadata.getHashSHA())
           .withInvoiceSize(invoiceMetadata.getFileSize())
           .withEncryptedInvoiceHash(encryptedInvoiceMetadata.getHashSHA())
           .withEncryptedInvoiceSize(encryptedInvoiceMetadata.getFileSize())
           .withEncryptedInvoiceContent(Base64.getEncoder().encodeToString(encryptedInvoice))
           .withOfflineMode(true)
           .withHashOfCorrectedInvoice(hashOfCorrectedInvoice)
        .build();
```

## Powiązane dokumenty
- [Tryby offline](../tryby-offline.md)
- [Kody QR](../kody-qr.md)  