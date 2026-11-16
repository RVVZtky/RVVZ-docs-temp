# Nastavení ceny
Člověk může mít nastavenou cenu buď:
- svojí specifickou cenu (nastavenou ve správě uživatelů u konkrétního uživatele), nebo (pokud tuto nemá uživatel zadanou):
- základní cenu rvvztky (nastavenou v platebních informacích (aneb. "cena pro všechny"))

Když si uživatel vygeneruje fakturu, bude na ní mít cenu, která se na něj aktuálně vztahuje.     \
(stejné platí když se bude generovat faktura za organizaci (více lidí))    \
\
Změny ceny:
- když se specifická cena pro uživatele změní (nebo nově nastaví), přihlašovadlo invaliduje faktury, které jsou na něj navázané.
- když se změní základní cena, přihlašovadlo invaliduje faktury, které jsou vázané na všechny uživatele, kteří nemají nastavenou specifickou cenu

Invalidace se netýká faktur, které už jsou zaplacené (tudíž ani uživatelů, kteří už jsou zaplacení).    \
O invalidaci pošle přihlašovadlo zprávu na stejný email, na který se zasílala kopie dotyčné faktury.    \
Pokud přihlašovadlo najde novou platbu (přes fio api token) za VS invalidované faktury, pošle o tom chybový email.


# Generování faktur a platby

### Fakturování
přihlašovadlo aktuálně umí generovat fakturu buď (přepínání odkazem vždy pod formulářem):
- **na vlastní osobu** (/events/{eventId}/platba) (zadáš jenom svojí adresu), kdy platíš jenom za sebe. VS je pak ve tvaru 202601[čtyrciferné id] (třeba 2026010005), kopie faktury přijde na email uživatele
- **na ičo** (/events/{eventId}/platba/organizace) (zadáš email, ičo, firmu, adresu a id uživatelů za které platíš). VS je pak ve tvaru 202601+[čtyrciferné, postupně od 8000 dál] (třeba 2026018002), kopie faktury přijde na zadaný email

_Prefix VS je nastavitelných v admin panelech, 202601 je zde použito jako ukázka_   \

v obou případech ti přihlašovadlo automaticky fakturu stáhne (na email jde jenom pro redundanci)   \

### Automatická kontrola plateb
systém je napojen na api Fio banky, kde kontroluje každých 15 minut účet pro nový platby   \
když dostane novou platbu tak:
- zkontroluje jestli VS sedí s nějakým, který vygeneroval (jestli ne nedělá nic)
- zkontroluje jestli částka souhlasí s VS (když platíš za víc lidí je částka násobená) (když ne, pošle o tom chybový e-mail)    \
_aktuálně kontroluje částku, která je na faktuře - tedy částku která odpovídá ceně eventu v čase vygenerování faktury (věc, která se změní s možností nastavení individuálních cen)_
- zkontroluje jestli už nějaký z uživatelů asociovaných s VS nebyl už zaplacen (ať už přes tenhle, nebo jiný VS) (když ano, pošle o tom chybový e-mail)
- Jestli všecho sedí, označí automaticky uživatele, že má zaplaceno (a pošle o tom dotyčným uživatelům e-mail, ovšem ne na email na který byla vygenerovaná faktura (při generaci na IČO))

oba chybové emaily chodí jen na emaily zadané v adminovi v sekci "Bankovní informace"
