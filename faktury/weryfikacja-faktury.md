# Weryfikacja faktury
29.07.2025

Faktura przesyłana do systemu KSeF podlega szeregowi kontroli technicznych i semantycznych. Weryfikacja obejmuje następujące kryteria:

## Zgodność ze schematem XSD
Faktura musi być przygotowana w formacie XML, kodowana w UTF-8 oraz zgodna z aktualnym schematem opublikowanym przez Ministerstwo Finansów.

## Unikalność faktury
- System wykrywa duplikaty na podstawie trzech pól:
  1. NIP sprzedawcy (`Podmiot1:NIP`)
  2. Rodzaj faktury (`RodzajFaktury`)
  3. Numer faktury (`P_2`)
- W przypadku duplikatu zwracany jest kod błędu 440 („Duplikat faktury”).

## Walidacja dat
Data wystawienia faktury (`P_1`) nie może być późniejsza niż data przyjęcia dokumentu do systemu KSeF.

## Rozmiar pliku
- Maksymalny rozmiar faktury bez załączników: **1 MB**
- Maksymalny rozmiar faktury z załącznikami: **3 MB**

## Ograniczenia ilościowe
Maksymalna liczba faktur w jednej sesji (zarówno interaktywnej, jak i wsadowej) wynosi 10 000 000.

## Poprawne szyfrowanie
- Faktura powinna być zaszyfrowana algorytmem AES-256-CBC (klucz symetryczny 256 bit, IV 128 bit, PKCS#7).
- Klucz symetryczny szyfrowany algorytmem RSAES-OAEP (SHA-256/MGF1).

## Zgodność metadanych faktury w sesji interaktywnej
- Obliczenie i weryfikacja skrótu faktury wraz z rozmiarem pliku.
- Obliczenie i weryfikacja skrótu zaszyfrowanej faktury wraz z rozmiarem pliku.

## Ograniczenia dotyczące załączników
- Wysyłka faktur z załącznikami jest dozwolona tylko w trybie wsadowym.
- Możliwość wysyłki faktur z załącznikami wymaga uprzedniego zgłoszenia tej opcji w usłudze `e-Urząd Skarbowy`.

## Wymagania dotyczące uprawnień
Wysłanie faktury do KSeF wymaga posiadania odpowiednich uprawnień do jej wystawienia w kontekście danego podmiotu.