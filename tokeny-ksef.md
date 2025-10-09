## Zarządzanie tokenami KSeF
29.06.2025

Token KSeF to unikalny, generowany identyfikator uwierzytelniający, który — na równi z [kwalifikowanym podpisem elektronicznym](uwierzytelnianie.md#21-uwierzytelnianie-kwalifikowanym-podpisem-elektronicznym) — umożliwia [uwierzytelnienie](uwierzytelnianie.md#22-uwierzytelnianie-tokenem-ksef) się do API KSeF.

```Token KSeF``` jest wydawany z niezmiennym zestawem uprawnień określonych przy jego tworzeniu; każda modyfikacja tych uprawnień wymaga wygenerowania nowego tokena.
> **Uwaga!** <br>
> ```Token KSeF``` pełni rolę **poufnego sekretu** uwierzytelniającego — należy przechowywać go wyłącznie w zaufanym i bezpiecznym magazynie.


### Wymagania wstępne

Wygenerowanie ```tokena KSeF``` jest możliwe wyłącznie po jednorazowym uwierzytelnieniu się [podpisem elektronicznym (XAdES)](uwierzytelnianie.md#21-uwierzytelnianie-kwalifikowanym-podpisem-elektronicznym).

### 1. Generowanie tokenu  

Token może być generowany wyłącznie w kontekście `Nip` lub `InternalId`. Generowanie odbywa się poprzez wywołanie endpointu:  
POST [/tokens](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens/post)

Podając w ciele żądania kolekcję uprawnień oraz opis tokena.

 **Przykłady implementacji:** <br>

| Pole        | Przykładowa wartość                         | Opis                                       |
|-------------|---------------------------------------------|--------------------------------------------|
| Permissions | `["InvoiceRead", "InvoiceWrite", "CredentialsRead", "CredentialsManage"]`        | Lista uprawnień przypisanych tokenowi      |
| Description | `"Token do odczytu faktur i danych konta"` | Opis tokena                                 |


Przykład w języku C#:
```csharp
 var tokenRequest = new KsefTokenRequest
    {
        Permissions = [
            KsefTokenPermissionType.InvoiceRead,
            KsefTokenPermissionType.InvoiceWrite
            ],
        Description = "Demo token",
    };
 var token = await ksefClient.GenerateKsefTokenAsync(tokenRequest, accessToken, cancellationToken);
```

Przykład w języku Java:
[KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/configuration/KsefTokenIntegrationTest.java)

```java
KsefTokenRequest request = new KsefTokenRequestBuilder()
        .withDescription("test description")
        .withPermissions(List.of(TokenPermissionType.INVOICE_READ, TokenPermissionType.INVOICE_WRITE))
        .build();
GenerateTokenResponse ksefToken = ksefClient.generateKsefToken(request, authToken.accessToken());
```

### 2. Filtrowanie tokenów

Metadane tokenów KSeF można pobierać i filtrować za pomocą wywołania:<br>
GET [/tokens](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens/get)

Przykład w języku C#:
```csharp
var result = new List<AuthenticationKsefToken>();
const int pageSize = 20;
var status = AuthenticationKsefTokenStatus.Active;
var continuationToken = string.Empty;

do
 {
     var tokens = await ksefClient.QueryKsefTokensAsync(accessToken, [status], continuationToken, pageSize, cancellationToken);
     result.AddRange(tokens.Tokens);
     continuationToken = tokens.ContinuationToken;
 } while (!string.IsNullOrEmpty(continuationToken));
```

Przykład w języku Java:
[KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
List<AuthenticationTokenStatus> status = List.of(AuthenticationTokenStatus.ACTIVE);
Integer pageSize = 10;
QueryTokensResponse tokens = ksefClient.queryKsefTokens(status, StringUtils.EMPTY, null, null, null, pageSize, accessToken);
```

W odpowiedzi zwracane są metadane tokenów, między innymi informacja kto i w jakim kontekœcie wygenerował token KSef oraz uprawnienia do niego przypisane.

### 3. Pobieranie konkretnego tokena

Aby pobrać szczegóły konkretnego token, należy użyć wywołania:<br>
GET [/tokens/\{referenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens~1%7BreferenceNumber%7D/get)

```referenceNumber``` to unikalny identyfikator tokena, który można uzyskać podczas jego tworzenia lub z listy tokenów.

Przykład w języku C#:
```csharp
var token = await ksefClient.GetKsefTokenAsync(referenceNumber, accessToken, cancellationToken);
```
Przykład w języku Java:
[KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
AuthenticationToken ksefToken = ksefClient.getKsefToken(token.getReferenceNumber(), accessToken);
```

### 4. Unieważnienie tokena

Aby unieważnić token, należy użyć wywołania:<br>
DELETE [/tokens/\{referenceNumber\}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens~1%7BreferenceNumber%7D/delete)

```referenceNumber``` to unikalny identyfikator tokena, który chcemy unieważnić.

Przykład w języku C#:
```csharp
await ksefClient.RevokeKsefTokenAsync(referenceNumber, accessToken, cancellationToken);
```

Przykład w języku Java:
[KsefTokenIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/KsefTokenIntegrationTest.java)

```java
ksefClient.revokeKsefToken(token.getReferenceNumber(), accessToken);
```
