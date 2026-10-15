# Přidávání programů

Umonuje upravovat:
- Stažení vygenerovaných faktur, buď všech nebo jenom zaplacených (stáhne zip soubor který obsahuje faktury a csv soubor ketrý obsahuje data těchto faktur)
- Měnit emaily na který systém zasílá zprávy o špatných platbách - _následně je nutné restartovat backend_
- měnit Fio api token - _následně je nutné restartovat backend_

_Z těchto dat se následně generují faktury:_
- Měnit cenu Eventu
- Měnit datum do kdy zaplatit
- měnit prefix VS (za tento VS bude přidáno 4 místné číslo, viz [platba](https://github.com/RVVZtky/RVVZ-docs-temp/blob/master/payment/README.md))
- Měnit obsah QR kódu (viz níže)
- Měnit šablonu faktury


_Pro přístup k datům je potřeba [permise](https://github.com/RVVZtky/RVVZ-docs-temp/blob/master/permissions/README.md) sensitive\_data\_access_   \
_Pro úpravu dat je potřeba [permise](https://github.com/RVVZtky/RVVZ-docs-temp/blob/master/permissions/README.md) event\_edit_


### QR kód
musí obsahovat palceholdery:
- {{VS}} - variabliný symbol
- {{price}} - cena
a může:
- {{amount}} - kolik lidí
- {{single_price}} - cena jednu položku
- {{pay_by}} - splatnost
- {{date}} - datum vydání
- {{ico}}

Doporučený tvar:
`SPD*1.0*ACC:CZ5020100000000000000000+FIOBCZPPXXX*AM:{{price}}*CC:CZK*MSG:Zpráva pro příjemce*X-VS:{{VS}}*RN:Jméno příjemce*`
nebo s informacemi o faktuře X-INV (to by prý pak mělo jít použít pro automatické účtování v programech jako pohoda a money S3):
`SPD*1.0*ACC:CZ0720100000000000000000+FIOBCZPPXXX*AM:{{price}}*CC:CZK*MSG:Zpráva pro příjemce*X-VS:{{VS}}*RN:Jméno příjemce*X-INV:SID%2A1.0%2AID:VFR{{VS}}%2ADD:{{date}}%2AMSG:Zpráva pro příjemce%2AINI:22743731%2AINR:{{ico}}%2ADUZP:20270110%2ADPPD:20270110%2ADT:{{pay_by}}%2ANTB:{{price}}%2AX-SW:RVVZPRIHLASOVADLO*`

### Faktura
faktura je zadaná jako text .svg souboru   \
Musí obsahovat placeholdry:
- {{number}} - VS faktury
- {{name}} - Jméno klienta
- {{ico}} - Ičo klienta
- {{ICO}} - Pokud je zadané ičo, na tomto místě bude text "IČO:"
- {{address}} - adresa klienta
- {{city}} - město klienta
- {{date}} - datum vydání
- {{amount}} - kolik lidí
- {{single_price}} - cena jednu položku
- {{price}} - celková cena
- {{pay_by}} - splatnost
- {{userIds}} - za jaké lidí se platí