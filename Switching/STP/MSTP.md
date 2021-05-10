# Multiple Spanning Tree Protocol
---

Byl vyvinut kooperací Cisca a IEEE jako nástupce Cisco Proprietárního protokolu MISTP, který je již považován za zastaralý a není podporovaný.

Jako jádro využívá [[Rapid STP]] se všemi jeho funkcemi.

Hlavním rozdílem oproti [[Per-VLAN STPs]] je jeho otevřenost, doposud zde byly proprietární protokoly, [[Per-VLAN STPs#PVSTPs|PVSTPs]], které neměly širokou podporu napříč výrobci. MST je otevřený standard, prvotně 802.1s později spojen do [IEEE 802.1Q-2005](https://standards.ieee.org/standard/802_1Q-2005.html).
Další nespornou výhodou je princip instancí, u PVST+ se vytváří instance, a tudíš i BPDUs, pro každou VLANu, počet těchto instancí je ale většinou omezen na 128 a pro další nefunguje STP.

MST tomuto problému předchází vytvářením ***VLAN to instance*** map číslovaných od 0 po 4096, i přesto, že aktivních, posílajících BPDUs, může být maximálně 16, ve standartu až 64, najednou, tak se předchází problému maximálních 128 VLAN, protože v jedné instanci může být VLAN více, klidně i všechny.

Další výhodou je i snížení zátěže, protože se vždy posílá pouze jedno BPDU v [[MSTP#IST|IST]] instanci a ostatní instance jsou v něm zanořené jako M-Recordy, odtud i pochází maximální počet 64 instancí, protože MTU je standatně 1500B.

Další výhodou je škálovatelnost, protože MST se dělí do regionů.

## MST Regions
---

MST region je koncept bodobný [[Routing#Terminologie#Obecné termíny|AS]] u routingu, jedná se o topologii pod jednotnou správou, může jich být více uvnitř jedné domény.
Pro vytvoření regionu musí mít switche shodných několik hodnot:
- Name
	- Defaultně prázdné, což je validní hodnota
	- maximálně 32 znaků
- Revision
	- 0 - 65535
	- Určuje verzi konfigurace
	- Oproti [[VTP]] si ho administrátor určuje sám
- VLAN to instance map
	- Musí mít shodný počet instancí a VLANy do nich přiřazené

### Intra-Region

Jedná se o vytváření uvnitř regionu.
Switche vytváří topologii pro všechny [[MSTP#MSTI|MSTIs]] a funguje tak zde Per-VLAN, respektive Per-Instance Spanning-Tree.

### Inter-Region

Jedná se o vytváření mezi regiony.
Switche vytváří topologii pouze pro [[#CIST]] a funguje tak pouze jako CST, tedy existuje pouze jediná logická topologie.
Regiony se mezi sebou navzájem vnímají jako logické switche a jejich vnitřní topologii neřeší.
Sice se zde posílají v BPDUs M-Records, na to pozor, často se píše, že se neposílají, ale switche na ně nereagují.

## Terminologie
---

### CST

*Common Spanning Tree*

Jedná se o pouze jednu instanci STP, typicky, pokud nepoužíváme žádné další, krom defaultních, **VLAN to instance mapy**.
Za CST se označuje STP mezi regiony, vzhledem k tomu, že tam se vytváří pouze jedna instance.

### IST

*Internal Spanning Tree* / *Common Internal Spanning Tree*

Jedná se o defaultní [[MSTP#MSTI|MSTI 0]] instanci, která je vždy funkční. Bývá označována i jako CIST.
Jako jediná posílá BPDUs, v rámci kterých jsou [[MSTP#M-records| M-Recordy]] pro každou [[MSTP#MSTI|MSTI]] instanci.
Defaultně do ní spadají všechny VLANy a pokud neexistují žádné další, tak MST funguje v módu [[#CST]].
Vzhledem k tomu, že jako jediná posílá BPDUs, tak její nastavení timerů se vztahuje na všechny ostatní.

### MSTIs

*Multiple Spanning Tree Instances*

Jedná se o další instance, maximálně 16, od 0 - 4095.
Jsou přepravovány uvnitř [[#IST]] BPDU jako [[MSTP#M-Records|M-Recordy]].

### CIST

*Common and Internal Spanning Tree*

Z pohledu sítě se jedná o celou MST doménu, IST + MSTIs uvnitř regionu a CST z vně regionu.

#### CIST Root

Jedná se o switch, který má nejnižší [[STP Terminologie#Bridge ID|Bridge ID]] v celé CIST topologii, tedy MST doméně.

#### CIST Regional Root

Jedná se o *boundary* switch zvolený v každém regionu na základě nejnižší cesty do CIST Root.
Je shodný s Root Bridgem IST (MSTI0) a musí se jednat o boundary switch, tedy musí sousedit s jiným regionem.

Vzhledem k tomu, že CST mezi regiony hledí na jednotlivé regiony jako na loop-free switche, musí si i určit [[STP Terminologie#Porty#Role|porty]].
Určování funguje jako u klasického [[Rapid STP|RSTP]] s tím, že funguje pouze s boundary porty jednotlivých regionů.
Jediná změna je ta, že [[STP Terminologie#Role|RP]] si určí CIST Regional Root, ostatní porty jsou [[STP Terminologie#Role|Designated]], [[STP Terminologie#Role|Alternate]] nebo [[STP Terminologie#Role|Backup]].

## BPDU
---

|BPDU Pole|Délka|
|:--------:|:------:|
|Protocol ID|2|
|Protocol Version|1|
|BPDU Type|1|
|[[#CIST Flags\|CIST Flags]]|1|
|[[#CIST Root Identifier]]|8|
|[[#CIST External Root Path Cost]]|4|
|[[#CIST Regional Root ID]]|8|
|[[#CIST Port ID]]|2|
|[[STP Terminologie#Message Age\|Message Age]]|2|
|[[STP Terminologie#Max Age\|Max Age]]|2|
|[[STP Terminologie#Hello Time\|Hello Time]]|2|
|[[STP Terminologie#Forward Delay\|Forward Delay]]|2|
|[[STP Terminologie#Version 1 length\|Version 1 Length]]|2|
|[[MSTP#Version 3 Length\|Version 3 Length]]|4|
|MST Extension|Version 3 Lenght|
|MST Configuration ID|2|
|[[#MST Config Name]]|32|
|[[#MST Config Revision]]|2|
|[[#MST Config Digest]]|16|
|[[#CIST Internal Root Path Cost]]|4|
|[[#CIST Bridge ID]]|8|
|[[#CIST Remaining Hops]]|1|
|[[#M-Records]]|$x \times 16$|

### CIST Flags

Jedná se o pole shodné s [[STP Terminologie#Flag|Flag]], například ve WireSharku je pojmenováno prostě *BPDU flags*, podobně jako ostatní hodnoty, ale vztahuje se na [[#IST|IST/CIST(MSTI0)]] instanci.

*BPDU flags*

### CIST Root Identifier

Jedná se o pole shodné s [[STP Terminologie#Root Bridge ID RBID|Root ID]], ale vztahuje se na [[MSTP#CIST]] instanci a označuje tak [[#CIST Root]].

*Root identifier*

### CIST External Root Path Cost

Jedná se o cenu, za kterou lze dosáhnout [[#CIST Root]].
Cena se skládá pouze z [[#CST]], tedy mezi regionálních, linek a upravuje se tedy pouze na *boundary* portech.

*Root path cost*

### CIST Regional Root ID

Jedná se o pole shodné s [[STP Terminologie#Root Bridge ID RBID|Root ID]], ale vztahuje se na [[#CIST]] instanci a označuje tak [[#CIST Regional Root]].

*Bridge Identifier*

### CIST Port ID

Jedná se o identifikaci portu, ze kterého se BPDU poslalo.

*Port identifier*

### Version 3 Lenght

Určuje délku [[#M-Records]](B).

### MST Config Name

Jedná se o jméno regionu, které je nakonfigurováno.

### MST Config Revision

Jedná se o číslo revize regionu.

### MST Config Digest

Jedná se o hash spočítaný nad **VLAN to Instance map**.

### CIST Internal Root Path Cost

Jedná se o cenu cesty k [[#CIST Regional Root]], neboli cesta k [[#IST|IST(MSTI0)]] Root Bridge, která se skládá ze vnitř regionu.

### CIST Bridge ID

Jedná se o [[STP Terminologie#Bridge ID|Bridge ID]] switche, který poslal BPDU pro [[MSTP#IST|IST(MSTI0)]].

### CIST Remaining Hops

Jedná se o převrácenou hodnotu [[STP Terminologie#Message Age|Message Age]], odečítá se od 20, která platí uvnitř regionu.

### M-Records

Může jich být více, až 16 (64).

#### Flag

Určuje [[STP Terminologie#Flag#RSTP|Flag]] pro [[MSTP#MSTIs|MSTI]].

#### Priority

Určuje [[STP Terminologie#Root Bridge ID RBID|RBID]] uvnitř MST instance.

#### MSTID

Určuje číslo instance.

#### Regional Root

Jedná se o [[STP Terminologie#Root Bridge ID RBID|RBID]] RB uvnitř instance.

#### Internal Root Path Cost

Určuje [[STP Terminologie#Root Path Cost RPC|Root Path Cost]] k RB uvnitř instance.

#### Bridge ID Priority

Určuje prioritu Sender Bridge.

#### Port ID Priority

Určuje prioritu portu Sender Bridge.

#### Remaining Hops

Jedná se o převrácenou hodnotu [[STP Terminologie#Message Age|Message Age]], odečítá se od 20, která platí uvnitř instance.

## Volba
---

### CIST Root Bridge

Jedná se o Bridge s nejnižším [[STP Terminologie#Bridge ID|Bridge ID]] z celé MST domény.

### CIST Regional Root Bridge

Jedná se o Bridge s nejnižší [[#CIST External Root Path Cost]], tento switch musí být a automaticky se tak stává i [[MSTP#IST|IST(MSTI0)]] RB.

## Další
---

### STP Dispute link

Jedná se o typ ochrany podobný [[STP Funkce#STP Loop Guard|Loop Guard]], v případě, že switch pošle superior BPDU, ale dostane BPDU s nastaveným Designated bitem, znamená to, že dolní switch nedostal BPDU a linka je jednosměrná, a tak ji označí za *STP dispute*.

### Možná nefunkčnost komunikace

Vzhledem k tomu, že se používá několik VLAN pro jednu instanci, je možné, že, při použitím VLAN pruningu, dojde k povolení portu pro instanci, na kterém je daná VLANa z instance blokovaná, pak nebude fungovat konektivita.

### VTPv3

[[VTPv3 vs VTPv2|VTPv3]] umožňuje rozesílat MST konfigurace. ([[Switching/VLAN/VTP/Konfigurace#MST]])

### Přechod z jiného STP

Vzhledem k tomu, že u MST musí být [[#CIST Root]] uvnitř domény a [[#CIST Regional Root]] uvnitř regionu, pokud MST dostane superiorní BPDU z non-MST protokolu, bude ho ignorovat.
Z tohoto důvodu je v případě přechodu infrastruktury na MST nutné začít od [[Enterprise Network Design#Core Layer|Core layer]].

### MasterPort

Jedná se o Boundary Root port pro MSTIs, tedy je na [[#CIST Regional Root]] Root portu.
Tento port lze vnímat jako Default Gateway, říká, že z tohoto portu je cesta k [[#CIST Root]], ale pouze na [[#IST]], a tak se po něm nebudou přenášet [[#M-Records]].

## Zpětná kompatibilita
---

### CSTPs

Z venku se MST doména jeví jako soubor switchů, přičemž každý region se jeví jako loop-free switch, mezi kterými funguje CST.
Na tomto principu MST funguje i s ostatnímy protokoly, na boundary portu vždy přetváří IST BPDU informace do funkčního BPDU protokolu STP nebo RSTP, které se napodobují, respektive spojující port místo MST použije protější verzi STP.

### PVSTPs

Z venku se MST doména jeví jako soubor switchů, přičemž každý region se jeví jako loop-free switch, mezi kterými funguje CST.
Na tomto principu MST funguje i s ostatnímy protokoly, na boundary portu vždy přetváří IST BPDU informace do funkčního BPDU [[Per-VLAN STPs#PVSTPs|PVSTP(+)]], tato funkce se nazývá *PVST simulation mechanism*.
V praxi to vypadá tak, že se pro každou VLANu pošle BPDU se shodnými informacemi a to IST informacemi a do IST zase přijímá informace pouze z VLANy 1, díky tomu se i non-MST switche můžou zapojit do vytváření stromu, ale pouze jako CST, z toho plyne, že Root musí být shodný pro celou síť a load-balancing je možný tedy pouze uvnitř regionů.
Root tedy musí být shodný pro všechny VLANy uvnitř, nebo vně MST domény, pokud boundary switch dostane superior BPDU pro nějakou VLANu, switch použije [[STP Funkce#BPDU Guard|BPDU Guard]] a zablokuje tento port a přesune ho do *root inconsistence* stavu, v případě starších switchů, v případě novějších použije *PVST Inconsistent designation*. 
I přesto, že toto může zablokovat a kompletně rozdělit síť mezi MST a non-MST, je to nutné pro zajištění loop-free topologie. Tato funkce je také nazývána *PVST simulation check*.
