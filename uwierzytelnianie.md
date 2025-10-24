## Uwierzytelnianie
10.07.2025

## Wstęp
Uwierzytelnianie w systemie KSeF API 2.0 jest obowiązkowym etapem, który należy wykonać przed dostępem do chronionych zasobów systemu. Proces ten oparty jest na **uzyskaniu tokena dostępowego** (```accessToken```) w formacie ```JWT```, który następnie wykorzystywany jest do autoryzacji operacji API.

Proces uwierzytelnienia opiera się na dwóch elementach:
* Kontekst logowania – określa podmiot, w imieniu którego wykonywane będą operacje w systemie, np. firmę identyfikowaną przez numer NIP.
* Podmiot uwierzytelniający – wskazuje, kto podejmuje próbę uwierzytelnienia. Sposób przekazania tej informacji zależy od wybranej metody uwierzytelnienia.

**Dostępne metody uwierzytelniania:**
* **Z wykorzystaniem podpisu XAdES** <br>
Przesyłany jest dokument XML (```AuthTokenRequest```) zawierający podpis cyfrowy w formacie XAdES. Informacja o podmiocie uwierzytelniającym odczytywana jest z certyfikatu użytego do podpisu (np. NIP, PESEL lub fingerprint certyfikatu).
* **Za pomocą tokena KSeF** <br>
Przesyłany jest dokument JSON zawierający wcześniej uzyskany token systemowy (tzw. [token KSeF](tokeny-ksef.md)). 
Informacja o podmiocie uwierzytelniającym odczytywana jest na podstawie przesłanego [tokena KSeF](tokeny-ksef.md).

Podmiot uwierzytelniający podlega weryfikacji – system sprawdzi, czy wskazany podmiot posiada co najmniej jedno aktywne uprawnienie do wybranego kontekstu. Brak takich uprawnień uniemożliwia uzyskanie tokena dostępowego i korzystanie z API.

Uzyskany token jest ważny tylko przez określony czas i może być odświeżany bez ponownego procesu uwierzytelniania.
Tokeny są automatycznie unieważniane w przypadku utraty uprawnień.

## Proces uwierzytelniania

> **Szybki start (demo)**
>
> W celu demonstracji pełnego przebiegu procesu uwierzytelnienia (pobranie wyzwania, przygotowanie i podpisanie XAdES, przesłanie, sprawdzenie statusu, pobranie tokenów `accessToken` i `refreshToken`) można skorzystać z aplikacji demonstracyjnej. Szczegóły znajdują się w dokumencie: **[Testowe certyfikaty i podpisy XAdES](auth/testowe-certyfikaty-i-podpisy-xades.md)**.
>
> **Uwaga:** samopodpisane certyfikaty są dopuszczalne wyłącznie w środowisku testowym.

### 1. Uzyskanie auth challenge

Proces uwierzytelniania rozpoczyna się od pobrania tzw. **auth challenge**, który stanowi element wymagany do dalszego utworzenia żądania uwierzytelniającego.
Challenge pobierany jest za pomocą wywołania:<br>
POST [/auth/challenge](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1challenge/post)<br>

Czas życia challenge'a wynosi 10 minut.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp

AuthChallengeResponse challenge = await KsefClient.GetAuthChallengeAsync();
```

Przykład w języku ```Java```:
[BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthenticationChallengeResponse challenge = createKSeFClient().getAuthChallenge();
```
Odpowiedź zwraca challenge i timestamp.

### 2. Wybór metody potwierdzenia tożsamości

### 2.1. Uwierzytelnianie **kwalifikowanym podpisem elektronicznym**

#### 1. Przygotowanie dokumentu XML (AuthTokenRequest)

Po uzyskaniu auth challenge należy przygotować dokument XML zgodny ze schematem [AuthTokenRequest](https://ksef-test.mf.gov.pl/docs/v2/schemas/authv2.xsd), który zostanie wykorzystany w dalszym procesie uwierzytelniania. Dokument ten zawiera:


|    Klucz     |           Wartość                                                                                                                              |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| Challenge    | `Wartość otrzymana z wywołania POST [/auth/challenge](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uzyskiwanie-dostepu/paths/~1api~1v2~1auth~1challenge/post)`                                                                                                          |
| ContextIdentifier| `Identyfikator kontekstu, dla którego realizowane jest uwierzytelnienie (NIP, identyfikator wewnętrzny, identyfikator złożony VAT UE)`                                                                       |
| SubjectIdentifierType | `Sposób identyfikacji podmiotu uwierzytelniającego się. Możliwe wartości: certificateSubject (np. NIP/PESEL z certyfikatu) lub certificateFingerprint (odcisk palca certyfikatu).` |    
|(opcjonalnie) AuthorizationPolicy | `Reguły autoryzacyjne. Obecnie obsługiwana lista dozwolonych adresów IP klienta.` |    
 

 Przykładowy dokumenty XML:
 * SubjectIdentifierType z [certificateSubject](auth/subject-identifier-type-certificate-subject.md)
 * SubjectIdentifierType z [certificateFingerprint](auth/subject-identifier-type-certificate-fingerprint.md)
 * ContextIdentifier z [NIP](auth/context-identifier-nip.md)
 * ContextIdentifier z [InternalId](auth/context-identifier-internal-id.md)
 * ContextIdentifier z [NipVatUe](auth/context-identifier-nip-vat-ue.md)

 W kolejnym kroku dokument zostanie podpisany z wykorzystaniem certyfikatu podmiotu.

 **Przykłady implementacji:** <br>

| `ContextIdentifier`                                    | `SubjectIdentifierType`                                       | Znaczenie                                                                                                                                                                                                                                                                                               |
| -------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Type: nip`<br>`Value: 1234567890` | `certificateSubject`<br>` (NIP 1234567890 w certyfikacie)`    | Uwierzytelnienie dotyczy firmy o NIP 1234567890. Podpis zostanie złożony certyfikatem zawierającym w polu 2.5.4.97 NIP 1234567890.                                                       |
| `Type: nip`<br>`Value: 1234567890` | `certificateSubject`<br>` (pesel 88102341294 w certyfikacie)` | Uwierzytelnienie dotyczy firmy o NIP 1234567890. Podpis zostanie złożony certyfikatem osoby fizycznej zawierającym w polu 2.5.4.5 numer PESEL 88102341294. System KSeF sprawdzi, czy ta osoba posiada **uprawnienia do działania** w imieniu firmy (np. na podstawie zgłoszenia ZAW-FA). |
| `Type: nip`<br>`Value: 1234567890` | `certificateFingerprint:`<br>` (odcisk certyfikatu  70a992150f837d5b4d8c8a1c5269cef62cf500bd)` | Uwierzytelnienie dotyczy firmy o NIP 1234567890. Podpis zostanie złożony certyfikatem o odcisku 70a992150f837d5b4d8c8a1c5269cef62cf500bd na który złożono **uprawnienia do działania** w imieniu firmy (np. na podstawie zgłoszenia ZAW-FA). |

Przykład w języku C#:
[KSeF.Client.Tests.Core\E2E\Authorization\AuthorizationE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/Authorization/AuthorizationE2ETests.cs)

```csharp
var authorizationPolicy = new AuthorizationPolicy
{
    AllowedIps = new AllowedIps
    {
        Ip4Addresses = ["192.168.0.1", "192.222.111.1"],
        Ip4Masks = ["192.168.1.0/24"], // Przykładowa maska
        Ip4Ranges = ["222.111.0.1-222.111.0.255"] // Przykładowy zakres IP
    }
};

AuthenticationTokenRequest authTokenRequest = AuthTokenRequestBuilder
    .Create()
    .WithChallenge(challengeResponse.Challenge)
    .WithContext(AuthenticationTokenContextIdentifierType.Nip, ownerNip)
    .WithIdentifierType(AuthenticationTokenSubjectIdentifierTypeEnum.CertificateSubject)
    .WithAuthorizationPolicy(authorizationPolicy)
    .Build();
```

Przykład w języku ```Java```:
[BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthTokenRequest authTokenRequest = new AuthTokenRequestBuilder()
    .withChallenge(challenge.getChallenge())
    .withContextNip(context)
    .withSubjectType(SubjectIdentifierTypeEnum.CERTIFICATE_SUBJECT)
    .build();
```

#### 2. Podpisanie dokumentu (XAdES)

Po przygotowaniu dokumentu ```AuthTokenRequest``` należy go podpisać cyfrowo w formacie XAdES (XML Advanced Electronic Signatures). Jest to wymagany format podpisu dla procesu uwierzytelniania. Do podpisania dokumentu można wykorzystać:
* Certyfikat kwalifikowany osoby fizycznej – zawierający numer PESEL lub NIP osoby posiadającej uprawnienia do działania w imieniu firmy,
* Certyfikat kwalifikowany organizacji (tzw. pieczęć firmowa) - zawierający numer NIP.
* Profil Zaufany (ePUAP) – umożliwia podpisanie dokumentu; wykorzystywany przez osoby fizyczne, które mogą go złożyć za pośrednictwem [gov.pl](https://www.gov.pl/web/gov/podpisz-dokument-elektronicznie-wykorzystaj-podpis-zaufany).
* [Certyfikat KSeF](certyfikaty-KSeF.md) – wystawiany przez system KSeF. Certyfikat ten nie jest certyfikatem kwalifikowanym, ale jest honorowany w procesie uwierzytelniania. Certyfikat KSeF jest wyłącznie wykorzystywany na potrzeby systemu KSeF.
* Certyfikat dostawcy usług Peppol - zawierający identyfikator dostawcy.

Na środowisku testowym dopuszcza się użycie samodzielnie wygenerowanego certyfikatu będącego odpowiednikiem certyfikatów kwalifikowanych, co umożliwia wygodne testowanie podpisu bez potrzeby posiadania certyfikatu kwalifikowanego.

Biblioteka KSeF Client ([csharp]((https://github.com/CIRFMF/ksef-client-csharp)), [java]((https://github.com/CIRFMF/ksef-client-java))) posiada funkcjonalność składania podpisu cyfrowego w formacie XAdES.

Po podpisaniu dokumentu XML powinien on zostać przesłany do systemu KSeF w celu uzyskania tymczasowego tokena (```authenticationToken```).

Szczegółowe informacje na temat obsługiwanych formatów podpisu XAdES oraz wymagań dotyczących atrybutów certyfikatów kwalifikowanych znajdują się [tutaj](auth/podpis-xades.md).

Przykład w języku ```C#```:

Wygenerowanie testowego certyfikatu (możliwego do użycia tylko na środowisku testowym) osoby fizycznej z przykładowymi identyfikatorami:
[KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
X509Certificate2 ownerCertificate = CertificateUtils.GetPersonalCertificate("Jan", "Kowalski", "TINPL", ownerNip, "M B");

//CertificateUtils.GetPersonalCertificate
public static X509Certificate2 GetPersonalCertificate(
    string givenName,
    string surname,
    string serialNumberPrefix,
    string serialNumber,
    string commonName,
    EncryptionMethodEnum encryptionType = EncryptionMethodEnum.Rsa)
{
    X509Certificate2 certificate = SelfSignedCertificateForSignatureBuilder
                .Create()
                .WithGivenName(givenName)
                .WithSurname(surname)
                .WithSerialNumber($"{serialNumberPrefix}-{serialNumber}")
                .WithCommonName(commonName)
                .AndEncryptionType(encryptionType)
                .Build();
    return certificate;
}
```
Wygenerowanie testowego certyfikatu (możliwego do użycia tylko na środowisku testowym) organizacji z przykładowymi identyfikatorami:

```csharp
// Odpowiednik certyfikatu kwalifikowanego organizacji (tzw. pieczęć firmowa)
X509Certificate2 euEntitySealCertificate = CertificateUtils.GetCompanySeal("Kowalski sp. z o.o", euEntityNipVatEu, "Kowalski");

//CertificateUtils.GetCompanySeal
public static X509Certificate2 GetCompanySeal(
    string organizationName,
    string organizationIdentifier,
    string commonName)
{
    X509Certificate2 certificate = SelfSignedCertificateForSealBuilder
                .Create()
                .WithOrganizationName(organizationName)
                .WithOrganizationIdentifier(organizationIdentifier)
                .WithCommonName(commonName)
                .Build();
    return certificate;
}
```

Używając ```ISignatureService``` oraz posiadając certyfikat z kluczem prywatnym do podpisania dokumentu:
```csharp
string unsignedXml = AuthTokenRequestSerializer.SerializeToXmlString(authTokenRequest);

string signedXml = await signatureService.SignAsync(unsignedXml, certificate);
```

Przykład w języku ```Java```:
[BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

Wygenerowanie testowego certyfikatu (możliwego do użycia tylko na środowisku testowym) organizacji z przykładowymi identyfikatorami

Dla organizacji

```java
X500Name x500 = new CertificateBuilders()
    .buildForOrganization("Kowalski sp. z o.o", "VATPL-" + subject, "Kowalski", "PL");

SelfSignedCertificate cert = certificateService.generateSelfSignedCertificateRsa(x500);
```

Lub dla osoby prywatnej

```java
X500Name x500 = new CertificateBuilders()
        .buildForPerson("Jan", "Kowalski", context, "Kowalski");

SelfSignedCertificate cert = certificateService.generateSelfSignedCertificateRsa(x500);
```

Używając SignatureService oraz posiadając certyfikat z kluczem prywatnym można podpisać dokument

```java
String xml = AuthTokenRequestSerializer.authTokenRequestSerializer(authTokenRequest);

String signedXml = signatureService.sign(xml.getBytes(), cert.certificate(), cert.getPrivateKey());
```

#### 3. Wysłanie podpisanego XML

Po podpisaniu dokumentu AuthTokenRequest należy przesłać go do systemu KSeF za pomocą wywołania endpointu <br>
POST [/auth/xades-signature](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1xades-signature/post). <br>
Ponieważ proces uwierzytelniania jest asynchroniczny, w odpowiedzi zwracany jest tymczasowy token operacji uwierzytelnienia (JWT) (```authenticationToken```) wraz z numerem referencyjnym (```referenceNumber```). Oba identyfikatory służą do:
* sprawdzenia statusu procesu uwierzytelnienia,
* pobrania właściwego tokena dostępowego (`accessToken`) w formacie JWT.


Przykład w języku ```C#```:

```csharp
SignatureResponse authOperationInfo = await ksefClient.SubmitXadesAuthRequestAsync(signedXml, verifyCertificateChain: false);
```

Przykład w języku ```Java```:
[BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
SignatureResponse submitAuthTokenResponse = createKSeFClient().submitAuthTokenRequest(signedXml, false);
```

### 2.2. Uwierzytelnianie **tokenem KSeF**
Wariant uwierzytelniania tokenem KSeF wymaga przesyłania **zaszyfrowanego ciągu** złożonego z tokena KSeF oraz znacznika czasu otrzymanego w challenge. Token stanowi właściwy sekret uwierzytelniający, natomiast znacznik czasu pełni rolę nonce (IV), zapewniając świeżość operacji i uniemożliwiając odtworzenie szyfrogramu w kolejnych sesjach.

#### 1. Przygotowanie i szyfrowanie tokena
Łańcuch znaków w formacie:
```csharp
{tokenKSeF}|{timestamp_z_challenge}
```
należy zaszyfrować kluczem publicznym KSeF, wykorzystując algorytm ```RSA-OAEP``` z funkcją skrótu ```SHA-256 (MGF1)```. Otrzymany szyfrogram należy zakodować w ```Base64```.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
AuthChallengeResponse challenge = await KsefClient.GetAuthChallengeAsync();
long timestampMs = challenge.Timestamp.ToUnixTimeMilliseconds();

// Przygotuj "token|timestamp" i zaszyfruj RSA-OAEP SHA-256 zgodnie z wymaganiem API
string tokenWithTimestamp = $"{ksefToken}|{timestampMs}";
byte[] tokenBytes = System.Text.Encoding.UTF8.GetBytes(tokenWithTimestamp);
byte[] encrypted = CryptographyService.EncryptKsefTokenWithRSAUsingPublicKey(tokenBytes);
string encryptedTokenB64 = Convert.ToBase64String(encrypted);
```

Przykład w języku ```Java```:
[KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
AuthenticationChallengeResponse challenge = ksefClient.getAuthChallenge();
String ksefToken = $otrzymanyToken;
byte[] encryptedToken;
encryptedToken = new DefaultCryptographyService(createKSeFClient())
    .encryptKsefTokenWithRSAUsingPublicKey(ksefToken.getToken(), challenge.getTimestamp());
```

#### 2. Wysłanie żądania uwierzytelnienia [tokenem KSeF](tokeny-ksef.md)
Zaszyfrowany token Ksef należy przesłać razem z

|    Klucz     |           Wartość                                                                                                                              |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| Challenge    | `Wartość otrzymana z wywołania /auth/challenge`                                                                                                          |
| Context| `Identyfikator kontekstu, dla którego realizowane jest uwierzytelnienie (NIP, identyfikator wewnętrzny, identyfikator złożony VAT UE)`                                                                       |
| (opcjonalnie) AuthorizationPolicy | `Reguły dotyczące walidacji adresu IP klienta podczas korzystania z wydanego tokena dostępu (accessTokena).` |  

za pomocą wywołania endpointu:

POST [/auth/ksef-token](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1ksef-token/post). <br>

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
// Sposób 1: Budowa zapytania za pomocą buildera
var builder = AuthKsefTokenRequestBuilder
    .Create()
    .WithChallenge(challenge)
    .WithContext(contextIdentifierType, contextIdentifierValue)
    .WithEncryptedToken(encryptedToken);   
var authKsefTokenRequest = builder.Build();

// Sposób 2: manualne tworzenie obiektu
AuthenticationKsefTokenRequest request = new AuthenticationKsefTokenRequest
{
    Challenge = challenge.Challenge,
    ContextIdentifier = new AuthenticationTokenContextIdentifier
    {
        Type = AuthenticationTokenContextIdentifierType.Nip,
        Value = nip
    },
    EncryptedToken = encryptedTokenB64,
    AuthorizationPolicy = null
};

SignatureResponse signature = await KsefClient.SubmitKsefTokenAuthRequestAsync(request, CancellationToken);
```

Przykład w języku ```Java```:
[KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
AuthKsefTokenRequest authTokenRequest = new AuthKsefTokenRequestBuilder()
        .withChallenge(challenge.getChallenge())
        .withContextIdentifier(new ContextIdentifier(ContextIdentifierType.NIP, contextNip))
        .withEncryptedToken(Base64.getEncoder().encodeToString(encryptedToken))
        .build();

        SignatureResponse response = createKSeFClient().authenticateByKSeFToken(authTokenRequest);
```

Ponieważ proces uwierzytelniania jest asynchroniczny, w odpowiedzi zwracany jest tymczasowy token operacyjny (```authenticationToken```) wraz z numerem referencyjnym (```referenceNumber```). Oba identyfikatory służą do:
* sprawdzenia statusu procesu uwierzytelnienia,
* pobrania właściwego tokena dostępowego (accessToken) w formacie JWT.

### 3. Sprawdzenie statusu uwierzytelniania

Po przesłaniu podpisanego dokumentu XML (```AuthTokenRequest```) i otrzymaniu odpowiedzi zawierającej ```authenticationToken``` oraz ```referenceNumber```, należy sprawdzić status trwającej operacji uwierzytelnienia podając w nagłówku ```Authorization``` Bearer \<authenticationToken\>. <br>
GET [/auth/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1%7BreferenceNumber%7D/get)
W odpowiedzi zwracany jest status – kod i opis stanu operacji (np. "Uwierzytelnianie w toku", Uwierzytelnianie "zakończone sukcesem").

**Uwaga**  
Na środowiskach przedprodukcyjnym i produkcyjnym system, oprócz poprawności podpisu XAdES, sprawdza aktualny status certyfikatu u jego wystawcy (usługi OCSP/CRL). Do czasu uzyskania wiążącej odpowiedzi od dostawcy certyfikatu status operacji będzie zwracał "Uwierzytelnianie w toku" - jest to normalna konsekwencja procesu weryfikacji i nie oznacza błędu systemu. Sprawdzanie statusu jest asynchroniczne; wynik należy odpytywać do skutku. Czas weryfikacji zależy od wydawcy certyfikatu.

**Rekomendacja dla środowiska produkcyjnego - certyfikat KSeF**  
Aby wyeliminować oczekiwanie na weryfikację statusu certyfikatu w usługach OCSP/CRL po stronie kwalifikowanych dostawców usług zaufania, zaleca się uwierzytelnianie [certyfikatem KSeF](certyfikaty-KSeF.md). Weryfikacja certyfikatu KSeF odbywa się wewnątrz systemu i następuje niezwłocznie po odbiorze podpisu.

**Obsługa błędów**  
W przypadku niepowodzenia, w odpowiedzi mogą pojawić się kody błędów związane z niepoprawnym podpisem, brakiem uprawnień lub problemami technicznymi. Szczegółowa lista kodów błędów będzie dostępna w dokumentacji technicznej endpointa.

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
AuthStatus status;
status = await KsefClient.GetAuthStatusAsync(signature.ReferenceNumber, signature.AuthenticationToken.Token);
```

Przykład w języku ```Java```:
[BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthStatus authStatus = createKSeFClient().getAuthStatus(referenceNumber, tempToken);
```

### 4. Uzyskanie tokena dostępowego (accessToken)
Endpoint zwraca jednorazowo parę tokenów wygenerowanych dla pomyślnie zakończonego procesu uwierzytelniania. Każde kolejne wywołanie z tym samym ```authenticationToken``` zwróci błąd 400.

POST [/auth/token/redeem](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1token~1redeem/post)

Przykład w języku ```C#```:
[KSeF.Client.Tests.Core\E2E\KsefToken\KsefTokenE2ETests.cs](https://github.com/CIRFMF/ksef-client-csharp/blob/main/KSeF.Client.Tests.Core/E2E/KsefToken/KsefTokenE2ETests.cs)

```csharp
AuthOperationStatusResponse tokens = await KsefClient.GetAccessTokenAsync(signature.AuthenticationToken.Token);
```

Przykład w języku ```Java```:
[BaseIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/BaseIntegrationTest.java)

```java
AuthOperationStatusResponse tokenResponse = createKSeFClient().redeemToken(response.getAuthenticationToken().getToken());
```

W odpowiedzi zwracane są:
* ```accessToken``` – token dostępowy JWT służący do autoryzacji operacji w API (w nagłówku Authorization: Bearer ...), ma ograniczony czas ważności (np. kilkanaście minut, określony w polu exp),
* ```refreshToken``` – token umożliwiający odświeżenie ```accessToken``` bez ponownego uwierzytelnienia, ma znacznie dłuższy okres ważności (do 7 dni) i może być używany wielokrotnie do odświeżania tokena dostępowego.

**Uwaga!**
1. ```accessToken``` oraz ```refreshToken``` powinien być traktowany jak dane poufne – jego przechowywanie wymaga odpowiednich zabezpieczeń.
2. Token dostępu (`accessToken`) pozostaje ważny aż do upływu daty określonej w polu `exp`, nawet jeśli uprawnienia użytkownika ulegną zmianie.

#### 5. Odświeżenie tokena dostępowego (```accessToken```)
W celu utrzymania ciągłego dostępu do chronionych zasobów API, system KSeF udostępnia mechanizm odświeżania tokena dostępowego (```accessToken```) przy użyciu specjalnego tokena odświeżającego (```refreshToken```). Rozwiązanie to eliminuje konieczność każdorazowego ponawiania pełnego procesu uwierzytelnienia, ale również poprawia bezpieczeństwo systemu – krótki czas życia ```accessToken``` ogranicza ryzyko jego nieautoryzowanego użycia w przypadku przechwycenia.

POST [/auth/token/refresh](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Uwierzytelnianie/paths/~1api~1v2~1auth~1token~1refresh/post) <br>
```RefreshToken``` należy przekazać w nagłówku Authorization w formacie:
```
Authorization: Bearer {refreshToken}
```

Odpowiedź zawiera nowy ```accessToken``` (JWT) z aktualnym zestawem uprawnień i ról.

 Przykład w języku ```C#```:

```csharp
RefreshTokenResponse refreshedAccessTokenResponse = await ksefClient.RefreshAccessTokenAsync(accessTokenResult.RefreshToken.Token);
```

Przykład w języku ```Java```:
[AuthorizationIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/AuthorizationIntegrationTest.java)

```java
AuthenticationTokenRefreshResponse refreshTokenResult = createKSeFClient().refreshAccessToken(token.refreshToken());
```

#### 6. Zarządzanie sesjami uwierzytelniania 
Szczegółowe informacje o zarządzaniu aktywnymi sesjami uwierzytelniania znajdują się w dokumencie [Zarządzanie sesjami](auth/sesje.md).

Powiązane dokumenty: 
- [Certyfikaty KSeF](certyfikaty-KSeF.md)
- [Testowe certyfikaty i podpisy XAdES](auth/testowe-certyfikaty-i-podpisy-xades.md)
- [Podpis XAdES](auth/podpis-xades.md)
- [Token KSeF](tokeny-ksef.md)
