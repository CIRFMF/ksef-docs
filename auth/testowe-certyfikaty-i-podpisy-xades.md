# Testowe certyfikaty i podpisy XAdES

Ten przewodnik pokazuje, jak **szybko uruchomić** konsolową aplikację demonstracyjną [`KSeF.Client.Tests.CertTestApp`](https://github.com/CIRFMF/ksef-client-csharp) w celu:
- wygenerowania **testowego (self‑signed) certyfikatu** na potrzeby środowiska testowego KSeF,
- zbudowania i **podpisania XAdES** dokumentu `AuthTokenRequest`,
- wysłania podpisanego dokumentu do KSeF i **pozyskania tokenów** dostępowych (JWT).

> **Uwaga**
> - Samopodpisane certyfikaty są **dozwolone wyłącznie** na środowisku **testowym**.
> - Dane w przykładach (NIP, numer referencyjny, tokeny) są **fikcyjne** i służą wyłącznie demonstracji.

---

## Wymagania wstępne
- **.NET 9 SDK**
- Git
- Windows lub Linux

---

## Co robi aplikacja?
- Pobiera **challenge** (wyzwanie) z KSeF.
- Buduje dokument XML `AuthTokenRequest`.
- **Podpisuje** dokument `AuthTokenRequest` w formacie **XAdES**.
- Wysyła podpisany dokument do KSeF i otrzymuje `referenceNumber` + `authenticationToken`.
- **Odpytuje status** operacji uwierzytelnienia do skutku.
- Po sukcesie pobiera parę tokenów: `accessToken` i `refreshToken` (JWT).
- Zapisuje artefakty (m.in. **certyfikat testowy** oraz **podpisany XML**) do plików, jeśli wybrano wyjście `file`.

---

## Windows

1. **Zainstaluj .NET 9 SDK**:
   ```powershell
   winget install Microsoft.DotNet.SDK.9
   ```
   Alternatywnie: pobierz instalator z witryny .NET.

2. **Otwórz nowe okno terminala** (PowerShell/CMD).

3. **Sprawdź instalację**:
   ```powershell
   dotnet --version
   ```
   Oczekiwany numer wersji: `9.x.x`.

4. **Sklonuj repozytorium i przejdź do projektu**:
   ```powershell
   git clone https://github.com/CIRFMF/ksef-client-csharp.git
   cd ksef-client-csharp/KSeF.Client.Tests.CertTestApp
   ```

5. **Uruchom (domyślnie losowy NIP, wynik na ekranie)**:
   ```powershell
   dotnet run --framework net9.0
   ```

6. **Uruchomienie z parametrami**:
   - `--output` – `screen` (domyślnie) lub `file` (zapis wyników do plików),
   - `--nip` {numer_nip} - np. `--nip 8976111986`,
   - opcjonalnie: `--no-startup-warnings`.

   ```powershell
   dotnet run --framework net9.0 --output file --nip 8976111986 --no-startup-warnings
   ```

---

## Linux (Ubuntu/Debian)

1. **Dodaj repozytorium Microsoft i zaktualizuj pakiety**:
   ```bash
   wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
   sudo dpkg -i packages-microsoft-prod.deb
   sudo apt-get update
   ```

2. **Zainstaluj .NET 9 SDK**:
   ```bash
   sudo apt-get install -y dotnet-sdk-9.0
   ```

3. **Odśwież środowisko powłoki lub otwórz nowy terminal**:
   ```bash
   source ~/.bashrc
   ```

4. **Sprawdź instalację**:
   ```bash
   dotnet --version
   ```
   Oczekiwany numer wersji: `9.x.x`.

5. **Sklonuj repozytorium i przejdź do projektu**:
   ```bash
   git clone https://github.com/CIRFMF/ksef-client-csharp.git
   cd ksef-client-csharp/KSeF.Client.Tests.CertTestApp
   ```

6. **Uruchom (wynik na ekranie, losowy NIP)**:
   ```bash
   dotnet run --framework net9.0
   ```

7. **Uruchomienie z parametrami**:
   - `--output` – `screen` (domyślnie) lub `file` (zapis wyników do plików),
   - `--nip` {numer_nip} - np. `--nip 8976111986`,
   - opcjonalnie: `--no-startup-warnings`.

   ```bash
   dotnet run --framework net9.0 --output file --nip 8976111986 --no-startup-warnings
   ```

---

Powiązane dokumenty: 
- [Uwierzytelnianie w KSeF](../uwierzytelnianie.md)
- [Podpis XAdES](podpis-xades.md)

---