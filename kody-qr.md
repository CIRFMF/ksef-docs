## Kody weryfikujące QR
21.08.2025

Kod QR (Quick Response) to graficzna reprezentacja tekstu, najczęściej adresu URL. W kontekście KSeF jest to zakodowany link zawierający dane identyfikujące fakturę — taki format pozwala na szybkie odczytanie informacji przy pomocy urządzeń końcowych (smartfonów lub skanerów optycznych). Dzięki temu link może być zeskanowany i przekierowany bezpośrednio do odpowiedniego zasobu systemu KSeF odpowiedzialnego za wizualizację i weryfikację faktury lub certyfikatu KSeF wystawcy.

Kody QR wprowadzono z myślą o sytuacjach, gdy faktura trafia do odbiorcy innym kanałem niż bezpośrednie pobranie z API KSeF (np. jako PDF, wydruk czy załącznik e-mail). W takich przypadkach każdy może:
- sprawdzić, czy dana faktura rzeczywiście znajduje się w systemie KSeF i czy nie została zmodyfikowana,
- pobrać jej wersję ustrukturyzowaną (plik XML) bez potrzeby kontaktu z wystawcą,
- potwierdzić autentyczność wystawcy (w przypadku faktur offline).

Generowanie kodów (zarówno dla faktur online, jak i offline) odbywa się lokalnie w aplikacji klienta na podstawie danych zawartych w wystawionej fakturze. Kod QR musi być zgodny z normą ISO/IEC 18004:2024. Jeśli nie ma możliwości umieszczenia kodu bezpośrednio na fakturze (np. format danych tego nie pozwala), należy dostarczyć go odbiorcy jako oddzielny plik graficzny lub link.

### Środowiska

Poniżej zestawiono adresy URL dla poszczególnych środowisk KSeF używanych do generowania kodów QR:

| Skrót     | Środowisko                        | Adres (QR)                                    |
|-----------|-----------------------------------|-----------------------------------------------|
| **TE**    | Testowe <br/> (Release Candidate) | https://qr-test.ksef.mf.gov.pl                |
| **DEMO**  | Przedprodukcyjne (Demo/Preprod)   | https://qr-demo.ksef.mf.gov.pl                |
| **PRD**   | Produkcyjne                       | https://qr.ksef.mf.gov.pl                     |

> **Uwaga**: Poniższe przykłady są przygotowane dla środowiska testowego (TE). Dla pozostałych środowisk należy wykonać analogicznie, używając odpowiedniego adresu URL z powyższej tabeli.

W zależności od trybu wystawienia (online czy offline) na wizualizacji faktury umieszczany jest:
- w trybie **online** — jeden kod QR (KOD I), umożliwiający weryfikację i pobranie faktury z KSeF,
- w trybie **offline** — dwa kody QR:
  - **KOD I** do weryfikacji faktury po jej przesłaniu do KSeF,
  - **KOD II** do potwierdzenia autentyczności wystawcy na podstawie [certyfikatu KSeF](/certyfikaty-KSeF.md).

### 1. KOD I – Weryfikacja i pobieranie faktury

```KOD I``` zawiera link umożliwiający odczyt i weryfikację faktury w systemie KSeF.
Po zeskanowaniu kodu QR lub kliknięciu w link użytkownik otrzyma uproszczoną prezentację podstawowych danych faktury oraz informację o jej obecności w systemie KSeF. Pełny dostęp do treści (np. pobranie pliku XML) wymaga wprowadzenie dodatkowych danych.

#### Generowanie linku
Link składa się z:
- adresu URL: `https://qr-test.ksef.mf.gov.pl/invoice`,
- daty wystawienia faktury (pole `P_1`) w formacie DD-MM-RRRR,
- NIP-u sprzedawcy,
- skrótu pliku faktury obliczonego algorytmem SHA-256 (wyróżnik pliku faktury) w formacie Base64URL.

Przykładowo dla faktury:
- data wystawienia: "01-02-2026",
- NIP sprzedawcy: "1111111111",
- skrót SHA-256 w formacie Base64URL: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE"

Wygenerowany link wygląda następująco:
```
https://qr-test.ksef.mf.gov.pl/invoice/1111111111/01-02-2026/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\QrCode\QrCodeOnlineE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeOnlineE2ETests.cs)
```csharp
string url = linkSvc.BuildInvoiceVerificationUrl(nip, issueDate, invoiceHash);
```

Przykład w języku Java:
```java
String url = linkSvc.buildInvoiceVerificationUrl(nip, issueDate, xml);
```

#### Generowanie kodu QR
Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
private const int PixelsPerModule = 5;
byte[] qrBytes = qrCodeService.GenerateQrCode(url, PixelsPerModule);
```

Przykład w języku Java:
[QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.generateQrCode(invoiceForOnlineUrl);
```

#### Oznaczenie pod kodem QR
Proces przyjęcia faktury przez KSeF zazwyczaj przebiega natychmiastowo — numer KSeF generowany jest niezwłocznie po przesłaniu dokumentu. W wyjątkowych przypadkach (np. wysokie obciążenie systemu) numer może być nadany z niewielkim opóźnieniem.

- **Jeżeli numer KSeF jest znany:** pod kodem QR umieszczany jest numer KSeF faktury (dotyczy faktur online oraz faktur offline już przesłanych do systemu).

![QR KSeF](qr/qr-ksef.png)

- **Jeżeli numer KSeF nie jest jeszcze nadany:** pod kodem QR umieszczany jest napis **OFFLINE** (dotyczy faktur offline przed przesłaniem lub online oczekujących na numer).

![QR Offline](qr/qr-offline.png)

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
byte[] labeled = qrCodeService.AddLabelToQrCode(qrBytes, GeneratedQrCodeLabel);
```

Przykład w języku Java:
[QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.addLabelToQrCode(qrOnline, invoiceKsefNumber);
```

### 2. KOD II - Weryfikacja certyfikatu

```KOD II``` jest generowany wyłącznie dla faktur wystawianych w trybie offline (offline24, offline-niedostępność systemu, tryb awaryjny) i pełni funkcję potwierdzenia autentyczności **wystawcy** oraz jego uprawnień do wystawienia faktury w imieniu sprzedawcy. Generowanie wymaga posiadania aktywnego [certyfikatu KSeF typu Offline](/certyfikaty-KSeF.md) – link zawiera kryptograficzny podpis URL przy użyciu klucza prywatnego certyfikatu KSeF typu Offline, co zapobiega sfałszowaniu linku przez podmioty nieposiadające dostępu do certyfikatu. 

> **Uwaga**: Certyfikat typu `Authentication` nie może być używany do generowania KODU II. Jego przeznaczeniem jest wyłącznie uwierzytelnienie w API.  

Certyfikat KSeF typu Offline można pozyskać za pomocą endpointu [`/certificates`](https://api-test.ksef.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1certificates~1enrollments/post).


#### Weryfikacja po zeskanowaniu kodu QR II

Po przejściu do linku z kodu QR system KSeF dokonuje automatycznej weryfikacji certyfikatu wystawcy.
Proces ten obejmuje następujące etapy:

1. **Certyfikat KSeF wystawcy**

   * Certyfikat istnieje w rejestrze certyfikatów KSeF i jest **ważny**.
   * Certyfikat nie został **odwołany**, **zablokowany** ani nie utracił ważności (`validTo`).

2. **Podpis wystawcy**

   * System weryfikuje **poprawność podpisu** dołączonego w URL.

3. **Uprawnienia wystawcy**

   * Podmiot identyfikowany przez certyfikat wystawcy posiada **aktywne uprawnienia** do wystawienia faktury w kontekście (`ContextIdentifier`),
   * Weryfikacja odbywa się zgodnie z zasadami opisanymi w dokumencie [uwierzytelnianie.md](uwierzytelnianie.md),
   * Przykładowo: księgowa podpisująca fakturę w imieniu firmy A musi mieć nadane prawo `InvoiceWrite` w kontekście tej firmy.

4. **Zgodność kontekstu i NIP sprzedawcy**

   * System sprawdza, czy kontekst (`ContextIdentifier`) ma prawo do wystawiania faktury dla danego **NIP-u sprzedawcy** (`Podmiot1` faktury).
     Dotyczy to m.in. przypadków:
     * samofakturowania (`SelfInvoicing`),
     * przedstawiciel podatkowy (`TaxRepresentative`),
     * grup VAT,
     * jednostek JST,
     * jednostek podrzędnych identyfikowanych identyfikatorem wewnętrzym,
     * komornik,
     * organ egzekucyjny,
     * faktur PEF wystawianych w imieniu innego podmiotu przez dostawcę usług Peppol,
     * faktur wystawionych przez podmiot europejski.

    **Przykład 1. Wystawienie faktury przez podmiot we własnym kontekście**

    Podmiot wystawia fakturę, używając certyfikatu zawierającego jego własny numer NIP.
    Faktura jest wystawiana w kontekście tego samego podmiotu, a w polu NIP sprzedawcy wskazany jest jego własny numer NIP.

    | Identyfikator wystawcy (certyfikat) | Kontekst   | NIP sprzedawcy |
    | -------------------------------------------- | ---------- | -------------- |
    | 1111111111                                   | 1111111111 | 1111111111     |

    **Przykład 2. Wystawienie faktury przez osobę uprawnioną w imieniu podmiotu**

    Osoba fizyczna (np. księgowa) posługująca się certyfikatem KSeF zawierającym numer PESEL wystawia fakturę w kontekście podmiotu, w imieniu którego posiada odpowiednie uprawnienia.
    W polu NIP sprzedawcy wskazany jest numer NIP tego podmiotu.


    | Identyfikator wystawcy (certyfikat) | Kontekst   | NIP sprzedawcy |
    | -------------------------------------------- | ---------- | -------------- |
    | 22222222222                                  | 1111111111 | 1111111111     |


    **Przykład 3. Wystawienie faktury w imieniu innego podmiotu**

    Osoba fizyczna wystawia fakturę w kontekście podmiotu A, jednak na fakturze w polu NIP sprzedawcy wskazany jest numer NIP innego podmiotu B.
    Sytuacja ta jest możliwa, gdy podmiot A posiada nadane uprawnienia wystawiania faktur w imieniu podmiotu B np. przedstawiciel podatkowy, samofakturowanie.

    | Identyfikator wystawcy (certyfikat) | Kontekst   | NIP sprzedawcy |
    | -------------------------------------------- | ---------- | -------------- |
    | 22222222222                                  | 1111111111 | 3333333333     |


#### Generowanie linku

Link weryfikacyjny składa się z:
- adresu URL: `https://qr-test.ksef.mf.gov.pl/certificate`,
- typu identyfikatora kontekstu logowania ([`ContextIdentifier`](uwierzytelnianie.md)): "Nip", "InternalId", "NipVatUe", "PeppolId"
- wartości identyfikatora kontekstu logowania,
- NIP-u sprzedawcy,
- numeru seryjnego certyfikatu KSeF wystawcy,
- skrótu pliku faktury SHA-256 w formacie Base64URL,
- podpisu linku przy użyciu klucza prywatnego certyfikatu KSeF wystawcy (zakodowany w formacie Base64URL).

**Format podpisu**  
Do podpisu używany jest fragment ścieżki URL bez prefiksu protokołu (https://) i bez końcowego znaku /, np.:
```
qr-test.ksef.mf.gov.pl/certificate/Nip/1111111111/1111111111/01F20A5D352AE590/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

**Algorytmy podpisu:**  

* **RSA (RSASSA-PSS)**  
  - Funkcja skrótu: SHA-256  
  - MGF: MGF1 z SHA-256  
  - Długość losowej domieszki (soli): 32 bajty
  - Wymagana długość klucza: Minimum 2048 bity.
  
  Ciąg do podpisu jest najpierw haszowany algorytmem SHA-256, a następnie generowany jest podpis zgodnie ze schematem RSASSA-PSS.  

* **ECDSA (P-256/SHA-256)**  
  Ciąg do podpisu jest haszowany algorytmem SHA-256, a następnie generowany jest podpis z użyciem klucza prywatnego ECDSA opartego na krzywej NIST P-256 (secp256r1), której wybór należy wskazać podczas generowania CSR.  

  Wartość podpisu to para liczb całkowitych (r, s). Może być zakodowana w jednym z dwóch formatów:  
  - **IEEE P1363 Fixed Field Concatenation** – **rekomendowany sposób** z uwagi na krótszy ciąg wynikowy i stałą długość. Format prostszy i krótszy niż DER. Podpis to konkatenacja R || S (po 32 bajty big-endian).  
  - **ASN.1 DER SEQUENCE (RFC 3279)** – podpis jest kodowany jako ASN.1 DER.  Rozmiar podpisu jest zmienny. Proponujemy użycie tego rodzaju podpisu tylko, gdy IEEE P1363 nie jest możliwy z powodu ograniczeń technologicznych.  

W obu przypadkach (niezależnie od wyboru RSA czy ESDSA) otrzymaną wartość podpisu należy zakodować w formacie Base64URL.


Przykładowo dla faktury:
- typ identyfikatora kontekstu logowania: "Nip",
- wartość identyfikatora kontekstu: "1111111111",
- NIP sprzedawcy: "1111111111",
- numer seryjny certyfikatu KSeF wystawcy: "01F20A5D352AE590",
- skrót SHA-256 w formacie Base64URL: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE",
- podpisu linku przy użyciu klucza prywatnego certyfikatu KSeF wystawcy: "mSkm_XmM9fq7PgAJwiL32L9ujhyguOEV48cDB0ncemD2r9TMGa3lr0iRoFk588agCi8QPsOuscUY1rZ7ff76STbGquO-gZtQys5_fHdf2HUfDqPqVTnUS6HknBu0zLkyf9ygoW7WbH06Ty_8BgQTlOmJFzNWSt9WZa7tAGuAE9JOooNps-KG2PYkkIP4q4jPMp3FKypAygHVnXtS0RDGgOxhhM7LWtFP7D-dWINbh5yXD8Lr-JVbeOpyQjHa6WmMYavCDQJ3X_Z-iS01LZu2s1B3xuOykl1h0sLObCdADrbxOONsXrvQa61Xt_rxyprVraj2Uf9pANQgR4-12HEcMw"

Wygenerowany link wygląda następująco:

```
https://qr-test.ksef.mf.gov.pl/certificate/Nip/1111111111/1111111111/01F20A5D352AE590/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE/mSkm_XmM9fq7PgAJwiL32L9ujhyguOEV48cDB0ncemD2r9TMGa3lr0iRoFk588agCi8QPsOuscUY1rZ7ff76STbGquO-gZtQys5_fHdf2HUfDqPqVTnUS6HknBu0zLkyf9ygoW7WbH06Ty_8BgQTlOmJFzNWSt9WZa7tAGuAE9JOooNps-KG2PYkkIP4q4jPMp3FKypAygHVnXtS0RDGgOxhhM7LWtFP7D-dWINbh5yXD8Lr-JVbeOpyQjHa6WmMYavCDQJ3X_Z-iS01LZu2s1B3xuOykl1h0sLObCdADrbxOONsXrvQa61Xt_rxyprVraj2Uf9pANQgR4-12HEcMw
```

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\QrCode\QrCodeOfflineE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeOfflineE2ETests.cs)
```csharp
 var certificate = new X509Certificate2(Convert.FromBase64String(certbase64));

 byte[] qrOfflineCode = QrCodeService.GenerateQrCode(
    linkService.BuildCertificateVerificationUrl(
        nip,
        nip,
        certificate.CertificateSerialNumber,
        invoiceHash,
        certificate));
```

Przykład w języku Java:
[QrCodeOfflineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOfflineIntegrationTest.java)

```java
String pem = privateKeyPemBase64.replaceAll("\\s+", "");
byte[] keyBytes = java.util.Base64.getDecoder().decode(pem);

String url = verificationLinkService.buildCertificateVerificationUrl(
    contextNip,
    ContextIdentifierType.NIP,
    contextNip,
    certificate.getCertificateSerialNumber(),
    invoiceHash,
    cryptographyService.parsePrivateKeyFromPem(keyBytes));
```

#### Generowanie QR kodu
Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
byte[] qrBytes = qrCodeService.GenerateQrCode(url, PixelsPerModule);
```

Przykład w języku Java:
[QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.generateQrCode(invoiceForOnlineUrl);
```

#### Oznaczenie pod kodem QR

Pod kodem QR powinien znaleźć się podpis **CERTYFIKAT**, wskazujący na funkcję weryfikacji certyfikatu KSeF.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\QrCode\QrCodeE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/QrCode/QrCodeE2ETests.cs)

```csharp
private const string GeneratedQrCodeLabel = "CERTYFIKAT";
byte[] labeled = qrCodeService.AddLabelToQrCode(qrBytes, GeneratedQrCodeLabel);
```

Przykład w języku Java:
[QrCodeOnlineIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/QrCodeOnlineIntegrationTest.java)

```java
byte[] qrOnline = qrCodeService.addLabelToQrCode(qrOnline, invoiceKsefNumber);
```

![QR  Certyfikat](qr/qr-cert.png)