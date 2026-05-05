# Klucze publiczne do szyfrowania

Dokument opisuje, w jaki sposób KSeF publikuje **klucze publiczne wykorzystywane do szyfrowania po stronie klienta**, jak przebiegają procesy **rotacji** oraz jak poprawnie obsłużyć scenariusz przełączenia na nowy klucz.

## 1. Wprowadzenie

W wybranych operacjach API wymagających szyfrowania po stronie klienta przekazywane dane są szyfrowane przy użyciu **klucza publicznego** KSeF, udostępnianego w postaci certyfikatu `X.509`.

## 2. Udostępnianie kluczy publicznych

Certyfikaty kluczy publicznych udostępniane są za pomocą endpointu:

- `GET /security/public-key-certificates`

Certyfikat `X.509` publikowany przez API KSeF jest wydany przez kwalifikowane centrum certyfikacji (CA).
W polu `Subject` certyfikatu wskazany jest podmiot, którego dotyczy klucz publiczny, w szczególności: `CN = Ministerstwo Finansów`.

Odpowiedź zawiera:

- `certificate` - certyfikat `X.509` w formacie `DER` (binarnym), zakodowany w `Base64` (bez nagłówków BEGIN/END).
- `certificateId` - identyfikator certyfikatu (skrót **SHA-256** z DER certyfikatu zakodowany w formacie `Base64`).
- `publicKeyId` - identyfikator klucza (skrót **SHA-256** z DER klucza publicznego (`SubjectPublicKeyInfo`) zawartego w certyfikacie, zakodowany w formacie `Base64`), używany jako selektor w wywołaniach, w których klient wskazuje, **jakiego klucza publicznego użył do szyfrowania**.
- `validFrom`, `validTo` - okres ważności certyfikatu.
- `usage` - przeznaczenie klucza (np. `KsefTokenEncryption`, `SymmetricKeyEncryption`).

Przykładowa odpowiedź:

```json
[
  {
    "certificate": "MIIGWDCCBECgAwIBAgIQGmXqNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQGEwJQ...",
    "certificateId": "1ehGlGObVNNm4OE6nIF879NoCIAYbwP0AZQDPJ4+tSQ=",
    "publicKeyId": "bCLIk+crwFEhRWdsWVy/MqwpR/8KiEmr+2cFZf1bPv0=",    
    "validFrom": "2025-09-29T06:03:19+00:00",
    "validTo": "2027-09-29T06:03:18+00:00",
    "usage": ["KsefTokenEncryption"]
  },
  {
    "certificate": "MIIGWDCCBECgAwIBADADADNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQD2D2...",
    "certificateId": "p7TsAeUHK/v5sPmPwTX+Dodxe3E2TQ/G0c5vlHbqri4=",
    "publicKeyId": "P2GDBhdfCxZMbmCXebZOsWr8pBpXeFBwD2qusHd3WZs=",
    "validFrom": "2025-09-29T06:17:45+00:00",
    "validTo": "2027-09-29T06:17:44+00:00",
    "usage": ["SymmetricKeyEncryption"]
  }
]
```

## 3. Rotacja certyfikatów i kluczy

W API KSeF publikowany jest certyfikat `X.509` zawierający klucz publiczny.  
Publikacja nowego certyfikatu może wynikać z jednej z dwóch sytuacji:

1. **Re-certyfikacja** - publikowany jest nowy certyfikat dla dotychczasowej pary kluczy.
2. **Rotacja klucza** - generowana jest nowa para kluczy, a wraz z nią publikowany jest nowy certyfikat zawierający nowy klucz publiczny.  

### 3.1. Re-certyfikacja (ten sam klucz)

Re-certyfikacja polega na publikacji nowego certyfikatu `X.509` dla tego samego klucza publicznego.

W tym przypadku:

- para kluczy pozostaje bez zmian,
- `publicKeyId` pozostaje bez zmian,
- zmienia się certyfikat (`certificate`, `certificateId`, okres ważności).

Jest to scenariusz planowy, najczęściej związany z upływem okresu ważności certyfikatu.

#### Przykład

Przed re-certyfikacją `GET /security/public-key-certificates` zwraca:

```json
[
  {
    "certificate": "MIIGWDCCBECgAwIBADADADNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQD2D2...",
    "certificateId": "p7TsAeUHK/v5sPmPwTX+Dodxe3E2TQ/G0c5vlHbqri4=",
    "publicKeyId": "P2GDBhdfCxZMbmCXebZOsWr8pBpXeFBwD2qusHd3WZs=",
    "validFrom": "2025-09-29T06:17:45+00:00",
    "validTo": "2027-09-29T06:17:44+00:00",
    "usage": ["SymmetricKeyEncryption"]
  },
  ...
]
```

Po re-certyfikacji `GET /security/public-key-certificates` zwraca:

```json
[
  {
    "certificate": "MIIHGDCCBQCgAwIBAgIQBPt+Qi6c4aJSoufT3qmudDANBgkqhkiG9w0BAQsFADBnMQswCQYDVQQGE...",
    "certificateId": "4jvqWdpkjTMwORT2hrRMcmnnBAuGMR9UEw1aEO4mQMk=",
    "publicKeyId": "P2GDBhdfCxZMbmCXebZOsWr8pBpXeFBwD2qusHd3WZs=",
    "validFrom": "2027-05-14T06:12:41+00:00",
    "validTo": "2029-05-14T06:12:40+00:00",
    "usage": ["SymmetricKeyEncryption"]
  },
  ...
]
```

### 3.2. Rotacja klucza (nowy klucz publiczny)

Rotacja klucza oznacza wygenerowanie nowej pary kluczy (publicznego i prywatnego). Implementacje korzystające z publikowanych kluczy powinny być zawsze przygotowane na możliwość rotacji - również w trybie awaryjnym.

W standardowym trybie działania (przy braku incydentów bezpieczeństwa) nie należy oczekiwać częstych rotacji. Nie można jednak zakładać, że klucz pozostanie niezmienny przez cały okres funkcjonowania systemu.

W przypadku rotacji klucza:

- generowana jest nowa para kluczy,
- zmianie ulega `publicKeyId`,
- publikowany jest nowy certyfikat `X.509` zawierający nowy klucz publiczny.

Typowe powody rotacji klucza:

- podejrzenie kompromitacji lub incydent bezpieczeństwa,
- zmiana wymagań kryptograficznych (np. algorytm, parametry, rozmiar klucza),
- wymogi zgodności lub polityk bezpieczeństwa (cykliczna rotacja),
- decyzje operacyjne.

### 3.2.1. Rotacja planowa

Rotacja planowa wynika z przyjętej polityki bezpieczeństwa, wymogów zgodności lub cyklicznej wymiany kluczy.

W przypadku rotacji planowej nowy certyfikat może zostać opublikowany z wyprzedzeniem.  
W okresie przejściowym API będzie zwracać co najmniej dwa certyfikaty dla tego samego `usage`:
- dotychczasowy certyfikat,
- nowy certyfikat zawierający nowy klucz publiczny.

Po zakończeniu okresu przejściowego poprzedni certyfikat zostanie usunięty z listy.

#### Przykład - rotacja planowa

Przed rotacją `GET /security/public-key-certificates` zwraca:

```json
[
  {
    "certificate": "MIIGWDCCBECgAwIBADADADNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQD2D2...",
    "certificateId": "p7TsAeUHK/v5sPmPwTX+Dodxe3E2TQ/G0c5vlHbqri4=",
    "publicKeyId": "P2GDBhdfCxZMbmCXebZOsWr8pBpXeFBwD2qusHd3WZs=",
    "validFrom": "2025-09-29T06:17:45+00:00",
    "validTo": "2027-09-29T06:17:44+00:00",
    "usage": ["SymmetricKeyEncryption"]
  }
  ...
]
```

Po rotacji `GET /security/public-key-certificates` zwraca:

```json
[
  {
    "certificate": "MIIGWDCCBECgAwIBADADADNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQD2D2...",
    "certificateId": "p7TsAeUHK/v5sPmPwTX+Dodxe3E2TQ/G0c5vlHbqri4=",
    "publicKeyId": "P2GDBhdfCxZMbmCXebZOsWr8pBpXeFBwD2qusHd3WZs=",
    "validFrom": "2025-09-29T06:17:45+00:00",
    "validTo": "2027-09-29T06:17:44+00:00",
    "usage": ["SymmetricKeyEncryption"]
  },
  {
    "certificate": "MIIGWDCCBECgAwIBAgIQGmXqNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQGE...",
    "certificateId": "tQL/24cjRkqTRKf3LhwQcAudH2SgjgTcVGsWs89jqK4=",
    "publicKeyId": "0e3zYM0m1wT85iHZZt1J8QLvCpnN2t+RcFgHXkCh3xA=",
    "validFrom": "2027-03-14T06:12:41+00:00",
    "validTo": "2029-03-14T06:12:40+00:00",
    "usage": ["SymmetricKeyEncryption"]
  },
  ...
]
```

### 3.2.2. Rotacja awaryjna (incydent bezpieczeństwa)

Rotacja awaryjna może zostać przeprowadzona w przypadku podejrzenia kompromitacji klucza prywatnego lub innego incydentu bezpieczeństwa.

W takim przypadku:  
- klucz prywatny przestaje być używany do operacji kryptograficznych po stronie KSeF,
- powiązane z nim certyfikaty `X.509` zostają odwołane (revoked) u dostawcy certyfikatu (CA),
- certyfikaty powiązane z wycofanym kluczem przestają być zwracane przez endpoint `GET /security/public-key-certificates`,
- dotychczasowy `publicKeyId` przestaje być akceptowany przez API,
- publikowany jest nowy certyfikat zawierający nowy klucz publiczny.

### Przykład - rotacja awaryjna

Przed rotacją `GET /security/public-key-certificates` zwraca:

```json
[
  {
    "certificate": "MIIGWDCCBECgAwIBADADADNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQD2D2...",
    "certificateId": "p7TsAeUHK/v5sPmPwTX+Dodxe3E2TQ/G0c5vlHbqri4=",
    "publicKeyId": "P2GDBhdfCxZMbmCXebZOsWr8pBpXeFBwD2qusHd3WZs=",
    "validFrom": "2025-09-29T06:17:45+00:00",
    "validTo": "2027-09-29T06:17:44+00:00",
    "usage": ["SymmetricKeyEncryption"]
  },
  ...
]
```

Po rotacji `GET /security/public-key-certificates` zwraca:


```json
[
  {
    "certificate": "MIIGWDCCBECgAwIBAgIQGmXqNRg5ye1JMZDOQ7HNCTANBgkqhkiG9w0BAQsFADBOMQswCQYDVQQGE...",
    "certificateId": "tQL/24cjRkqTRKf3LhwQcAudH2SgjgTcVGsWs89jqK4=",
    "publicKeyId": "0e3zYM0m1wT85iHZZt1J8QLvCpnN2t+RcFgHXkCh3xA=",
    "validFrom": "2027-03-14T06:12:41+00:00",
    "validTo": "2029-03-14T06:12:40+00:00",
    "usage": ["SymmetricKeyEncryption"]
  },
  ...
]
```

## 4. Scenariusz użycia

### 4.1. Pobranie certyfikatów

1. Klient wywołuje:
   - `GET /security/public-key-certificates`
2. Klient wybiera certyfikat odpowiedni dla `usage` (np. `KsefTokenEncryption` albo `SymmetricKeyEncryption`), ważny w chwili operacji; w przypadku wielu ważnych certyfikatów preferuje certyfikat o najpóźniejszej dacie `validFrom`.
3. Klient szyfruje dane używając **klucza publicznego** z wybranego certyfikatu.

### 4.2. Przekazanie selektora `publicKeyId` do wybranych endpointów

`publicKeyId` jest przekazywany w modelu żądania do endpointów, dla których system musi jednoznacznie ustalić, którym kluczem publicznym klient wykonał szyfrowanie.

Dotyczy to:
- `POST /auth/ksef-token`
- `POST /sessions/online`
- `POST /sessions/batch`,
- `POST /invoices/exports`.

### 4.3. Walidacja po stronie KSeF

Endpoint waliduje: 
- czy `publicKeyId` jest znany dla danego `usage`,
- czy wskazany klucz jest aktualnie akceptowany (ważny i nie wycofany).

Jeśli klucz nie jest akceptowany, API zwraca błąd:
  - HTTP `400`
  - code: `21470` - Przesłany identyfikator klucza jest nieznany lub wskazuje na wycofany klucz.

### 4.4. Obsługa błędu 21470

Jeśli klient otrzyma błąd `21470`:  
1. Ponownie pobiera listę:
   - `GET /security/public-key-certificates`
2. Wybiera aktualny certyfikat dla danego `usage` (np. `KsefTokenEncryption` albo `SymmetricKeyEncryption`), ważny w chwili operacji; w przypadku wielu ważnych certyfikatów preferuje certyfikat o najpóźniejszej dacie `validFrom`.
3. Powtarza operację z użyciem nowego klucza (nowy `publicKeyId`).