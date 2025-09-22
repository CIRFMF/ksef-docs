# Weryfikacja faktury
22.09.2025

Faktura przesyłana do systemu KSeF podlega szeregowi kontroli technicznych i semantycznych. Weryfikacja obejmuje następujące kryteria:

## Zgodność ze schematem XSD
Faktura musi być przygotowana w formacie XML, kodowana w UTF-8 oraz zgodna z aktualnym schematem opublikowanym przez Ministerstwo Finansów.

## Unikalność faktury
- KSeF wykrywa duplikaty faktur globalnie, w oparciu o dane przechowywane w systemie. Kryterium identyfikacji duplikatu stanowi kombinacja:
  1. NIP sprzedawcy (`Podmiot1:NIP`)
  2. Rodzaj faktury (`RodzajFaktury`)
  3. Numer faktury (`P_2`)
- W przypadku duplikatu zwracany jest kod błędu 440 („Duplikat faktury”). 
- Unikalność faktury utrzymywana jest w KSeF przez okres 10 pełnych lat, liczonych od końca roku kalendarzowego, w którym faktura została wystawiona.
- Kryterium unikalności odnosi się zawsze do sprzedawcy (Podmiot1:NIP). W przypadku, gdy w imieniu tego samego podmiotu faktury wystawiają różne jednostki (np. oddziały, jednostki organizacyjne JST, inne uprawnione podmioty), muszą one uzgodnić zasady numeracji, aby uniknąć duplikatów.

## Walidacja dat
Data wystawienia faktury (`P_1`) nie może być późniejsza niż data przyjęcia dokumentu do systemu KSeF.

## Rozmiar pliku
- Maksymalny rozmiar faktury bez załączników: **1 MiB** (1 048 576 bajtów).
- Maksymalny rozmiar faktury z załącznikami: **3 MiB** (3 145 728 bajtów).

## Ograniczenia ilościowe
- Maksymalna liczba faktur w jednej sesji (zarówno interaktywnej, jak i wsadowej) wynosi 10 000 000.
- W ramach wysyłki wsadowej można przesłać maksymalnie 50 plików ZIP; rozmiar każdego pliku nie może przekroczyć 100 MiB (104 857 600 bajtów), a łączny rozmiar paczki - 5 GiB (5 368 709 120 bajtów).

## Poprawne szyfrowanie
- Faktura powinna być zaszyfrowana algorytmem AES-256-CBC (klucz symetryczny 256 bit, IV 128 bit, PKCS#7).
- Klucz symetryczny szyfrowany algorytmem RSAES-OAEP (SHA-256/MGF1).

## Zgodność metadanych faktury w sesji interaktywnej
- Obliczenie i weryfikacja skrótu faktury wraz z rozmiarem pliku.
- Obliczenie i weryfikacja skrótu zaszyfrowanej faktury wraz z rozmiarem pliku.

## Ograniczenia dotyczące załączników
- Wysyłka faktur z załącznikami jest dozwolona tylko w trybie wsadowym.  
**Wyjątek:** W przypadku przesyłania [korekty technicznej faktury offline](../offline/korekta-techniczna.md) dopuszczalne jest użycie sesji interaktywnej.
- Możliwość wysyłki faktur z załącznikami wymaga uprzedniego zgłoszenia tej opcji w usłudze `e-Urząd Skarbowy`.



## Wymagania dotyczące uprawnień
Wysłanie faktury do KSeF wymaga posiadania odpowiednich uprawnień do jej wystawienia w kontekście danego podmiotu.