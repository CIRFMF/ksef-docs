## Automatyczne określanie trybu wysyłki offline
04.10.2025  

W przypadku faktur przesyłanych jako online (`offlineMode: false`) system KSeF może przypisać im tryb offline - na podstawie porównania daty wystawienia z datą przyjęcia do przetwarzania.

## Algorytm mechanizmu

Dla faktur wysyłanych jako `offlineMode: false` system porównuje:
- **datę wystawienia** faktury (`issueDate`, np. `P_1` dla faktury zgodnej z FA(3)),
- **datę przyjęcia** faktury w systemie KSeF do dalszego przetwarzania (`invoicingDate`).

Reguły:
- Jeśli dzień kalendarzowy z `issueDate` jest wcześniejszy niż dzień kalendarzowy z `invoicingDate` (porównanie po dacie, nie po godzinie), system automatycznie oznacza fakturę jako **offline**, nawet jeśli nie była tak zadeklarowana.
- Jeśli dzień `issueDate` i dzień `invoicingDate` są takie same, faktura pozostaje **online**.

Wartość `invoicingDate` zależy od trybu przesyłki:
- **sesja wsadowa** - `invoicingDate` to moment otwarcia sesji,
- **sesja interaktywna** - `invoicingDate` to moment przesłania faktury.

To oznacza, że jeśli np. faktura została wystawiona 2025-10-03 (`P_1`), a przesłana 2025-10-04 o godzinie 00:00:01, to mimo offlineMode: false zostanie oznaczona jako faktura offline.

## Przykłady
**Sesja wsadowa** otwarta o 23:59:59 3 października:
Nawet jeśli paczka będzie przesłana po północy, faktury pozostaną online – ponieważ `invoicingDate` to 3 października (data otwarcia sesji).

**Sesja interaktywna** rozpoczęta o 23:59:59 3 października, a faktury zostały przesłane po północy:
Jeżeli `P_1` = 2025-10-03, system oznaczy je jako offline – ponieważ dzień `P_1` jest wcześniejszy niż dzień przesłania.


## Powiązane dokumenty
- [Tryby offline](../tryby-offline.md)