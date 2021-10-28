# EIGRP Pakety
---

EIGRP pakety jsou přenášeny přímo na L3 pomocí protokolu 88, jejich velikost je odvozena od MTU linky, na klasické 1500 MTU lince se tedy jedná až o 1480 B.

Každý paket obsahuje `20B` dlouhou hlavičku nasledovanou [[TLV]] poli se samotnými informacemi.

![[EIGRP_Packet.png]]

## Hlavička
---
### Version Field

Indikuje verzi protokolu, od vytvoření protokolu se hodnota nezměnila, je vždy 2.

### Opcode

Určuje typ paketu, důležité jsou:
1.[[#Update]]
3.[[#Query]]
4.[[#Reply]]
5.[[#Hello/Ack]]
10.[[#SIA-Query]]
11.[[#SIA-Reply]]

Ostatní typy již nejsou a ani nikdy nebyly používány.

### Checksum

Obsahuje kontrolní součet pro EIGRP Payload.

### Flags

- `0x1` - Init
	- Používán při navazování sousedství
- `0x2` - Conditional Receive
	- Používán RTP pro určení skupiny cílových zařízení
- `0x4` - Restart
	- Indikuje, že router byl restartován
- `0x8` - End-of-Table
	- Při prvotní synchronizaci RIB indikuje, že došlo ke kompletnímu přenosu tabulky

### Sequence Number

Sekvenční číslo používané RTP pro určení pořadí paketů a jejich spolehlivé doručení.

### Acknowledgment Number

Používáno RTP, obsahuje sekvenční číslo posledního paketu, které router dostal od příjemce, opět tedy funguje jako nástroj pro zajištění spolehlivého přenosu.

Hello paket s nenulovým ACK číslem je brán jako Acknowledgment paket.
Zároveň nenulový ACK paket je vždy unicast, protože toto pole není používáno u multicastovaných paketů.

### Virtual RID

Identifikuje router, který paket odeslal.

- `0x1` - Unicast Address Family
- `0x2` - Multicast Address Family
- `0x8000` - Unicast Service Address Family

### AS Number

Identifikuje EIGRP doménu, ta defaultně slouží jako číslo AS (krom [[Named EIGRP]]).

## TVL
---

Obsahuje informace pro RIB a DUAL.

EIGRP porporuje několik typů TLVs:

- `0x0001` - EIGRP Parameters ( *General TLV Types* )
- `0x0002` - Authentication Type ( *General TLV Types* )
- `0x0003` - Sequence ( *General TLV Types* )
- `0x0004` - Software Version ( *General TLV Types* )
- `0x0005` - Next Multicast Sequence ( *General TLV Types* )
- `0x0102` - IPv4 Internal Routes ( *IP-Specific TLV Types* )
- `0x0103` - IPv4 External Routes ( *IP-Specific TLV Types* )
- `0x0402` - IPv6 Internal Routes ( *IP-Specific TLV Types* )
- `0x0403` - IPv6 External Routes ( *IP-Specific TLV Types* )
- `0x0602` - Multi Protocol Internal Routes (*AFI-Specific TLV Types*)
- `0x0603` - Multi Protocol External Routes ( *AFI-Specific TLV Types* )

## Typy paketů
---
- [[#Hello]]
- [[#Acknowledgment]]
- [[#Update]]
- [[#Query]]
- [[#Reply]]
- [[#SIA-Query]]
- [[#SIA-Reply]]

### Hello

EIGRP periodicky rozesílá Hello pakety z interfaců, na kterých byl zapnut EIGRP proces (pomocí `network` příkazu).
Tyto zprávy mají identifikovat sousedy, ověřit jejich funkčnost a kompatibilitu (IP subnet, AS, K-values a autentizace). Dále slouží pro udržování sousedského spojení.
Defaultně jsou posílány 1 x 5s na `224.0.0.10` nebo `FF02::A`, pokud je manuálně nakonfigurované sousedství, nepoužívá se multicast adresa, ale unicast.
Na NBMA (Frame Relay) interfacech s bandwithem pod `1544 kbps` se interval snižuje na 1 x 60s.
Opcode je 5 bez ACK.

### Acknowledgment

Jedná se nejčastěji o prázdný paket, tedy bez TLV, s vyplněným [[#Acknowledgment Number]].
Používá ho RTP pro potvrzení přijmutí paketů:
- [[#Update]]
- [[#Query]]
- [[#Reply]]
- [[#SIA-Query]]
- [[#SIA-Reply]]

Pro ušetření provozu je možné použit jako ACK paket i jiný, než čistý hello paket, v zásadě jakýkoli paket může obsahovat mimo jiných informací [[#Acknowledgment Number]], kterým potvrzuje přijetí jiného paketu.

### Update

Tento paket obsahuje samotné routovací informace ve formě TLV.
Může být unicast i multicast.

Při vytváření nových sousedství jsou Update pakety většinou posílány jako unicast, vyjímkou může být situace, kdy router objeví spoustu potencionálních sousedů v relativně krátkém čase, typicky při použití DMVPN, v takovém případe router použije multicast.
Proces, dle kterého se router v takových případech rozhoduje je proprietární a není tedy znám veřejnosti, ale nemá dopad na konečný výsledek, pouze efektivitu synchronizace.

Po plném sesynchronizování se následné update pakety posílají jako multicast, v případě, že na první multicast paket nepříjde ACK, router odešle unicast update pro specifické zařízení.

Na P-t-P linkách a při statickém určení sousedů se vždy využívá unicast.

Používají Opcode 1.

### Query

Jedná se o paket, kterým se router dotazuje na konektivitu do specifické sítě pokud ztratí primární konektivitu a *Feasible successor*.

 Query pakety se posílají jako multicast, v případě, že na první multicast paket nepříjde ACK, router odešle unicast update pro specifické zařízení.

Na P-t-P linkách a při statickém určení sousedů se vždy využívá unicast.

Používají Opcode 3.

### Reply

Jedná se o odpověď s cestou do sítě požadované Query paketem, paket obsahuje odesílatelovu cestu s přidáním vlastních metrik (v případě, že jde přes více, než 1 router).

Tyto pakety jsou vždy posílány unicastem.

Používají Opcode 4.

### SIA-Query

Pomáhá zjistit, zda router, soused, kterému byl poslán klasický Query paket stále pracuje na zjištění informací a je stále aktivní.
Router, který dostane tento paket na něj okamžitě odpoví s SIA-Reply.

Používá Opcode 10.

### SIA-Reply

Jedná se o odpověď na SIA-Query potvrzující funkčnost a participování ve výpočtu nové cesty do určené sítě.

Používá Opcode 11.
