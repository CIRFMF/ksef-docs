## Tryby offline
10.07.2025

## Wstęp 

W systemie KSeF dostępne są dwa podstawowe tryby wystawiania faktur:
* tryb ```online``` - faktura wystawiana i przesyłana w czasie rzeczywistym do systemu KSeF,
* tryb ```offline``` - faktura wystawiana i przesyłana do KSeF w późniejszym, ustawowo określonym terminie. 

W trybie offline faktury wystawiane są elektronicznie, zgodnie z obowiązującym wzorem struktury FA(3). Najważniejsze aspekty techniczne:
* Przy wysyłce faktury – zarówno w trybie interaktywnym, jak i wsadowym – należy ustawić parametr `offlineMode: true`.
* W przypadku faktur przesyłanych jako online (offlineMode: false) system KSeF samodzielnie może przypisać im tryb offline - na podstawie porównania daty wystawienia z datą przyjęcia. Szczegóły mechanizmu: [Automatyczne określanie trybu wysyłki offline](offline/automatyczne-okreslanie-trybu-offline.md).
* Za datę wystawienia system KSeF przyjmuje wyłącznie wartość zawartą w polu ```P_1``` struktury e-faktury.
* Data otrzymania faktury to data nadania numeru KSeF lub w przypadku udostępnienia poza KSeF, data faktycznego jej otrzymania.
* Po wystawieniu faktury w trybie offline aplikacja kliencka powinna wygenerować dwa [kody QR](kody-qr.md) służące do wizualizacji faktury:
  * **KOD I** – umożliwia weryfikację faktury w systemie KSeF,  
  * **KOD II** – potwierdza tożsamość wystawcy.  
* Fakturę korygującą przesyła się dopiero po nadaniu numeru KSeF dokumentowi pierwotnemu.
* W przypadku gdy przesłana faktura offline zostanie odrzucona z przyczyn technicznych, możliwe jest skorzystanie z mechanizmu [korekty technicznej](/offline/korekta-techniczna.md).


### Zestawienie trybów wystawiania faktur w KSeF – offline24, offline i awaryjny

| Tryb          | Strona odpowiedzialności | Okoliczności uruchomienia                                              | Termin przesłania do KSeF                                                               | Podstawa prawna                             |
| ------------- | ------------------------ | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------- |
| **offline24** | klient                   | Brak ograniczeń (dowolność podatnika)                          | do następnego dnia roboczego po dacie wystawienia                                             | art. 106nda ustawy o VAT (projekt KSeF 2.0) |
| **offline**   | system KSeF              | Niedostępność systemu (ogłoszona w BIP i oprogramowaniu interfejsowym) | do następnego dnia roboczego po zakończeniu niedostępności                                    | art. 106nh ustawy o VAT (od 1 II 2026)      |
| **awaryjny**  | system KSeF              | Awaria KSeF (komunikat w BIP MF i w oprogramowaniu interfejsowym)      | do 7 dni rob. od zakończenia awarii (przy kolejnym komunikacie licznik liczony od nowa) | art. 106nf ustawy o VAT (od 1 II 2026)      |

### Termin przesłania faktury do KSeF przy kolejnych zdarzeniach
W trybach offline24 i offline, jeśli w oczekiwanym okresie dosyłania faktury zostanie ogłoszona awaria KSeF (komunikat w BIP MF lub w oprogramowaniu interfejsowym), termin przesłania przesuwa się i jest liczony od dnia zakończenia ostatnio ogłoszonej awarii, nie dłużej niż 7 dni roboczych.

W trybie awaryjnym, jeżeli podczas siedmiodniowego okresu na dosłanie faktury pojawi się kolejny komunikat o awarii, licznik terminu resetuje się i biegnie od dnia zakończenia tej kolejnej awarii.

Ogłoszenie awarii całkowitej w trakcie któregokolwiek z powyższych trybów powoduje zniesienie obowiązku przesyłania faktur do KSeF.

#### Przykład: tryb offline24 z ogłoszoną awarią KSeF
1. 2025-07-08 (środa)
    * Podatnik generuje fakturę w trybie offline24 (offlineMode = true).
    * Termin przesłania do KSeF ustala się na 2025-07-09 (następny dzień roboczy).
2. 2025-07-09 (czwartek)
    * Ministerstwo Finansów publikuje komunikat o awarii KSeF (BIP i interfejs API).
    * Zgodnie z zasadą: pierwotny termin zostaje przesunięty, a nowy liczony jest od dnia zakończenia awarii.
3. 2025-07-12 (sobota)
    * Awaria zostaje usunięta – system ponownie dostępny.
    * Rozpoczyna się okres 7 dni roboczych na dosłanie zaległej faktury.
4. 2025-07-22 (wtorek)
    * Upływa termin 7 dni roboczych od zakończenia awarii.
    * Aplikacja do tej daty ma czas na wysyłkę faktury do KSeF z ustawionym offlineMode = true.


### Tryb awarii całkowitej
W przypadku ogłoszenia awarii całkowitej (środki społecznego przekazu: TV, radio, prasa, Internet):
* Fakturę można wystawić w formie papierowej lub elektronicznej, bez obowiązku stosowania wzoru FA(3).
* Nie ma obowiązku przesyłania faktury do KSeF po ustaniu awarii.
* Przekazanie nabywcy odbywa się dowolnym kanałem (osobiście, e-mail, inne).
* Datą wystawienia jest zawsze datą faktyczna wskazana na fakturze a data otrzymania data faktycznego otrzymania faktury zakupu.
* Faktury z tego trybu nie są opatrywane kodami QR.
* Faktura korygująca w trakcie trwającej awarii KSeF wystawiana jest analogicznie – poza KSeF, z datą faktyczną.

## Powiązane dokumenty
- [Korekta techniczna faktury offline](offline/korekta-techniczna.md)
- [Kody QR](kody-qr.md)  