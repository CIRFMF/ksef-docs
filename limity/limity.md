# Limity #
21.10.2025

## Wstęp ##

W systemie KSeF 2.0 zastosowano mechanizmy limitujące liczbę i wielkość operacji API oraz parametry związane z przesyłanymi danymi. Celem tych limitów jest:
- ochrona stabilności systemu przy dużej skali działania,
- przeciwdziałanie nadużyciom i nieefektywnym integracjom,
- zapobieganie nadużyciom i potencjalnym zagrożeniom cyberbezpieczeństwa,
- zapewnienie równych warunków dostępu dla wszystkich użytkowników.

Limity zostały zaprojektowane z możliwością elastycznego dostosowania do potrzeb konkretnych podmiotów wymagających większej intensywności operacji.

## Limity żądań API ##
System KSeF ogranicza liczbę zapytań, jakie można wysyłać w krótkim czasie, aby zapewnić stabilne działanie systemu i równy dostęp dla wszystkich użytkowników.
Więcej informacji znajduje się w [Limity żądań API](limity-api.md).

## Limity na kontekst ##

| Parametr                                                    | Wartość domyślna                       |
| ----------------------------------------------------------- | -------------------------------------- |
| Maksymalny rozmiar faktury bez załącznika                | 1 MB                                  |
| Maksymalny rozmiar faktury z załącznikiem                 | 3 MB                                  |
| Maksymalna liczba faktur w sesji interaktywnej/wsadowej | 10 000                                 |

## Limity na uwierzytelniony podmiot ##

### Wnioski i aktywne certyfikaty

| Identyfikator z certyfikatu            | Wnioski o certyfikat KSeF | Aktywne certyfikaty KSeF |
| -------------------------------------- | ------------------------- | ------------------------ |
| NIP                                    | 300                       | 100                      |
| PESEL                                  | 6                         | 2                        |
| Odcisk palca certyfikatu (fingerprint) | 6                         | 2                        |



## Dostosowanie limitów ##

System KSeF umożliwia indywidualne dostosowanie wybranych limitów technicznych dla:
- limitów API - np. zwiększenie liczby żądań dla wybranego endpointu,
- kontekstu - np. zwiększenie maksymalnego rozmiaru faktury,
- podmiotu uwierzytelniającego - np. zwiększenie limitów aktywnych certyfikatów KSeF dla osoby fizycznej (PESEL).

Na **środowisku produkcyjnym** zwiększenie limitów możliwe jest wyłącznie na podstawie uzasadnionego wniosku, popartego realną potrzebą operacyjną.
Wniosek składa się za pośrednictwem [formularza kontaktowego](https://ksef.podatki.gov.pl/formularz/), wraz ze szczegółowym opisem zastosowania.

## Sprawdzanie indywidualnych limitów ##  
System KSeF udostępnia endpointy pozwalające na sprawdzenie aktualnych wartości limitów dla bieżącego kontekstu lub podmiotu:

### Pobranie limitów dla bieżącego kontekstu ###

GET [/limits/context](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1limits~1context/get)

Zwraca wartości obowiązujących limitów sesji interaktywnych i wsadowych dla bieżącego kontekstu.

### Pobranie limitów dla bieżącego podmiotu ###

GET [/limits/subject](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1limits~1subject/get)

Zwraca obowiązujące limity certyfikatów i wniosków certyfikacyjnych dla bieżącego podmiotu uwierzytelnionego.

## Modyfikacja limitów na środowisku testowym ##

Na **środowisku testowym** udostępniono zestaw metod umożliwiających zmianę oraz przywracanie limitów do wartości domyślnych.
Operacje te dostępne są wyłącznie dla uwierzytelnionych podmiotów i nie mają wpływu na środowisko produkcyjne.

### Zmiana limitów sesji dla bieżącego kontekstu ###

POST [/testdata/limits/context/session](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1context~1session/post)

### Przywrócenie limitów sesji dla kontekstu do wartości domyślnych ###

DELETE [/testdata/limits/context/session](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1context~1session/delete)

### Zmiana limitów certyfikatów dla bieżącego podmiotu ###

POST [/testdata/limits/subject/certificate](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1subject~1certificate/post)

### Przywrócenie limitów certyfikatów dla podmiotu do wartości domyślnych ###

DELETE [/testdata/limits/subject/certificate](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Limity-i-ograniczenia/paths/~1api~1v2~1testdata~1limits~1subject~1certificate/delete)


Powiązane dokumenty: 
- [Limity żądań api](limity-api.md)
- [Weryfikacja faktury](../faktury/weryfikacja-faktury.md)
- [Certyfikaty KSeF](../certyfikaty-KSeF.md)