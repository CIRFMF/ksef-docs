## Zarządzanie sesjami uwierzytelniania
10.07.2025

### Pobranie listy aktywnych sesji uwierzytelniania

Zwraca listę aktywnych sesji uwierzytelnienia.

GET [/auth/sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions/get)

Przykład w języku ```C#```:
```csharp
const int pageSize = 20;
string? continuationToken = null;
var activeSessions = new List<Item>();
do
{
    var response = await ksefClient.GetActiveSessions(accessToken, pageSize, continuationToken, cancellationToken);
    continuationToken = response.ContinuationToken;
    activeSessions.AddRange(response.Items);
}
while (!string.IsNullOrWhiteSpace(continuationToken));
```

Przykład w języku ```Java```:
[SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
int pageSize = 10;
AuthenticationListResponse activeSessions = createKSeFClient().getActiveSessions(10, null, accessToken);
while (Strings.isNotBlank(activeSessions.getContinuationToken())) {
    activeSessions = createKSeFClient().getActiveSessions(10, activeSessions.getContinuationToken(), accessToken);
}
```

### Unieważnienie bieżącej sesji

DELETE [`/auth/sessions/current`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions~1current/delete)

Unieważnia sesję związaną z tokenem użytym do wywołania tego endpointu. Po operacji:
- powiązany ```refreshToken``` zostaje unieważniony,
- aktywne ```accessTokeny``` pozostają ważne do upływu ich terminu ważności.

Przykład w języku ```C#```:
```csharp
await ksefClient.RevokeCurrentSessionAsync(token, cancellationToken);
```

Przykład w języku ```Java```:
[SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
createKSeFClient().revokeCurrentSession(accessToken);
```

### Unieważnienie wybranej sesji

DELETE [`/auth/sessions/{referenceNumber}`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions~1%7BreferenceNumber%7D/delete)

Unieważnia sesję o wskazanym numerze referencyjnym. Po operacji:
- powiązany ```refreshToken``` zostaje unieważniony,
- aktywne ```accessTokeny``` pozostają ważne do upływu ich terminu ważności.

Przykład w języku ```C#```:
```csharp
await ksefClient.RevokeSessionAsync(referenceNumber, accessToken, cancellationToken);
```

Przykład w języku ```Java```:
[SessionIntegrationTest.java](https://github.com/CIRFMF/ksef-client-java/blob/main/demo-web-app/src/integrationTest/java/pl/akmf/ksef/sdk/SessionIntegrationTest.java)

```java
createKSeFClient().revokeSession(secondSessionReferenceNumber, firstAccessTokensPair.accessToken());
```
