# IS-IS Pakety
---
Oproti [[OSPF]], které má několik typů [[OSPF Areas & LSAs#LSAs|LSA]] paketů, *Link State Protocol Data Unit* (LSP) obsahuje všechny informace:
- Sousední routery
- Připojené sítě
- Inter-area prefixy
- Externí prefixy

[[Routing/Routing Protocols/IS-IS/IS-IS|IS-IS]] tyto informace diferencuje na základě [[TLV]].
Speciálním případem jsou [[IS-IS Network Types#Multiaccess|Multiaccess]] sítě, u kterých se volí *Designated IS* (DIS).
Informace o adresách je sdílena každým připojeným routerem pomocí jeho vlastních LSP.
Informace o topologii je generována DIS pomocí *Pseudonode LSP*.

Každý router tedy generuje LSP popisující jeho vlastní adresní informace + jedno pseudonode LSP pro každou síť, ve které je DIS.
LSP jsou generovány přo každé změně, adresní nebo topologické.

## Hello paket
---
*IS-IS Hello* (IIH) má za úkol:
- Objevování sousedů a ověřování jejich dostupnosti
- Ověření obousměrné komunikace
- Navázání adjacency
- Volbu DIS

Na broadcast sítích se posílají odděleně hello pro L1 a L2, na P-t-P spojích se, z důvodu šetření bandwithu, posílají L1L2 hello dohromady.

## Timers
---
Časovače se pro navázání spojení nemusí nutně shodovat.

#### Hello timer

Hello timer je defaultně *10s*, možno nastavit od 1 do 65535.

#### Hold timer

Hold timer jako takový nemá staticky nakonfigurovanou hodnotu, určuje se jako násobek *multiplier* a *Hello timer*, multiplier je defaultně *3*, defaultní hold timer je tedy *30s*.

#### LSP Lifetime

*LSP Remaining LifeTime* timer je defaultně nastaven na *1200s* (20 minut), označuje životnost informací, pokud nedojde po nastavenou dobu k obnovení informací,k čemuž by mělo docházet každých *900s* (15 minut), router smaže tělo LSP, ponechá si hlavičku, a začne LSP rozesílat s LifeTime = 0, což vyprodukuje stejnou reakci u přijímacích routerů, tato funkce se nazývá *LSP Purge*.

Aby se bezpečně LSP Purge dopropagoval celou areou, router si defaultně poněchá hlavičku LSP v LSDB po dobu dalších *ZeroAgeLifeTime*, *60s*. Toto je defaultní chování IS-SI předpisu, ale Cisco implementace uchovává LSP halvičku po dobu *1200s*.


## Link State PDU
---
LSP, jako paket, je podobný [[OSPF Areas & LSAs#LSAs|LSU]] u OSPF, má za úkol sdílet konkrétní směrovací informace mezi IS.
LSP je zároveň ale termín, podobně jako LSA, popisující všechny jednotlivé informace v LSDB.

### LSPID

Jednotlivé LSP jsou mezi sebou diferenciovány na základě LSPID, které se skládá z:
- **System ID**
	- `6B` *System ID* IS, které generovalo LSP
- **Pseudonode ID**
	- Používá se pro rozdělení informací přímo o routeru a segmentu sítě
	- Je používáno DIS
	- Pokud LSP popisuje router je rovno `0`
- **LSP Number**
	- Obsahuje číslo [[#Fragmentace|fragmentu]]

### LSP Sequence Number

Pro oddělení jednotlivých verzí LSP [[Routing/Routing Protocols/IS-IS/IS-IS|IS-IS]] používá *Sequence Number*, `32b` číslo. Každé LSP začíná na `0` a postupně, s každou novou verzí, se zvyšuje o 1.

Koncept je téměř shodný s OSPF, s jedním rozdílem, pokud sequence number dojde maximální hodnotě u OSPF, router se z tohoto stavu dokáže sám dostat, u IS-IS je nutné buď změnit *System ID*, nebo vypnou router a počkat na vypršení platnosti současného LSP.
Toto může být vnímáno jako zásadní chyba, ale rozsah $2^{32}$ umožňuje pro bezpečné fungování po dobu 136 let, pokud by router generoval novou verzi každou sekundu + řada dnešních implementací zpomaluje tuto generaci.

## Complete & Partial Sequence Numbers PDU
---
*Complete Sequence Numbers PDU* (CSNP) a *Partial Sequence Numbers PDU* (PSNP) mají za úkol sesynchronizovat verze LSP v LSDB a určit LSP, o které je potřeba si zažádat.
Princip je stejný, jako u [[OSPF Areas & LSAs|DD a LSR]].

### CSNP

Má za úkol poslat všechny LSP, které má router v LSDB a jejich *Sequence Number* pro určení nejnovější verze.

Každý paket obsahuje seznam všech LSPID, které popisuje, seznam je zapsán jako range od *Start LSPID*, nejnižší LSPID, po *End LSPID*, poslední LSPID.
Defualtně by měl tento range vypadat takto:
- `0000.0000.0000.00-00` - `FFFF.FFFF.FFFF.FF-FF`
V takovém případě se všechny LSP vešli do jednoho framu, ale pokud by došlo k překročení MTU, lze je rozdělit do několika framů:

| Frame |     Start LSPID      |      End LSPID       |
|:-----:|:--------------------:|:--------------------:|
|  1.   | `0000.0000.0000.00-00` | `AAAA.AAAA.AAAA.AA-AA` |
|  2.   | `AAAA.AAAA.AAAA.AA-AA` | `EEEE.EEEE.EEEE.EE-EE` |
|  3.   | `EEEE.EEEE.EEEE.EE-EE` | `FFFF.FFFF.FFFF.FF-FF` |

Na [[IS-IS Network Types#Point-to-Point|P-t-P]] linkách jsou CSNP vyměňovány pouze při prvotním navazování spojení, na [[IS-IS Network Types#Broadcast|Broadcast]] jsou vyměňovány periodicky od DIS.

### PSNP

Tento paket nahrazuje funkci LSA a LSR, má za úkol potvrdit přijetí CSNP a požádat o doplňující informace o chybějících/zastaralých LSP.

## Fragmentace
---
Protože IS-IS funguje na L2, musí se starat o MTU (`1500B`).
Každý frame/paket se skládá z fixní hlavičky a libovolně dlouhého těla tvořeného TLV poli, pokud se všechny TVL pole LSP nevejdou do jednoho framu, lze je rozdělit do více framů, poté přichází na řadu *LSP Number*, také nazýváno *Fragment Number*, tedy číslo, které se o 1 zvyšuje s každým framem a má za úkol kompletní doručení LSP.
Protože s IS-IS pakety může manipulovat pouze router, který je vygeneroval, MTU se musí, pro správnou funkci, shodovat napříč areou.
Pokud by například jeden router s MTU 1500 poslal 1500B dlouhý paket přes zařízení s MTU 900, IS-IS si s tím umí defaultně poradit a rozdělit frame, ale protože router s MTU 900 nebyl původnem paketu, nemůže ho upravit a zahodí ho.
Pokud nemáme shodné MTU v celé aree, můžeme nastavit specificky pro IS-IS nejvyšší možnou velikost framu, která musí být rovna nejnižšímu MTU v arei.


