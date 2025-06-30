## Przykładowe scenariusze
26.06.2025

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
      "description": "Członek grupy nr 1"
    },
    {
      "subjectNip": "8225900795",
      "description": "Członek grupy nr 2"
    }
  ]
}
```

W wyniku tej operacji w systemie zostaną utworzone wskazane podmioty oraz powiązania między nimi. Następnie należy nadać uprawnienia właścicielskie w kontekście NIP-ów należących do członków grupy oraz całej grupy VAT, zgodnie z zasadami ZAW-FA. Operację tę można wykonać za pomocą metody `/v2/testdata/permissions`.

Przykładowy JSON dla członka grupy:
```json
{
  "contextIdentifier": {
    "value": "4972530874",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "60137010667054276304203484",
    "type": "fingerprint"
  },
  "permissions": [
    {
      "permissionType": "InvoiceRead",
      "description": "Odczyt faktur"
    },
    {
      "permissionType": "InvoiceWrite",
      "description": "Wysyłanie faktur"
    },
    {
      "permissionType": "Introspection",
      "description": "Przeglądanie historii"
    },
    {
      "permissionType": "CredentialsRead",
      "description": "Odczyt uprawnień"
    },
    {
      "permissionType": "CredentialsManage",
      "description": "Zarządzanie uprawnieniami"
    },
    {
      "permissionType": "SubunitManage",
      "description": "Zarządzanie jednostkami podrzędnymi"
    }
  ]
}
```

Taką operację można wykonać zarówno dla członków grupy VAT, jak i dla całej grupy VAT.

Przykładowy JSON dla nadania uprawnienia **CredentialsManage** przedstawicielowi członka grupy:
```json
{
  "contextIdentifier": {
    "value": "4972530874",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "98943410705538041801748826",
    "type": "fingerprint"
  },
  "permissions": [
    {
      "permissionType": "CredentialsManage",
      "description": "Zarządzanie uprawnieniami"
    }
  ]
}
```

Dzięki tej operacji przedstawiciel członka grupy VAT (lub całej grupy VAT) uzyskuje możliwość nadawania uprawnień sobie lub innym osobom (np. pracownikom) w standardowy sposób, za pośrednictwem systemu KSeF.
