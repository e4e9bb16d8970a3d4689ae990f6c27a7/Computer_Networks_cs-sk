# Diffusing Update Algorithm (DUAL)
---

DUAL je jádro EIGRP procesu, které zajišťuje konvergenci sítě na *loop-free*  topologii.

## Topology Table
---
Jedná se o centrální tabulku pro ukládání dat o síti, respektive vzhledem k tomu, že EIGRP je distance-vector protokol, spíše o cestách.
Obsahuje všechny důležité informace, se kterými DUAL pracuje:
- Adresy sítí
- *Feasible Distance* sítí
- Adresy next-hop souterů spolu s egress interfacy
- Metriku, kterou posílají sousedé
- Vlastní metriku
- Stav cílové sítě
- Další informace typu různých označení sítý, jejich typů a pod.

Je sestavována nad přímo připojenými sítěmi, redistribuovanými sítěmi a z příchozích paketů EIGRP.
Pro každou síť získanou skrze EIGRP si proces najde souseda s nejnižší celkovou metrikou a ověří, zda se bude jednat o loop-free topologii, pokud ano, tuto cestu zapíše do RIB (s upravenou metrikou [[EIGRP Metrika]]).

Tabulka může obsahovat sítě 2 typů:

### Active

Tento stav značí síť, do které ještě není určena nejlepší loop-free cesta a je tedy nezbytné provést další výpočty.

### Passive

Tento stav značí, že cesta do této sítě je určena a EIGRP ji nemá potřebu měnit, plně konvergovaná síť by měla mít pouze Passive záznamy.

### Metrika

Topology Table může mít více cest do jedné cítě, v takovém případě se rozhoduje na základě lokálně spočítané ***Computed Distance*** (CD), která se skládá z metriky získané od souseda, ***Reported Distance*** (RD), kterou si router spočítá z informací, které mu soused poslal, + metriky spoje k sousedovi.

V terminálu IOS je po zadání příkazu `R#show ip eigrp topology all-links` zobrazena tímto stylem:

```R
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply, r - reply Status, s - sia Status

P <NETWORK>/<PREFIX>, 1 successors, FD is <CD>, serno 4

	via <Next-Hop> (CD/RD), <eg_INT>
	via <Next-Hop> (CD/RD), <eg_INT>
	via <Next-Hop> (CD/RD), <eg_INT>
```

#### Feasibility

Dalším specifickým vyjádřením metriky je ***Feasible Distance*** (FD), často se uvádí, že se jedná o nejnižší a používanou CD, to je ale pravda pouze z části.

FD je záznam nejlepší metriky pro každou destinaci od doby vytvoření cesty, jejího přesunu z Active stavu na Passive.
Jinými slovy se jedná o historicky nejnižší cenu cesty do určité sítě, z tohoto důvodu nemusí být FD nutně shodná se CD, protože je možné, že například změnou metriky, ať už v CD nebo RD dojde ke zvýšení CD, FD zůstane stále rovno historickému minimu.
Naopak, pokud se CD sníží, sníží se i FD.

Pokud se ale, v ten čas, nejlepší CD metrika zvýší natolik, že je k dispozici lepší, dojde k přepočítání cesty, a to i pokud tato nová CD je horší, než FD.

FD je pouze pomocná hodnota pro vybírání loop-free topologie v případě selhání primární cesty, nikdy není posílána v paketech, ani není na jejím základě vytvářená RIB.

##### Feasibility Condition

Pro snížení doby odstávky sítě na minimum se určují "záložní" cesty do určité sítě, tyto záložní cesty, respektive next-hopy jsou nazývané jako ***Feasible Successor*** (FS).
Pro jejich vytvoření je potřeba splnit FC podmínku, aby bylo naprosto jasné, že jejich použitím se nevytvoří smyčka:

$$
RD < FD
$$
Jinými slovy metrika, kterou nám posílá soused, musí být nižší, než naše nejnižší zaznamenaná metrika, v případě, že je jeho metrika vyšší, může dojít k tomu, že on nám sice nabízí sebe jako next-hop do sítě, ale on sám považuje nás za jeho next-hop do oné sítě.
V takovém případě by byla jeho RD zákonitě vyšší, nebo rovná naší FD.

Ne vždy vyšší RD znamená smyčku, ale vždy nižší RD znamená loop-free cestu.

V případě, že není splněna FC a není tedy žádný FS, je nutné znovu vytvořit cestu do sítě.

## Topology Change
---

Topology Change se projevuje v několika případech:

- Změní se metrika do dané sítě
- Objeví se nový soused poskytující cestu do dané sítě
- Vypadnutí cesty do dané sítě

O změně sítě se může zařízení dozvědět několika způsoby:

- Pomocí paketů
	- [[EIGRP Pakety#Update|Update]]
	- [[EIGRP Pakety#Query|Query]]
	- [[EIGRP Pakety#Reply|Reply]]
	- [[EIGRP Pakety#SIA-Query|SIA Query]]
	- [[EIGRP Pakety#SIA-Reply|SIA Update]]
- Změnou lokálních metrik
- Vypadnutím spoje

Zároveň zde funguje mechanismus [[RIP#Route poisoning|Route Poisoning]], v případě, že vypadne Successor pro danou síť a neexistuje náhrada, jeho metrika je přeposílána s nekonečnou cenou ([[EIGRP Metrika#Delay]]).


V případě změny router řeší, zda nový router s nejnižší CD je FS, dle [[#Feasibility Condition]],  to zjistí tak, že pro jistotu, kdyby změna ovlivnila i jeho, pošle Request dotazující se na nejaktuálnější metriky do dané sítě, nad kterými provede FC a zjistí, zda se jedná o FS, pokud ano, řídí se následujícími kroky:
1. FS s nejnižší CD je nový Successor
2. Pokud je současná CD nižší, než FD, FD se upraví, pokud není, zůstane stejná
3. Dojde k úpravě RIB
4. V případě změny CD router přepošle Update paket s novou metrikou všem sousedům

>*Tento proces se nazývá **Local computation**, protože probíhá pouze lokálně na základě informací, které již má router uložen v Topology Table, nedochází ke koordinaci se sousedy, pouze aktualizaci informací, je li třeba.
Po dobu této změny zůstává cesta v Passive stavu, pouze se mění její vlastnosti.*

pokud není, řídí se následujícími kroky:
1. Problémový záznam v RIB je uzamčen, nemůže být smazán ani nijak změněn do doby dokončení algoritmu a přepnutí stavu cesty na Passive
	- Toto může vést ke krátkodobému výpadku sítě, ale zaručuje loop-free topologii
2. Router změní FD na současnou CD Successora, stále ten starý, může ji i zvýšit
	- Pokud potřebuje router přeposlat metriku sítě ve stavu Active, přepošle také tuto hodnotu CD
3. Router začne posílat Query pakety obsahující prefix Active sítě/í a současné CD

Sousední router přijímající Query zkontroluje metriku požadované sítě a mohou nastat 3 scénáře:
1. Router má Successora nebo FS
	- V tomto případě router jednoduše přepošle jeho vlastní CD metriku do dané sítě
	- Původní router si poté, co mu příjdou všechny odpovědi, vybere nabízenou cestu s nejnižší CD, převede záznam na Passive a už nekontroluje FC
2. Router ztrátí cestu do sítě, protože Query přišlo od jeho Successora a nemá FS
	- V tu chvíli se začne chovat jako původní router a začne rozesílat vlastní Query
	- Tento proces se opakuje, dokud Query nedojde k někomu, kdo má cestu do dané sítě
		- V případě, že router dostane Reply s danou cestou, začne ho šířit pomocí Update paketů
3. Router nemá cestu do sítě a nemá (již) komu poslat Query
	- V takovém případě odešle Reply s tím, že cesta neexistuje

> *V tomto případě se jedná o **Diffusing Computation**, do hledání nejlepší cesty do sítě je zapojeno více routerů, třeba i celá síť. * 

## DUAL FSM
---

Předešlý proces funguje velmi dobře za předpokladu, že se v době probíhajícího výpočtu v síti nenastane další změna, která by změnila výpočet.
Toto nelze vždy zajistit, a proto EIGRP implementuje *Diffusing Update Algorithm Finite State Machine* (DUAL FSM).

Logický princip tohoto algoritmu je poměrně složitý a komplikovaný, ale hlavně ho není potřeba znát do detailu pro získání CCIE R&Sv7, proto jsou zde shrnuta pouze pravidla, kterými se zařízení řídí.
Pro detailnější informace můžete vyhledat [RFC EIGRP](https://datatracker.ietf.org/doc/html/rfc7868#section-3.5).

>Pokud nastane změna metriky a soused s nejnižší CD stále splňuje FC, cesta zůstává v Passive módu.

>Pokud je Query dostáno od současného Successora a po započítání nové metriky soused s novou nejnižší CD nesplňuje FC, router se přepne do $A_3$ (*Successor Origin*) Active stavu.
>Router začne posílat vlastní Query a čekat na Replies, pokud v době čekání na Replies nedojde k dalšímu zvýšení metriky, router po dostání posledního Reply zvolí souseda s nejnižší CD za nového Successora.

>Pokud je detekována změna jinak, než Query od Successora (Update, změna metriky interfacu, vypadnutí souseda), a poté, co, po přepočítání metrik, soused s nejnižší CD nesplňuje FC, router se přepne do $A_1$ (*Local Origin*) Active stavu.
>Router začne posílat vlastní Query a čekat na Replies, pokud v době čekání na Replies nedojde k dalšímu zvýšení metriky nebo příjmutí Query od Successora, router po dostání posledního Reply zvolí souseda s nejnižší CD za nového Successora.

>Pokud v průběhu stavů $A_1$ nebo $A_3$ dojde ke zvýšení metrik jinými způsoby, než pomocí Query od Successora, muselo dojít k další změně v topologii. Router tuto změnu nemůže přeposlat, protože síť se nachází v Active módu a je "uzamknuta", proto musí dojít k dalšímu výpočtu, to se provádí tak, že $A_3$ router se přepne na $A_2$(*Multiple Origins*) a $A_1$ ne $A_0$(*Local Origin with Distance Increase*).
>Po příchodu všech Replies router zkontroluje, zda soused s nejnižší CD splňuje FC s původní FD, kterou rotuer měl, když vstoupil do Active stavu, ne se zvýšenou, která přišla v průbehu výpočtu.
>>Pokud splňuje FC, síťové informace se aktualizují a rotuer ji přepne zpět do Passive módu.
>
>>Pokud nesplňuje FC, router se přepne zpět do $A_1$ nebo $A_3$ stavu a začne nové kolo výpočtu (rozesílání Replies), tentokrát s novou FD.

> Pokud v průběhu stavů $A_1$ nebo $A_0$ router dostane Query od Successora, znamenáto, že došlo ke změně v topologii.Router tuto změnu nemůže přeposlat, protože síť se nachází v Active módu a je "uzamknuta", proto musí dojít k dalšímu výpočtu, to se provádí tak, že se router přepne do $A_2$ a následně se chová, jako v předchozím příkladě.

## Stuck-In-Active
---

Při připojení routeru do Diffusing algoritmu (přepnutí sítě do Active stavu), router rozešle Queries a až po příjmutí zpět všech Replies může provést vlastní vyhodnocení nejlepší cesty, celý proces se zkomplikuje v případě, že naše rozeslané Query přepne souseda také do Active stavu a také tedy rozešle Queries, v tomto případě nečekáme pouze na Reply od souseda, ale, vzhledem k tomu, že on ho pošle až poté, co jemu příjdou všechny Replies, na Replies od jeho souseda a tak dále...
Tento řetězec může být dlouhý, teoreticky i celá síť, a jeden neodpovídající router může celý algoritmus shodit, důvodů pro nefunkčnost routeru je více:

- Přetížení CPU a router nemůže včas zpracovávat EIGRP pakety
- Pakcet loss
- Ohromná velikost topologie sítě nebo její vysoká komplexnost

proto EIGRP nabízí mechanismus, jak se s takovou eventualitou, někdy nazývanou EIGRP Achilovou patou, vypořádat.

Při přepnutí sítě do Active stavu začne router  odpočítávat *Active* timer, defaultně je nastaven na 3 minuty.
Po uplynutí tohoto času  bude neodpovídající cesta, soused, označena za *Stuck-in-Active* (SIA), bude s ním rozvázáno sousedství a do výběru nejlepší cesty se bude tato cesta brát s *Infinite* metrikou.

Toto sice řeší prvnotní problém, ale představuje řadu dalších:

- Rozvázání sousedství
	-  Může dojít k odstranění i jiných cest
	-  Po Dalším příchozím Hello paketu se znovu naváže
- Obtížnost diagnostiky
	- Vzhledem k tomu, že označení za SIA je založeno pouze na časovači, v případě řetězce neodpovídání může dojít k přerušení komunikace i se zařízením, které není vadným, pouze čeká na vlastní odpověď, kterou nedostal, celá síť se tak může destabilizovat a vznikají tak problémy s konvergencí

SIA problémy je z těchto důvodů extrémně těžké diagnostikovat.

Pro jednoduchost diagnostiky a zlepšení stavility sítě v případě problémů SIA se v další implementaci tento postup poupravil.
Pokud v polovině Active timeru nedostane router Reply, odešle [[EIGRP Pakety#SIA-Query|SIA-Query]] z důvodu ověření korektní konektivity sousedního zařízení, pokud router odpoví s [[EIGRP Pakety#SIA-Reply|SIA-Reply]], nemá smysl rozvazovat sousedský vztah s ním, protože chyba není v něm.
Pro případ, že sousední router již poslal Reply, ale to se ztratilo, SIA-Reply může mít 2 verze:

- Stále očekává na vlastní Reply a dává vědět, že on je v pořádku
- Již došlo k určení cesty a posílá vlastní metriku

Příjmutí SIA-Relpy umožňuje routeru vyresetovat Active timer, ale pouze 2x, v případě, že se odešle a získá 3. SIA-Query - SIA-Reply, nedojde j resetování Active časovače, a pokud tedy nedojde v odeslání původního Reply do uplynutí druhé půlky Active timeru, dojde k rozvázání sousedství a počítání s Infinite metrikou.

Ve výsledku tedy tento princip umožňuje rozšířit čas až na 360s, nebo správně určit SIA router.

Celému problému se SIA se lze vyhnout využíváním [[EIGRP Stub]].








