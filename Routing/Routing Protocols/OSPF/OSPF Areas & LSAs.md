# OSPF Areas & LSAs
---

## OSPF Areas Design 
---
OSPF design z důvodů rychlejší konvergence a vyšší scaleability umožňuje rozdělit celou síť do několika Areí, které spolu navzájem komunikují za speciálních podmínek.
Arei jsou číslované od `0` do `4294967295` nebo v IP formátu, speciální síť je `area 0`, také nazývaná ***Backbone***, jedná se o pomyslné jádro sítě, do kterého musí mít přístup všechny ostatní arei.
Router, který propojuje jednotlivé arei, má tedy interface ve více, než jedné se nazývá ***Area Border Router*** (ABR), kdežto router pouze v jedné arei je ***Internal Router***, pokud je tato area Backbone, jedná se i o ***Backbone Router***. 
Router na hranici Routing Protocolu, který provádí redistribuci z jiného protokolu, nebo jiné OSPF instance se nazývá ***Autonomous System Boundary Router*** (ASBR).

>Nutno podotknout, že v [RFC 3509](https://datatracker.ietf.org/doc/html/rfc3509) není specificky určeno, že ABR musí mít alespoň jeden interface v Backbone arei, i přes to v Cisco implementaci je router ABR pouze tehdy, pokud má interface přímo v Backbone arei, nebo do ní má spoj pomocí Virtual Linku. 
>Toto pravidlo se může měnit na základě implementace.

OSPF router vždy uchovává separátní LSDB pro areu, ve které se nachází, Internal Routers tedy mají pouze jednu LSDB, kdežto ABR jich mohou mít více. Tyto tabulky jsou oddělené a informace z jedné se nemohou jakýmkoli způsobem dostávat do dalších, je úkolem ABR všechny tyto informace zabudovat do RIB a následně je sdílet mezi areami v kontrolované formě.
[[OSPF SPF|SPF]] ABR provádí pro každou LSDB separovaně a výsledné cesty jsou podrobovány [[#OSPF Path Preference]].

Používání různých areí, krom pouze jedné, přináší řadu výhod:

- Menší LSDB pro Internal Routers, méně potřebné paměti
- Rychlejší [[OSPF SPF]]
- Selhání linky vyžaduje pouze částečné SPF výpočty v jiných areách
- ABR jsou výborným bodem pro sumarizaci a filtrování

## OSPF Path Selection 
---
OSPF metrika se vypočítává jako:

$$
C=\frac{100Mb}{Bandwith}
$$
Cena se připočítává na výstupním interfacu.

V některých případěch ale OSPF vybírá cestu nejenom na základě metriky:

- OSPF vždy preferuje cestu do sítě, která vede uvnitř arei, i přes horší metriku
- V případě externích cest OSPF preferuje E1/N1 cesty nad E2/N2
- Následně se rozhoduje na základě Costu
	- Pokud je Cost stejný, zabudují se obě
		- Pokud se ale jedná o E2, tedy Cost ke určen dopředu, příchází do hry ještě *Forwarding Cost* ->

> Specifický případ nastává u šíření E2 cest, u kterých je dopředu určen *Cost* a neměl by se tedy měnit. SPF ale stále potřebuje způsob, jak najít nejoptimálnější cestu, navíc, pokud by se metrika neměnila, tak by mohli existovat 10 - 100 možných cest do takovéhoto místa, proto u těchto cest OSPF má takzvannou *Forwarding Cost*, i přesto, že metrika cesty jako taková se u E2 cest nemění, SPF stále určuje cestu do dané sítě pomocí vzdálenosti mezi ním a routerem distribuujícím danou síť.

> ```
>*>  0.0.0.0/0, Ext2, cost 1, fwd cost 3, tag 1
>     SPF Instance 28, age 00:00:00
>     Flags: RIB
>      via 10.26.27.2, GigabitEthernet0/0
>      Flags: RIB
>       LSA: 5/0.0.0.0/31.31.31.31
>      via 10.26.29.2, GigabitEthernet0/1
>       Flags: RIB
>       LSA: 5/0.0.0.0/30.30.30.30
>```

> Zde je vidět, že metrika cesty *cost* zůstává stejný (1), ale přidává se hodnota *fwd cost*, která je na hodnotě 3, toto je cena cesty mezi lokálním routerem a routerem, který poslal daná LSA, v tomto případě 2 routery (`30.30.30.30` a `31.31.31.31`).
> Pokud zvýšíme Cost jednoho z výstupních interfaců (`G0/0`), pak, kvůli zvýšené *fwd cost*, vypadne cesta z LRIB.

> ```
> *>  0.0.0.0/0, Ext2, cost 2, fwd cost 3, tag 1
>     SPF Instance 29, age 00:00:00
>     Flags: RIB
>      via 10.26.29.2, GigabitEthernet0/1
>       Flags: RIB
>       LSA: 5/0.0.0.0/30.30.30.30
>```

> Pokud zvýšíme cenu i `G0/1`, pak se nám vrátí i druhá cesta, protože budou mít stejný *fwd cost*, a ten se zvýší.
> (Aby se změnila topologie, je třeba přepočítat *SPF* příkazem (`clear ip ospf force-spf`))

>```
> *>  0.0.0.0/0, Ext2, cost 2, fwd cost 4, tag 1
>     SPF Instance 31, age 00:00:02
>     Flags: RIB
>      via 10.26.27.2, GigabitEthernet0/0
>       Flags: RIB
>       LSA: 5/0.0.0.0/31.31.31.31
>      via 10.26.29.2, GigabitEthernet0/1
>       Flags: RIB
>       LSA: 5/0.0.0.0/30.30.30.30
>```

Mezi areami se OSPF chová jako [[Routing#Distance-Vector|Distance-Vector]] protokol, OSPF nepoužívá tradiční loop-avoidance prvky, ale používá koncepty podobné, například [[RIP#Split Horizon|Split Horizon (RIP)]].

Split Horizon je používáno například u type 3 LSAs, kdy ABR blokuje tyto LSAs, pakliže obsahují sítě již obsažené uvnitř arei, zároveň ABR nepoužívá pro [[OSPF SPF]] type 3 LSAs, které nepřišli z Backbone arei.
Dalším opatřením je, že ABR nikdy nepoužije jako cestu do sítě vně arei next-hop uvnitř arei.

![[OSPF_LoopPrevention.png]]

## LSAs
---
Různé typy paketů, respektive původů LSA informací, napomáhají multi-area designu OSPF.
Pouze router, který je původcem určité LSA zprávy ji může upravit nebo smazat, příjemci ji mohou pouze zpracovat a přeposlat všem ostatním, nikdy ji nemohou upravovat, jedinou vyjímkou je vypršení životnosti zprávy.
Toto pravidlo sice umožňuje všem zařízením uvnitř arei mít shodné LSDB, ale velmi limituje použití sumarizace a filtrovacích technik, jelikož většina routerů pouze přeposílá zprávy.

|LSA Type|Jméno|Popis|
|:-:|:-:|:-:|
|[[#Type 1 2\|1]]|Router|(Každý router, per-area) Tato zpráva obsahuje RID a IP adresy všech interfaců v dané oblasti, včetně *Stub* sítí, přeposílána pouze v rámci arei.|
|[[#Type 1 2\|2]]|Network|(DR, per-transit network) Vytvořena DR obsahující informace o subnetu a všech připojených zařízeních, přeposílána v rámci segmentu.|
|[[#Type 3\|3]]|Net Summary|(ABR, per-area) Vytvořena ABR obsahující informace o sítích a metrice z jedné arei a přeposílána do další, neobsahuje topologické informace, pouze nabízí sebe, jako next-hop, přeposílána v rámci arei a dále na ABR.|
|[[#Type 4 & 5 + External Route Types 1 & 2\|4]]|ASBR Summary|(ASBR, per-area) Vytvořena ASBR a  přeposílána ABR obsahující informace o cestě k ASBR, přeposílána v rámci arei a dále na ABR.|
|[[#Type 4 & 5 + External Route Types 1 & 2\|5]]|AS External|(ASBR, OSPF doména) Vytvořena ASBR obsahující redistribuované sítě do OSPF, přeposílána v celé OSPF doméně.|
|6|Group Membership|Používána pro MOSPF, nepoužívána na Cisco IOS|
|7|NSSA|(ASBR, intra-area) Vytvořeno v NSSA, jedná se o kamufláž pro LSA 5, ABR na hranici NSSA přepošle LSA 7 jako LSA 5.|
|8|External Attributes|Vytvořeno na ASBR při BGP-to-OSPF redistribuce pro zachování BGP atributů uvnitř redistribuované sítě, není používáno na Cisco routerech.|
|9-11|Opaque|Používáno jako generická LSA pro budoucí použití, například LSA 10 se používá pro *MPLS Traffic Engineering*. Tyto typy mají rozdílné scopy přeposílání, například LSA 9 je *link-local*, LSA 10 je *area-local* a LSA 11 je *AS-local*.|

### Transit network

Jedná se o síť, ve které je 2 a více OSPF-enabled zařízeních, které navázali sousedství a zvolili DR/BDR, provoz tedy může téct z jednoho bodu do dalších. 
Jinými slovy se jedná o díť, do které existuje více vstupních a výstupních bodů.
Toto pravidlo je lehce ohnuto P-t-P spoji, toto spojení je OSPF bráno jako kombinace P-t-P linky a Stub IP sítě.

### Stub network

Jedná se o síť, ve které router nenavázal žádné sousedství a je tedy do ní pouze jedna vstupní a výstupní cesta, typicky koncové zařízení nebo loopback.

### Type 1 & 2

Každý router generuje vlastní LSA 1, ty obsahují informace o interfacech routeru, které jsou uvnitř arei, jejich IP adresy a list na nich sousedících routerů. LSA je identifikovatelné pomocí *link-state ID* (LSID), které se rovná RID.

LSA 2 je generováno DR a má za úkol sumarizovat informace o [[#Transit network]], na které může být více zařízení, než, aby každé muselo posílat LSA 1 každému, prostě každý router pošle LSA 1 DR & BDR, DR je zpracuje, sesumarizuje, a jako LSA 2 přepošle všem zařízením na segmentu.

Zároveň každé LSA identifikuje bod v grafu pro [[OSPF SPF]] algoritmus, LSA 1 identifikuje každý router, jeho sousedy a připojené sítě, kdežto LSA 2 reprezentuje celý segment, síť, na který je připojeno několik zařízení, LSA 2 je někdy nazýváno jako *pseudonode*, protože obsahuje více, než jedno zařízení, ale v kombinaci s konkrétními LSA 1 každého připojeného zařízení, které DR & BDR mají, SPF umožňuje sestavit přesný graf sítě.

V případě potřeby odstranění LSA z LSDB, například z důvodu vypnutí sítě, existují dva způsoby. Buď se přepošle nové LSA bez problémového záznamu, nebo je celé LSA předem zastaráno pomocí nastavení časovače na 3600s a přeposláním, oba tyto způsoby vedou k okamžitému vymazání ze všech LSDB.

### Type 3

ABRs nepřeposílají type 1 a 2 LSAs z jedné arei do druhé, místo toho generují type 3 LSAs, které jednoduše popisují sítě, masky a ABR's metriku do sítí v dané sousední arei a nabízí sebe, jako next-hop, podobně, jako [[Routing#Distance-Vector|Distance-Vector]] protokoly.

Celková metrika do těchto sítí se pak skládá z reportované metriky ABR a metriky k ABR, díky tomu je [[OSPF SPF]] značně optimalizované, protože v případě selhání cesty uvnitr arei musí celá area provádět kompletní SPF kalkulaci, kdežto vně arei stačí pouze na základě upravené metriky pouze zkontrolovat nejbližší cestu k ABR, čímž se zmenčila failure doména, tomuto omezenému výpočtu se říká *partial SPF*.

ABR zpracovává pouze type 3 LSAs, které dostal přes backbone areu, ostatní nejsou používané pro SPF výpočet a nejsou tedy ani zahrnuty v jeho vlastních type 3 LSAs, přesto jsou zapsány v LSDB a přeposílány v rámci příchozí non-backbone arei.
Z toho vyplývá, že inter a intra area routery a sítě jsou do non-backbone arei přeposílány pouze z backbone, což navazuje na dřívější pravidlo, že router musí mít buď přímý spoj, nebo Virtual Link spoj do backbone arei, není tedy možné OSPF Area Design větvit do nekonečna, reálně jsou možné pouze 3 vrstvy areí.

> Type 3 & 4 LSAs jsou v RFC nazývány jako *summary*!

### Type 4 & 5 + External Route Types 1 & 2

OSPF umožňuje dělit redistribuované, externí, cesty do 2 kategorií:

- *External type 1* (E 1)
	- Metrika této cesty se, stejně, jako u ABR, skládá z externí a vnitřní metriky
	- Vhodné použít v případě více cest k ASBR
- *External type 2* (E2)
	- Metrika této cesty se skládá pouze z externí hodnoty
	- V případě, že do stejné sítě existuje více cest se stejnou metrikou, použije se nejbližší ASBR dle OSPF metriky, pokud i tak je více výstupních bodů, použíjí se všechny (do maximálního počtu cest)

Cisco IOS defaultně redistribuuje dle E2 metriky, určení typu je ve výsledku ale na adminitrátorovy a jeho úvaze.

ASBR při redistribuci cesty vytváří type 5 LSA obsahující síť, metriku a typ metriky (E1/E2). Příjemce si zároveň uchovává metriku k ASBR, kterou využíje na základě typu externí cesty, v rámci stejné arei to není problém, metriku si zařízení zjistí pomocí LSA 1 & 2, ale problém nastává s metrikou mezi areami.
V tuto chvílí se zapojuje ABR, který přijímá type 5 LSA a přeposílá ho dál do dalších areí, spolu s type 4 LSA, které obsahuje jeho metriku k ASBR, od kterého pochází type 5 LSA, a ASBR\`s RID.
Pokud se jedná o E1 cestu, koncový router následně k E metrice přičte type 4 LSA metriku, z paketu se shodným RID, a jeho metriku k ABR.

## Stubby Areas
---

Stejně, jako [[#Stub network]], ani Stubby area má pouze jeden vstupní a výstupní bod, OSPF umožňuje využít tohoto faktu a dále zjednodušit [[OSPF SPF|SPF]] proces, protože tyto arei, nehledě na velikost skutečné domény, budou mít vždy pouze jeden výstupní bod, a tak umožňuje nahradik kompletní LSDB defaultní cestou.

Naopak ostatní typy areí, non-stubby, jsou označovány jako ***regular*** nebo ***standard***.

### Stubby Area

Standardní *stubby area* tento koncept zahrnuje pouze na ASBR a externí cesty, protože area může být spojená s dalšími areami, ale přesto mít pouze jeden výstupní bod z OSPF domény.

V arei nastavené jako *stubby* bude tedy ABR blokovat veškeré type 4 & 5 LSAs, internal-routers budou tyto LSAs ignorovat, pro případ, že ASBR je přímo uvnitř arei a ABR bude generovat defaultní cestu s ním jako next-hopem pomocí type 3 LSAs.
Díky tomu budou mít vnitřní routery stále konektivitu do non-OSPF domény, ale nebudou muset zabudovávat všechny externí routy.

> [RFC 2328](https://datatracker.ietf.org/doc/html/rfc2328) přímo specifikuje blokování pouze type 5 LSA, ale vzhledem k tomu, že použití type 4 LSA bez type 5 je zbytečné, většina implementací blokuje obě.

### Totally stubby Area

V případě jediného výstupního bodu ABR opět není nutné, aby routery uvnitř arei znaly celou topologii, ABR v *TS Area* negeneruje type 3 LSAs se všemi známými sítěmi, ale všechny je nahrazuje defaultní cestou, včetně cest OSPF.

### Not-so-stubby Area

V případě, že potřebujeme umístit ASBR uvnitř stubby arei, defaultně, vzhledem k tomu, že stubby area blokuje type 4 & 5 LSAs, ASBR by nefungoval.
Nastavením arei na NSSA nebude ASBR distribuovat externí cesty pomocí type 5 LSAs, ale type 7, poté co tyto type 7 LSAs dojdou na ABR, ABR s nejvyšším RID je přepne zpět do stavu type 5 LSA, spolu s generováním type 4 LSA a externí cesty je tedy možné distribuovat v OSPF doméně.
Routery uvnitř NSSA zpracovávají type 7 LSA a jejich cesty zabudovávají jako *N1/2* routy.

Zároveň se jedná o jedinou *stubby* areu, u které ABR nevytváří automaticky defaultní cestu, pro její generování je potřeba na ABR nastavit `default-information-originate`.

### Totally NSSA

Tato area kombinuje funkce NSSA a TS, plně distribuuje externí cesty spolu s vytvořením defaultní cesty pro areu do zbytku OSPF domény.

## Virtual Link

ABR musí mít vždy přístup do Backbone arei a stejné arei musí mít vždy přímé spoje mezi sebou, toto jsou podmínky, které není vždy možné splnit, ať už kvůli designu sítě nebo kvůli selhání vnitřních linek v arei.
OSPF nabízí **Virtual Link**, který vytvoří OSPF spoj přes IP síť, nejedná se o VPN ani tunel, je to pouze IP směrovaná sessiona.
Area, přes kterou Virtual Link session prochází je označována za *transit* a musí být normální areou (ne Stubby).
Z pohledu OSPF Virtual LInk vytváří session, která umožňuje routeru mít interface v Backbone area, a tak má i Area 0 LSDB obsah.

![[virtual_link.png]]



