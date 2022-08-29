# Next Hop Resolution Protocol (NHRP)
---

Jedná se o nástroj pro poskytování adres pro hosty nebo sítě (podobně jako ARP) na NBMA sítích, jako FR/ATM.

DMVPN používá mGRE a potřebuje tedy způsob mapování underlay IP adres na tunelové IP adresy, to nabízí právě NHRP.

NHRP je *client/server* protokol umožňující zařízením (klientům) se zaregistrovat u serveru přes síťové spojení.
NHRP *next-hop servers* (*NHSs*) jsou zodpovědné za registraci adres nebo sítí, udržování NHRP záznamů a odpovídání na dotazy od *next-hop clients* (*NHCs*).

DMVPN spokes (NHCs) mají staticky nakonfigurované adresy DMVPN hubů (NHSs) tak, aby mohli registrovat své transportní a tunelové IP adresy.
Při pokusu o navázání tunelu s danou tunelovou IP adresou, se NHC dotáže NHSs o transportní IP adresu cíle.

## Zprávy
---
NHRP používá pro svou funkci několik typů zpráv, otevřených a těch, které si přidalo Cisco, každá zpráva ale musí obsahovat:
- Zdrojovou transportní IP adresu
- Zdrojovou tunelovou IP adresu
- Cílovou tunelovou IP adresu
- Typ NHRP zprávy

|Zpráva|Popis|
|:-:|:-:|
|*Registration*|Jedná se o registraci (přidělení) tunelové a transportní IP adresy, odesílané od NHC na NHSs spolu s dalšímy atributy jako je platnost záznamu.|
|*Resolution*|Jedná se o žádost NHC na NHSs o transportní IP adresu k, jemu známé, tunelové.|
|*Redirect*|Dávají prostředníkovy tunelové komunikace možnost informovat jejího původce o tom, že existuje lepší cesta přímo k cíli a že by tedy měl navázat tunel přímo s ním (spoke-to-spoke). Původce může poslat *Suppress* zprávu pro zastavení posílání redirektů v případě, že mu to například nařízuje nějaká politika nebo cesta není dostupná. Tato funkcionaliza je klíčová pro DMVPN Phase 3.|
|*Purge*|Tato zpráva má za úkol vymazat NHRP záznam v Cache. Informuje router o ztrátě cesty používané NHRP. Tato zpráva je typicky posílána od NHS na NHC. |
|*Error*|Tato zpráva slouží k informování odesálatele předešlé zprávy, že došlo k chybě při přenosu.|

Tyto zprávy mohou obsahovat další informace obsažené v *Extension part*:

|NHRP Extension|Popis|
|:-:|:-:|
|Responder address|Má za úkol zjistit adresu odpovídajícího zařízení.|
|Forward transit NHS record|Obsahuje seznam NHSs, které může NHRP request využít.|
|Reverse transit NHS record|Obsahuje seznam NHSs, které může NHRP reply využít.|
|Authentication|Obsahuje autentizační informace pro účastníky NHRP, pole je posíláno jako čístý text.|
|Vendor private|Obsahuje Vendor-specific informace mezi NHRP účastníky|
|NAT|DMVPN funguje i přes NAT a při použití IPsec. Pomocí tohoto rozšíření je možnost zjistit *claimed NBMA address* (inside local) pomocí NHRP protokolu a inside global adresu pomocí IP hlavičky NHRP paketu.|

## NHRP Cache
---
Každé zařízení si uchovává chache requestů, které příjme nebo zpracovává.
Obsahuje:
- IP pro hosta (/32;/128) nebo síť 
- Next-hop (via)
- NMBA adresu (underlaying IP)
- Interface, dobu existence a vypršení
- Typ
- Flag

|Typ záznamu|Popis|
|:-:|:-:|
|*static*|Záznam vytvořen staticky na DMVPN interfacu|
|*dynamic*|Záznam vytvořen dynamicky|
|*incomplete*|Dočasný záznam umistěn lokálně dokud probíhá NHRP resolution request, má za úkol zabránít opakovanému dotazování na stejný záznam|
|*local*|Zobrazuje lokální informaci k mappingu, typicky reprezentuje lokální síť, která byla přeposlána jako NHRP resolution reply, udržuje informace o tom, jaké nody tento záznam dostaly|
|*(no-socket)*|Záznamy, které nemají asociován IPsec soket a nepoužívá se tedy šifrování|
|*NBMA address*|Underlaying IP adresa, ze které byl záznam získán|

|Flag|Popis|
|:-:|:-:|
|*Used*|Záznam byl využit pro forwarding v posledních 60s|
|*Implicit*|Indikuje, že záznam byl naučen implicitně (nepřímo), typicky se ho zařízení naučilo forwardováním NHRP resolution paketu|
|*Unique*|Záznam musí být unikání, nelze ho tedy přepsat záznamem, který má stejnou tunnel IP, ale jinou NBMA IP|
|*Router*|Záznam poskytuje cizí router, který poskytuje spojení do dané sítě nebo host za daným routerem|
|*Rib*|Daný záznam má odpovídající H routu b RIB|
|*Nho*|Daná cesta mění next-hop, který je do dané sítě instalován routing protokolem|
|*Nhop*|Daný záznam pro remote next-hop je asociován s NBMA adresou|

## NHRP Autentizace

NHRP podporuje autentizaci, ale pouze *plaintext*, reálně se tedy nejedná o zabezpečení, pouze se používá pro zaručení toho, že se nenaváží spojení, které administrátor nechtěl.

## Unique IP NHRP Registration

Defaultně při registraci NHC u NHS požaduje NHC (pomocí *Uniqueness* bitu), že protocol IP (tunnel IP) bude pevně přiřazena jeho NBMA (transport) IP adrese, to označuje i flag *Unique*. Pokud se NHS pokusí registrovat danou adresu pomocí jiné NBMA adresy, proces neprojde.
To může způsobovat problémy u připojeních s DHCP, u kterých se mění IP adresy, v takovém případě by tedy musel NHC čekat do vypršení platnosti záznamu.
Pomocí příkazu zadávaném na NHC: `ip nhrp registration no-unique`, se toto nastavení zruší a NHC bude moci měnit své adresy.

> Aby se nastavení uchytilo musí dojít k oznámení na NHS, k tomu nedojde okamžitě, pouze při vypršení time-outu nebo znovu navázání spojení, nejrychlejší je tedy vypnout a zapnout interface.