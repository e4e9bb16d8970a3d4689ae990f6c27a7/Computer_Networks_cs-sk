# OSPF Adjacency
---

## Navazování spojení a výměna informací
---

Na samotný proces navazování spojení nemá vliv rozdělení do areí.

### Router ID

Při zapnutí procesu si musí každý router vygenerovat `32-bit` dlouhý identifikátor (RID), který identifikuje zařízení v LSDB, Cisco routery ho vybírají následovně:

1. Pomocí příkazu `router-id`
2. Nejvyšší IP adresa **loopback** interfacu, která ještě není přidělena jinému OSPF procesu
3. Nejvyšší IP adresa libovolného, povoleného, interfacu, která ještě není přidělena jinému OSPF procesu

- Samotný interface nemusí být nutně zahrnut do OSPF pomocí příkazu `network`.
- Samotný interface může být v *down/down* stavu, ale nesmí být *administratively down*
- Samotný RID funguje i jako označení přístupnosti do dané sítě
- Síť  RID nemusí být inyerovaná v OSPF
- RID nemusí být přístupné skrze RIB
- Router změní RID pouze při restartování OSPF procesu nebo manuálním nastavení
- Při změně RID musí ostatní routery v arei provést novou SFP kalkulaci

### Navazování sousedství

OSPF má 5 vlastních zpráv, které enkapsuluje uvnitř IP protokolu 89.

OSPF používá Hello pakety pro objevování OSPF-enabled zařízení, kontrolu kompatibilních nastavení, verifikace obousměrné komunikace a kontrolu spojení.

Pro objevování posílá defaultně OSPF Hello pakety na `224.0.0.5` a `FF02::5` multicast adresy, jako zdrojové IPv4 adresy používá primární adresy na egress interfacu, nikoli sekundární, ostatně, jako [[EIGRP]] a [[RIP]].

Pro navázání sousedství musí mít routery shodných několik hodnot:

- Autentizaci
- Musí být ve stejné síti, včetně shodné masky
- Musí být ve stejné OSPF arei
- Musí mít shodný typ arei
- Nesmí mít stejné RID
- Musí mít shodné Hello a Dead časovače
- MTU
	- Sice není technicky požadováno v rámci Hello verifikace, ale DD pakety bez shodného MTU nebudou správně zpracovány
	- Router se zastaví v 2-Way stavu

Nutno podotknout, že číslo procesu se shodovat nemusí, jedná se o lokálně signifikantní označení.

Další funkce pro verifikaci obousměrněho spojení funguje tak, že router v rámci Hello paketů posílá seznam všech známých RID na segmentu, pokud od sousedních routerů dostává Hello pakety s jeho vlastním RID, může si být jistý, že konektivita je obousměrná.

Poslední funkce je ověření funkčnosti spojení mezi routery, každých *Hello timer* (10s) posílá Hello paket, na broadcast nebo p-t-p spojeních a 30s na non-broadcast nebo P-t-MP spojeních, pokud naopak paket nedostane od souseda do uplynutí *Dead Timeru* ($4\times$ *Hello Timer*), pak spojení rozváže.

#### Master/Slave

Při vytváření sousedství dochází k vytvoření Master-Slave vztahu a určení rolí. Master má za úkol Sekvenovat a vytvářet prvnotní DD pakety, na které Slave(s) mohou odpovídat vlastními.
DD paket obsahuje 3 flagy:

- Master (MS) flag
	- 1 Pokud paket odeslal Master
	- 0 Pokud paket odeslal Slave
- More (M) flag
	- 1 Pokud router potřebuje poslat další DD po tomto
- Init (I) flag
	- Indikuje prvnotní DD, které nastavuje sekvenční číslo

Při určení Master-Slave stavu oba routery jako první DD pošlou paket s nastavenými MS, M a I flagy a jejich vlastním sekvenčním číslem, router s nižším RID na tento paket odpoví DD paketem bez I a MS flagu se sekvenčním číslem od sousedního routeru a přepne se tak do Slave role.

#### LSA

Mnohdy se na *Link-State Adwertisement* (LSA) pohlíží jako na paket, který přenáší routovací informace, ve skutečnosti se ale jedná o datovou strukturu, do které jsou uspořádány sítě uvnitř LSDB, tyto LSAs se poté posílají v rámci LSU zpráv.

Každý LSA záznam obsahuje i sekvenční číslo, které označuje jeho verzi. Jedná se o `32-bit` dlouhý `int`, dle něho sousední router při výměně DD pozná, že má zastaralou informaci.
Samotné sekvenční číslo je zapsáno hexadecimálně v hodnotách `0x80000001 - 0xFFFFFFFF`.

#### Type 1 Hello

Používána pro objevování sousedů a udržování spojení.

#### Type 2 Database Description (DD/DBD)

Používána pro výměnu LSA hlaviček při initializaci výměny LSDB, aby routery znali navzájem list LSAs a jejich verzí.

Jedná se o úspornější variantu oproti [[EIGRP]], ve kterém se při navázání spojení vymění celá tabulka, zde pouze rozdíly.

Pro korekci chyb a spolehlivé doručení používá sekvenční číslo, na kterém se routery domluví, první odešle DD s číslem nastaveným na $x$, načež mu druhý pošle zpět DD s číslem nastaveným také na $x$, sekvenční číslo dalšího poslaného DD bude $x+1$.

#### Type 3 Link-State Request (LSR)

V případě, že router pomocí DD zjistí, že sousedova LSDB má některé LSA záznamy novější, než jeho, pošle request o kompletní informace ke konkrétním LSA.

#### Type 4 Link-State Update (LSU)

Jedná se o kompletní informace ke konkrétním LSA jako odpověď na LSR.

Pro zajištění spolehlivosti přenosu může sousední router buď odpovědět kopií LSU, nebo pomocí LSAck, které obsahuje LSA hlavičku přijatého LSA.

#### Type 5 Link-State Acknowledgment (LSAck)

Používá se pro potvrzení příjmutí 

#### Proces

![[OSPF_Nei.png]]

Při sestavování sousedství prochází routery řadou tranzitních, ale i trvalých stavů popisujících část vytváření sousedství, což značne ulehčuje throubleshooting.

Dále je nutné podoktnout, že následující stavy se neshodují s vytvářením *Adjacency*, ale s vytvářením sousedství.

##### Down

Jedná se o první stav při vytváření sousedství, indikuje ztracení spojení se sousedem, například kvůli vypršení Dead intervalu nebo manuálně nastaveného souseda nereagujícího na Hello pakety.

V případě, že OSPF registruje souseda, ale má ho v *Down* stavu, OSPF již má určité informace o sousedovy, typicky IP adresu.

##### Attempt

Tento stav se nachází pouze u NBMA a P2PNB sítích.
Na těchto sítích je soused okamžitě nastaven na Attempt úroveň a jsou mu posílány Hello pakety, pokud na ně ovšem nereaguje, je přepnut do *Down* stavu a interval odesílání paketů je snížen.

##### Init

Do tohoto stavu se soused přepne v případě, že router od něho dostane validní Hello paket, který ale neobsahuje jeho vlastní RID a nemůže si tedy být jistý oboustranou konektivitou.

##### 2-Way

V tomto stavu router dostal od souseda Hello paket s vlastním RID a je tedy ověřena obousměrná konektivita.
Tento stav je na Multiaccess sítích trvalý, pokud není očekávané plné spojení.

##### ExStart

Do tohoto stavu jsou routery přesunuty po ověření obousměrné konektivity, tento stav slouží k synchronizaci počátečních hodnot pro výměnu LSAs.
Routery si vyměňují prázdné DD pakety pro určení *Master/Slave* vztahu, porovnání RID a shodnutí se na počítečním sekvenčním číslu pro LSAck pakety.

##### Exchange

V tomto stavu se posílají DD pakety s hlavičkami LSA pro synchronizaci LSDB.
Každý router si v tomto stavu buduje seznam LSAs, které bude potřebovat dodoplnit.

##### Loading

Do tohoto stavu se router přepne po odeslání všech LSA hlaviček pomocí DD a pokud potřebuje následně zjistit doplňující informace pro konkrétní LSAs.

V tomto stavu probíhá výměna routovacích informací a faktická synchronizace tabulek pomocí LSR a LSU paketů.

##### Full

Do tohoto stavu se router přepne z Exchange nebo Loading stavu v případě, že LSDB jsou plně, mezi sousedy, synchronizovány.

## Designated & Backup Designated Routers
---

Na Multiaccess sítích by routery musely navázat spojení každý s každým, znamenaje, že na segmentu 6 routerů by bylo 15 spojení, 15 synchronizací LSDB.
Fakticky router navazuje spojení pouze s DR a BDR, pokud není DR/BDR, poté ho navazuje se všemi, ostatní router jsou v *2WAY/DROTHER* stavu, je tedy navázané sousedství, ale není potřeba plné spojení, OSPF proto odděluje 2 druhy spojení:

- Neighbors
	- Dvě zařízení na stejném segmentu výměňující si Hello pakety
	- [[#2-Way]]
- Adjacent (Fully adjacent)
	- Dva sousedé, kteří si přímo sesynchronizovali LSDB pomocí DD a LSU


Na Multiaccess segmentu se navazují sousedství pouze s DR/BDR routery. Těmto se posílají [[#Link-State Update LSU|LSU]] s [[#LSA]] na `224.0.0.6`, zabudují si je do LSDB a přepošlou je na `224.0.0.5` všem ostatním zařízením, protože oni sousedí s každým routerem.
DR/BDR neposílají na první [[#Link-State Update LSU|LSU]] [[#Link-State Acknowledgment LSAck|LSAck]], jako acknowledgment je bráno následné rozeslání všem ostatním routerům.
Na Multiaccess segmentu tedy musí existovat DR & BDR, jinak se zařízení nemohou přepnout do Full stavu.

### Volba DR & BDR

Volba se odehrává v [[#ExStart]] stavu.
V případě, že příchozí Hello paket obsahuje adresu DR na `0.0.0.0`, DR ještě nebyl zvolen, zařízení do zahájení volby čekají *OSPF wait time*, který se rovná *Dead Timeru* (40s), aby se každé zařízení mohlo přepnout do 2-Way stavu a účastnit se volby.
V případě, že příchozí Hello paket obsahuje validní DR adresu, zařízení nemusí čekat na volbu, mohou s prosecem začít hned, v takovém případě se router nesnaží o novou volbu, považuje DR z Hello paketu za validní.

- Volby se může účastnit pouze zařízení s prioritou `1 - 255` včetně, pokud je priorita rovna 0, nemuže se stát DR/BDR.
- Každý router provádí vlastní proces na základě dat, které dostal, ale algoritmus se postará o to, aby nakonec všechny zařízení na segmentu měli shodné hodnoty
- Během wait intervalu router shromažďuje informace o sousedních zařízeních, na základě kterých po uplynutí timeru provede volbu, sám posílá Hello pakety, ale zatím se neprohlašuje za DR/BDR
- Pokud v průběhu wait intervalu router dostane hello paket s nastaveným DR, ale bez BDR, zařízení okamžitě začne s volbou BDR a DR příjme za funkční
- Po uplynutí wait intervalu s volí neurčené role, DB/DBR, volba se zakládá na nejvyšší a druhé nejvyšší prioritě, pokud jsou priority shodné, o DR/BDR rozhodne nejvyšší a druhé nejvyšší RID
- Pokud se do již navoleného segmentu připojí nový router s lepšími vlastnostmi (priorita, RID), nebo se zvýší priorita nějakého zařízení, nedochází ke změně DR/BDR, nefunguje preemptivita
- Při selhání DR jeho roli okamžitě zastane BDR a uskuteční se volba na nový BDR

Může se stát, že dva nebo více routerů, například z důvodu rozdělení sítě kvůli [[STP#Indirect Failures|STP Failure]], budou na stejném segmentu mít rozdílné hodnoty DR/BDR, v takovém případě se ignoruje pravidlo ne-preemptivity a dochází k nové volbě.
Rozdíl, mezi tímto scénářem a novým připojením routeru do segmentu je v tom, že nový router nemá nastaveného DR/BDR, kdež to tyto zařízení aktivně propagují své DR/BDR routery.

### DR/BDR na dalších typech sítí

Na různých sítích může být optimalizace různě prospěšná, například na P-t-P linkách není DR/BDR potřeba, protože se jedná pouze o spoj 2 zařízení, ale na Multiaccess médiu, kde mohou být i tisíce zařízení se jedná o značnou úsporu zdrojů.
U ostatním typů, jako například Non-broadcast multiaccess (NBMA) může být o prospěšnosti diskutováno.
Proto je možné u Cisco zařízení nastavit typ připojené sítě a tím tak i chování interfaců, tímto nastavením se ovlivní:
- Zda se má volit DR/BDR
- Multicastové, automatické, odhalování sousedů / manuální nastavení
- Zda je možné mít více, než 1 souseda na segmetnu

OSPF si děfautlně umí typ nastavit samo na základě L2 protokolu, například klasický Ethernet (LAN) používá *Broadcast*, [[PPP]] a [[HDLC]] používají *Point-to-point*.

|Typ sítě|DR/BDR|Hello interval|Nutnost statického nastavení sousedů|Více, než 2 zařízení na spoji|
|:-:|:-:|:-:|:-:|:-:|
|Broadcast|Ano|10s|Ne|Ano|
|Point-to-point|Ne|10s|Ne|Ne|
|Non-broadcast multiaccess (NBMA)|Ano|30s|Ano|Ano|
|Point-to-multipoint nonbroadcast|Ne|30s|Ano|Ano|
|Loopback|Ne|-|-|Ne|

**Point-to-point - Defautlně na Frame Relay point-to-point subinterfacech.**
**NBMA - Defaultně na Frame Relay fyzických a multipoint subinterfacech.**

### Neighbor

Tímto příkazem se staticky nastavují sousedé, kteří se mají kontaktovat pro navázání Adjacency, za IP adresou lze nastavit i prioritu samotného souseda, která je brána v potaz při volbě DR/BDR.

Defaultní priorita je 0 a nepřímo zasahuje do volby, sice nemá vliv na samotnou volbu, ale pokud má router nějaký interface s nenulovou prioritou, při wait timeru, než se zvolí DR/BDR, bude posílat Hello pakety pouze těmto nenulovým interfacům, až po zvolení je začne posílat i ostatním.
Pokud je tato funkce nastavné správně, tedy na obou stranách spoje, administrátor jí může vybrat pouze úzkou skupinu zařízení, které se mohou stát DR/BDR, protože ostatní se volby nebudou účastnit.
Tato funkce se využívá zejména u [[OSPF Frame Relay]], kde je nutné mít u DR/BDR nastavenou PVC ke každému non-DR zařízení.

Neighbor priority se nemusí shodovat s OSPF prioritami sousedů, ale  pokud se lišší, router může zahrnout do DR/BDR volby i původně neurčené routery, je tedy doporučeno, aby byly shodné, v některých starších verzích IOSu systém sám upravil neigbor prioritu na základě hello paketů.

Toto lze nastavit pomocí příkazu `R(config-router)#neighbor <IP> priority <PR>` nebo `R(config-if)#ip ospf priority <PR>`.
V případě nastavení pomocí `ip ospf priority`, router automaticky smaže odpovídající `neighbor` záznamy, aby router zbytečně neposílal vlastní hello pakety s OSPF prioritou 0, která mu znemožňuje se stát DR/BDR.

Routery při statickém nastavení `neighbor` naváží spojení, i pokud je příkaz nastaven pouze na jedné straně, ale jedoporučováno, pro stabilitu, tento příkaz konfigurovat a obou stranách spoje.



