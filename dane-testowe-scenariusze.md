## Przykładowe scenariusze
05.08.2025

### Scenariusz nr 1 – Komornik

Jeśli na środowisku testowym chcemy korzystać z systemu KSeF jako osoba fizyczna z uprawnieniami komornika, należy dodać taką osobę za pomocą endpointu `/v2/testdata/person`, ustawiając flagę *isBailiff* na **true**.

Przykładowy JSON:
```json
{
  "nip": "7980332920",
  "pesel": "30112206276",
  "description": "Komornik",
  "isBailiff": true
}
```

W wyniku tej operacji osoba logująca się w kontekście podanego NIP-u, za pomocą numeru PESEL lub NIP-u, otrzyma uprawnienia właścicielskie (**Owner**) oraz egzekucyjne (**EnforcementOperations**), co umożliwi korzystanie z systemu z perspektywy komornika.

---

### Scenariusz nr 2 – JDG

Jeśli na środowisku testowym chcemy korzystać z systemu KSeF jako jednoosobowa działalność gospodarcza, należy dodać taką osobę za pomocą endpointu `/v2/testdata/person`, ustawiając flagę *isBailiff* na **false**.

Przykładowy JSON:
```json
{
  "nip": "7980332920",
  "pesel": "30112206276",
  "description": "JDG",
  "isBailiff": false
}
```

W wyniku tej operacji osoba logująca się w kontekście podanego NIP-u, za pomocą numeru PESEL lub NIP-u, otrzyma uprawnienie właścicielskie (**Owner**) co umożliwi korzystanie z systemu z perspektywy JDG.

---

### Scenariusz nr 3 – Grupa VAT

Jeśli na środowisku testowym chcemy utworzyć strukturę grupy VAT oraz nadać uprawnienia administratorowi grupy i administratorom jej członków, należy w pierwszym kroku utworzyć strukturę podmiotów za pomocą endpointu `/v2/testdata/subject`, wskazując NIP jednostki nadrzędnej oraz jednostki podrzędne.

Przykładowy JSON:
```json
{
  "subjectNip": "3755747347",
  "subjectType": "VatGroup",
  "description": "Grupa VAT",
  "subunits": [
    {
      "subjectNip": "4972530874",
      "description": "NIP 4972530874: członek grupy VAT dla 3755747347"
    },
    {
      "subjectNip": "8225900795",
      "description": "NIP 8225900795: członek grupy VAT dla 3755747347"
    }
  ]
}
```

W wyniku tej operacji w systemie zostaną utworzone wskazane podmioty oraz powiązania między nimi. Następnie należy jakiejś osobie nadać uprawnienia w kontekście NIPu grupy VAT, zgodnie z zasadami ZAW-FA. Operację tę można wykonać za pomocą metody `/v2/testdata/permissions`.

Przykładowy JSON dla osoby uprawnionej w kontekście grupy VAT:
```json
{
  "contextIdentifier": {
    "value": "3755747347",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "38092277125",
    "type": "pesel"
  },
  "permissions": [
    {
      "permissionType": "InvoiceRead",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "InvoiceWrite",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "Introspection",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "CredentialsRead",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "CredentialsManage",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "SubunitManage",
      "description": "praca w kontekście 3755747347: uprawniony PESEL: 38092277125, Adam Abacki"
    }
  ]
}
```

Taką operację można wykonać zarówno grupy VAT (jak wyżej), jak i dla członków grupy VAT. Należy zauważyć, że o ile dla grupy VAT jest to jedyna możliwość nadania inicjalnych uprawnień, o tyle dla członków grupy nie ma takiej konieczności. Można to już zrobić przy użyciu standardowego endpointu /v2/permissions/subunit/grants powołując administratorów członków grupy VAT. 

Alternatywnie można posłużyć się opisanym wyżej endpointem do tworzenia danych testowych. Przykładowy JSON dla nadania uprawnienia `CredentialsManage` administratorowi członka grupy:
```json
{
  "contextIdentifier": {
    "value": "4972530874",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "3388912629",
    "type": "nip"
  },
  "permissions": [
    {
      "permissionType": "CredentialsManage",
      "description": "praca w kontekście 4972530874: uprawniony NIP: 3388912629, Bogdan Babacki"
    }
  ]
}
```

Dzięki tej operacji przedstawiciel członka grupy VAT uzyskuje możliwość nadawania uprawnień sobie lub innym osobom (np. pracownikom) w standardowy sposób, za pośrednictwem systemu KSeF.

### Scenariusz nr 4 – Włączenie możliwości wysyłania faktur z załącznikiem
Na środowisku testowym można zasymulować podmiot, który ma włączoną możliwość przesyłania faktur z załącznikami. Operację należy wykonać za pomocą endpointu /testdata/attachment.

```json
{
  "nip": "4972530874"
}
```

W efekcie podmiot o NIP 4972530874 otrzyma możliwość przesyłania faktur zawierających załączniki.

### Scenariusz nr 5 – Wyłączenie możliwości wysyłania faktur z załącznikiem
Aby przetestować sytuację, w której dana jednostka nie ma już możliwości przesyłania faktur z załącznikami, należy użyć endpointu /testdata/attachment/revoke.

```json
{
  "nip": "4972530874"
}
```

W efekcie podmiot o NIP 4972530874 traci możliwość przesyłania faktur zawierających załączniki