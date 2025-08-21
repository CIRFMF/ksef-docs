## Podpis XAdES 
https://www.w3.org/TR/XAdES/  

Dopuszczalne formaty podpisu:
- otaczany
- otaczający

Podpis w formacie zewnętrznym (detached) nie jest akceptowany.

Dopuszczalne transformaty zawarte w podpisie XAdES:
- http://www.w3.org/TR/1999/REC-xpath-19991116 - not(ancestor-or-self::ds:Signature)
- http://www.w3.org/2002/06/xmldsig-filter2
- http://www.w3.org/2000/09/xmldsig#enveloped-signature
- http://www.w3.org/2000/09/xmldsig#base64
- http://www.w3.org/2006/12/xml-c14n11
- http://www.w3.org/2006/12/xml-c14n11#WithComments
- http://www.w3.org/2001/10/xml-exc-c14n#
- http://www.w3.org/2001/10/xml-exc-c14n#WithComments
- http://www.w3.org/TR/2001/REC-xml-c14n-20010315
- http://www.w3.org/TR/2001/REC-xml-c14n-20010315#WithComments

### Dozwolone typy certyfikatów

Dozwolone typy certyfikatów w podpisie XAdES:
* Certyfikat kwalifikowany osoby fizycznej – zawierający numer PESEL lub NIP osoby posiadającej uprawnienia do działania w imieniu firmy,
* Certyfikat kwalifikowany organizacji (tzw. pieczęć firmowa) - zawierający numer NIP,
* Profil Zaufany (ePUAP) – umożliwia podpisanie dokumentu; wykorzystywany przez osoby fizyczne,
* Certyfikat wewnętrzny KSeF – wystawiany przez system KSeF. Certyfikat ten nie jest certyfikatem kwalifikowanym, ale jest honorowany w procesie uwierzytelniania.

**Certyfikat kwalifikowany** – certyfikat wydany przez kwalifikowanego dostawcę usług zaufania, wpisanego do unijnego rejestru [EU Trusted List (EUTL)](https://eidas.ec.europa.eu/efda/trust-services/browse/eidas/tls), zgodnie z rozporządzeniem eIDAS. W KSeF akceptowane są certyfikaty kwalifikowane wydane w Polsce oraz w innych państwach członkowskich Unii Europejskiej.

### Wymagane atrybuty certyfikatów kwalifikowanych

#### Certyfikaty podpisu kwalifikowanego (wydawane dla osób fizycznych)

Wymagane atrybuty podmiotu:<br/>
| Identyfikator (OID) | Nazwa          | Znaczenie                                |
|---------------------|----------------|------------------------------------------|
| 2.5.4.42            | givenName      | imię                                     |
| 2.5.4.4             | surname        | nazwisko                                 |
| 2.5.4.5             | serialNumber   | numer seryjny                            |
| 2.5.4.3             | commonName     | nazwa powszechna właściciela certyfikatu |
| 2.5.4.6             | countryName    | nazwa kraju, kod ISO 3166                |

Rozpoznawane wzorce atrybutu `serialNumber`:<br>
**(PNOPL|PESEL).\*?(?<number>\\d{11})**<br>
**(TINPL|NIP).\*?(?<number>\\d{10})**<br>

#### Certyfikaty pieczęci kwalifikowanej (wydawane dla organizacji)

Wymagane atrybuty podmiotu:<br/>
| Identyfikator (OID) | Nazwa                   | Znaczenie                                                           |
|---------------------|-------------------------|---------------------------------------------------------------------|
| 2.5.4.10            | organizationName        | pełna formalna nazwa podmiotu, dla którego wydawany jest certyfikat |
| 2.5.4.97            | organizationIdentifier  | identyfikator podmiotu                                              |
| 2.5.4.3             | commonName              | nazwa powszechna organizacji                                        |
| 2.5.4.6             | countryName             | nazwa kraju, kod ISO 3166                                           |

Niedopuszczalne atrybuty podmiotu:
| Identyfikator (OID) | Nazwa       | Znaczenie |
|---------------------|------------ |-----------|
| 2.5.4.42            | givenName   | imię      |  
| 2.5.4.4             | surname     | nazwisko  |

Rozpoznawane wzorce atrybutu `organizationIdentifier`:<br>
**(VATPL).\*?(?<number>\\d{10})**<br>

### Odcisk palca certyfikatu

W przypadku certyfikatów kwalifikowanych nieposiadających właściwych identyfikatorów zapisanych w atrybucie podmiotu OID.2.5.4.5 możliwe jest uwierzytelnienie takim certyfikatem po uprzednim nadaniu uprawnień na skrót SHA-256 (tzw. odcisk palca) tego certyfikatu.