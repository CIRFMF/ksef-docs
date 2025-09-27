## Limity
27.09.2025

### Wstęp ###

W systemie KSeF 2.0 zastosowano mechanizmy limitujące liczbę i wielkość operacji API oraz parametry związane z przesyłanymi danymi. Celem tych limitów jest:
- ochrona stabilności systemu przy dużej skali działania,
- przeciwdziałanie nadużyciom i nieefektywnym integracjom,
- zapobieganie nadużyciom i potencjalnym zagrożeniom cyberbezpieczeństwa,
- zapewnienie równych warunków dostępu dla wszystkich użytkowników.

Limity zostały zaprojektowane z możliwością elastycznego dostosowania do potrzeb konkretnych podmiotów wymagających większej intensywności operacji.

### Limity API ###
System KSeF ogranicza liczbę zapytań, jakie można wysyłać w krótkim czasie, aby zapewnić stabilne działanie systemu i równy dostęp dla wszystkich użytkowników.
Więcej informacji znajduje się w [Limity żądań API](limity-api.md)

### Pozostałe limity ###

| Parametr                                                    | Wartość domyślna                       |
| ----------------------------------------------------------- | -------------------------------------- |
| Maksymalny rozmiar faktury bez załącznika                | 1 MiB                                  |
| Maksymalny rozmiar faktury z załącznikiem                 | 3 MiB                                  |
| Maksymalna liczba faktur w sesji interaktywnej/wsadowej | 10 000                                 |
| Limit certyfikatów KSeF na podmiot                          | zgodny z parametrami środowiska        |


### Dostosowanie limitów do potrzeb podmiotu ###

System KSeF umożliwia indywidualne dostosowanie wybranych limitów technicznych dla konkretnego kontekstu, np. zwiększenie maksymalnego rozmiaru faktury lub liczby wysłanych faktury na godzinę.

Na środowisku testowym zmiany limitów można realizować samodzielnie, za pomocą dedykowanego endpointu:

```
POST `/testdata/limits`
```

Na **środowisku produkcyjnym** zwiększenie limitów możliwe jest wyłącznie na podstawie uzasadnionego wniosku, popartego realną potrzebą operacyjną. Wniosek składa się za pośrednictwem formularza kontaktowego, wraz ze szczegółowym opisem zastosowania.

### Sprawdzanie indywidualnych limitów ###
System KSeF udostępnia endpoint pozwalający na sprawdzenie limitów zmodyfikowanych indywidualnie.

```
GET `/testdata/limits`
```

Powiązane dokumenty: 
- [Limity api](limity-api.md)