# Numer KSeF – struktura i walidacja

Numer KSeF to unikalny identyfikator faktury nadawany przez system. Zawsze ma długość **35 znaków** i jest globalnie unikalny – jednoznacznie identyfikuje każdą fakturę w KSeF.

## Ogólna struktura numeru 
```
9999999999-RRRRMMDD-FFFFFFFFFFFF-FF  
```
Gdzie:
- `9999999999` – NIP sprzedawcy (10 cyfr),
- `RRRRMMDD` – data przyjęcia faktury (rok, miesiąc, dzień) do dalszego przetwarzania,
- `FFFFFFFFFFFF` – część techniczna składająca się z 12 znaków alfanumerycznych,
- `FF` – suma kontrolna (2 znaki, CRC-8 w formacie HEX, wielkie litery).

## Przykład
```
5265877635-20250826-0100001AF629-AF
```
- `5265877635` - NIP sprzedawcy,
- `20250826` - data przyjęcia faktury do dalszego przetwarzania,
- `0100001AF629` - część techniczna,
- `AF` - suma kontrolna CRC-8.

## Walidacja numeru KSeF

Proces walidacji obejmuje:
1. Sprawdzenie, czy numer ma **dokładnie 35 znaków**.  
2. Rozdzielenie części danych (32 znaki) i sumy kontrolnej (2 znaki).  
3. Obliczeniu sumy kontrolnej z części danych **algorytmem CRC-8**.  
4. Porównaniu obliczonej sumy z wartością znajdującą się w numerze.

## Algorytm CRC-8

Do obliczenia sumy kontrolnej stosowany jest algorytm **CRC-8** z parametrami:

- **Polinom:** `0x07`  
- **Wartość początkowa:** `0x00`  
- **Format wyniku:** 2-znakowy zapis szesnastkowy (HEX, wielkie litery)

Przykład: jeśli obliczona suma kontrolna wynosi `0x46`, do numeru KSeF zostanie dodane `"46"`.

## Przykład w języku C#:
```csharp
using KSeF.Client.Core;

bool isValid = KsefNumberValidator.IsValid(ksefNumber, out string message);
```

## Przykład w języku Java:
```java

```