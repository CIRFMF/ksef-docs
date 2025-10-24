## Kody weryfikujące QR
21.08.2025

Kod QR (Quick Response) to graficzna reprezentacja tekstu, najczęściej adresu URL. W kontekście KSeF jest to zakodowany link zawierający dane identyfikujące fakturę — taki format pozwala na szybkie odczytanie informacji przy pomocy urządzeń końcowych (smartfonów lub skanerów optycznych). Dzięki temu link może być zeskanowany i przekierowany bezpośrednio do odpowiedniego zasobu systemu KSeF odpowiedzialnego za wizualizację i weryfikację faktury lub certyfikatu KSeF wystawcy.

Kody QR wprowadzono z myślą o sytuacjach, gdy faktura trafia do odbiorcy innym kanałem niż bezpośrednie pobranie z API KSeF (np. jako PDF, wydruk czy załącznik e-mail). W takich przypadkach każdy może:
- sprawdzić, czy dana faktura rzeczywiście znajduje się w systemie KSeF i czy nie została zmodyfikowana,
- pobrać jej wersję ustrukturyzowaną (plik XML) bez potrzeby kontaktu z wystawcą,
- potwierdzić autentyczność wystawcy (w przypadku faktur offline).

Generowanie kodów (zarówno dla faktur online, jak i offline) odbywa się lokalnie w aplikacji klienta na podstawie danych zawartych w wystawionej fakturze. Kod QR musi być zgodny z normą ISO/IEC 18004:2015. Jeśli nie ma możliwości umieszczenia kodu bezpośrednio na fakturze (np. format danych tego nie pozwala), należy dostarczyć go odbiorcy jako oddzielny plik graficzny lub link.

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
- adresu URL: `https://ksef-test.mf.gov.pl/client-app/invoice`,
- daty wystawienia faktury (pole `P_1`) w formacie DD-MM-RRRR,
- NIP-u sprzedawcy,
- skrótu pliku faktury obliczonego algorytmem SHA-256 (wyróżnik pliku faktury) w formacie Base64URL.

Przykładowo dla faktury:
- data wystawienia: "01-02-2026",
- NIP sprzedawcy: "1111111111",
- skrót SHA-256 w formacie Base64URL: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE"

Wygenerowany link wygląda następująco:
```
https://ksef-test.mf.gov.pl/client-app/invoice/1111111111/01-02-2026/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

Przykład w języku ```C#```:
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

### 2. KOD II – Weryfikacja certyfikatu

```KOD II``` jest generowany wyłącznie dla faktur wystawianych w trybie offline (offline24, offline-niedostępność systemu, tryb awaryjny) i pełni funkcję potwierdzenia autentyczności wystawcy oraz integralności faktury. Generowanie wymaga posiadania aktywnego [certyfikatu KSeF typu Offline](/certyfikaty-KSeF.md) – link zawiera kryptograficzny podpis URL przy użyciu klucza prywatnego certyfikatu KSeF typu Offline, co zapobiega sfałszowaniu linku przez podmioty nieposiadające dostępu do certyfikatu. 

> **Uwaga**: Certyfikat typu `Authentication` nie może być używany do generowania KODU II. Jego przeznaczeniem jest wyłącznie uwierzytelnienie w API.


Certyfikat KSeF typu Offline można pozyskać za pomocą endpointu [`/certificates`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post).

#### Generowanie linku

Link weryfikacyjny składa się z:
- adresu URL: `https://ksef-test.mf.gov.pl/client-app/certificate`,
- typu identyfikatora kontekstu: "Nip", "InternalId", "NipVatUe", "PeppolId"
- wartości identyfikatora kontekstu,
- NIP-u sprzedawcy,
- numeru seryjnego certyfikatu KSeF,
- skrótu pliku faktury SHA-256 w formacie Base64URL,
- podpisu linku przy użyciu klucza prywatnego certyfikatu KSeF (zakodowany w formacie Base64URL).

**Format podpisu**  
Do podpisu używany jest fragment ścieżki URL bez prefiksu protokołu (https://) i bez końcowego znaku /, np.:
```
ksef-test.mf.gov.pl/client-app/certificate/Nip/1111111111/1111111111/01F20A5D352AE590/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
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
- typ identyfikatora kontekstu: "Nip",
- wartość identyfikatora kontekstu: "1111111111",
- NIP sprzedawcy: "1111111111",
- numer seryjny certyfikatu KSeF: "01F20A5D352AE590",
- skrót SHA-256 w formacie Base64URL: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE",
- podpisu linku przy użyciu klucza prywatnego certyfikatu KSeF: "BRoRSfcLRh71PAonJCFPg55JYXZW24aEsQrZBRctRjQUnxngrVUJmWhMSHH7ikTp7VMnWYkfWOrUTXELmhJ6x-PNZn3cjm0e741c59h6Q5E-KWIQKONvBmn3XWLkncMrOlFMufwP3lFFXz58hSOvnoOzu3j87nLr7niV0jfkwmWZVV2oEjrWZTBCKueWX7Dk7WBUX9pPjFFafkE2iCQdm8MuaW8l-y94xTXYesn3mi8IxpCNo3hcTw_yrGnw-ucAABdhVw7K7MJJacCT2-7_Luh4qiWFiPNcP7Jp_IiI9RQH05xWsxXKA-Z9kgDyjP2KADyKu_vro82bAab4_VW8zQ"

Wygenerowany link wygląda następująco:

```
https://ksef-test.mf.gov.pl/client-app/certificate/Nip/1111111111/1111111111/01635E98D9669239/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE/BRoRSfcLRh71PAonJCFPg55JYXZW24aEsQrZBRctRjQUnxngrVUJmWhMSHH7ikTp7VMnWYkfWOrUTXELmhJ6x-PNZn3cjm0e741c59h6Q5E-KWIQKONvBmn3XWLkncMrOlFMufwP3lFFXz58hSOvnoOzu3j87nLr7niV0jfkwmWZVV2oEjrWZTBCKueWX7Dk7WBUX9pPjFFafkE2iCQdm8MuaW8l-y94xTXYesn3mi8IxpCNo3hcTw_yrGnw-ucAABdhVw7K7MJJacCT2-7_Luh4qiWFiPNcP7Jp_IiI9RQH05xWsxXKA-Z9kgDyjP2KADyKu_vro82bAab4_VW8zQ
```

Przykład w języku ```C#```:
```csharp
 var cert = new X509Certificate2(Convert.FromBase64String(certbase64));
 var url = linkSvc.BuildCertificateVerificationUrl(nip, certSerial, invoiceHash, cert, privateKey);
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
qrOnline = qrCodeService.addLabelToQrCode(qrOnline, invoiceKsefNumber);
```

![QR  Certyfikat](qr/qr-cert.png)