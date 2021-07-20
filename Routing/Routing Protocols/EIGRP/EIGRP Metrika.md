# EIGRP Metrika
---

EIGRP vypočítává poměrně komplexní metriku se stávající z několika vlastností spoje pro co nejlepší popsání cesty.
Tyto vlastnosti se nazývají *Metric components* a v **Classic** metrice se jedná o:
- [[#Bandwith]]
- [[#Delay]]
- [[#Reliability]]
- [[#Load]]
- [[#MTU]]
- [[#Hop Count]]

**Wide metric** obsahuje:
- [[#Throughput]]
- [[#Latency]]
- [[#Reliability]]
- [[#Load]]
- [[#MTU]]
- [[#Hop Count]]
- [[#Extended Metrics]]

Je možné manuálně upravit určité hodnoty, ze kterých se vypočítává metrika a ovlivnit tak EIGRP proces.
Tyto hodnoty jsou 2: Bandwith a Delay.

Je nedoporučování přenastavovat ručně Bandwith z několika důvodů:
1. K volbě se vždy používá pouze nejnižší bandwith
2. Hodnota interfacu bandwith se používá i pro řadu dalších funkcí a může je tak negativně ovlivnit, například [[QoS]]
3. EIGRP je omezeno pro uživání maximálně 50% bandwithu linky, jeho přenastavení, tedy snížení, nebo naopak zvýšení, může vést k narušení této logiky a nespolehlivosti přenosu

Naopak, pokud chceme ovlivnit EIGRP selekci, je možné použív hodnotu Delay, které nebývá nikde jinde využívána a je kumulativní po celé lince, tudíš je vždy započítána.

## Classic Metric
---

### Bandwith

Jedná se o hodnotu zaznamenanou v ***kbps***, kterou lze nastavit příkazem `bandwith`, nebo si ji IOS "vygeneruje" sám, to funguje dobře u Ethernetového spoje, kde IOS zná skutečnou rychlost, hůře jsou na tom spoje seriové nebo virtuální, například tunely.

EIGRP vždy počítá s nejnižší metrikou na cestě, která je přeposílána spolu jako vlastnost cesty.
EIGRP dokáže rozlišit bandwith od `1 kbps` do `10 Gbps`.

### Delay

Jedná se o hodnotu zaznamenanou v mikrosekundách, nastavitelnou pomocí příkazu `delay` v desítkách mikrosekund.
Bez manuálního nastavení IOS určí hodnotu na základě HW typu spoje (Ethernet, Serial, Sonet ...).

EIGRP vždy počítá se sumou všech spoždění, které se po cestě nacházejí.
EIGRP dokáže rozlišit delay od 1 do 167 772 14 desítek mikrosekund.

Hodnota 16 777 215 je považována, podobně jako metrika 16 u [[RIP|RIPu]], za nedosažitelnou a je používána pro šíření nedostupné cesty pomocí [[RIP#Split Horizon with Poisoned Reverse|Split Horizon with Poisoned Reverse]] a [[RIP#Route poisoning|Route poisoning]]. 

### Reliability

Jedná se o odhad spolehlivosti interfacu, který se vypočítává jako poměr: 
(**úspěšně přijaté pakety**:**všechny přijaté pakety** )$\times$ 255

Při výpočtu do celkové metriky EIGRP bere vždy pouze nejnižší hodnotu z celé cesty, kterou mu poslal soused a kterou porovná s vlastní hodnotou a dále ji přepošle.

Oproti ostatním metrikám, při změně této hodnoty nedojde k přepočítávání metriky, protože by to znamenalo, že téměř nikdy by nedošlo ke konvergenci, jelikož se tato hodnota může často měnit, například pouze v řádu promile.

### Load

Jedná se o odhad zatížení interfacu, který se vypočítává jako poměr:
(**Txload**:**celkový bandwith**) $\times$ 255

Pro vyvážení momentálních spiků v provozu IOS vypočítává exponenciálně vážený průměr, který by měl hodnotu rozmělnit.

EIGRP vždy počítá s maximální hodnotou po celé nabízené lince.

Podobně jako u Reliability ani tady se nepřepočítává metika vždy při změně hodnoty, započítává se do ní pouze snapshot z doby vytváření metriky.

### MTU

I přesto, že metrika je povžována za komponentu metriky a přeposílá se v paketech nejnižší MTU na cestě, MTU se nezapočítává do konečné (*Composed*) metriky, čistě proto, že jeho použití nikdy nebylo zakomponováno.

### Hop Count

Jedná se o počet hopů do nabízené sítě, podobně, jako u MTU, ani tato metrika není zakomponovaná do konečné metriky, jedná se pouze o ověření velikosti sítě, protože defaultní maximální Hop limit je 100 (lze změnit pomocí `metric maximum-hops`).

### Výpočet *Composed* metriky

Vzhledem k rozdílnému přístumu ke kompozičním metrikám (některé jsou brané jako nejnižší, některé nejvyšší), EIGRP má koncept *Composed metric (CM)*, kterou si každý router vypočítá na základě informací z paketů.
Tato matrika samotná naní nikdy přeposílána, pouze v jednom případě, při redistribuci z jiného EIGRP procesu a i poté pouze jako kontrola.

$$
CM=(K_1 \times BW_s + K_2 \times \frac{BW_s}{256-L_{Max}}+ K_3 \times D_s) \times (\frac{K_5}{K_4+R_{Min}})
$$


$$
BW_s=\frac{256 \times 10^7}{Bandwith}
$$


$$
D_s=256 \times Delay
$$

#### K-Values

Tyto hodnoty v rozpětí 0 - 255 určují váhu jednotlivých komponent celkové metriky:
- K1 - Bandwith
- K2 - Bandwith a Load
- K3 - Delay
- K4 - Reliability
- K5 - Reliability

Jsou konfigurovatelné a musí být shodné mezi všemi zařízeními, pokud router zjistí, že soused má rozdílné K-values nenaváže s ním sousedství a nebude tak ani přijímat jeho routovací informace.

Defaultně jsou hodnoty $K_1$ a $K_3$ nastavené na `1` a ostatní na `0`, z čehož vyplývá, že metrika pracuje pouze s bandwithem a delayem a lze dle toho zjednodušit vzoreček:

$$
CM= BW_s + D_s
$$

## Wide Metric
---

Zejména z důvodu nizkých rozlišovacích schopností u spoždění a bandwithu (`10Gbps` a 10mcs), bylo nutné předělat systém některých metrik pro lepší rozlišení kvalitních linek. 
Zároveň Badwith a Delay jsou v paketech přeposílány v $Scaled$ ($s$) formě, a tak požadují nejprve zpětné vypočtení původní hodnoty, což u Cisco CPU výpočtů může vést, a vedlo, k chybě zaokrouhlování a pozměňování hodnot. 

Z tohoto důvodu EIGRP vývojáři přišli s rozšířením tzv. *Wide Metric*, jejíchž podporu lze zjistit pomocí 
`R#show eigrp plugins`.

Tyto metriky jsou podporovány a automaticky aktivovány v *Named* režimu, jsou tedy většinou podporovány od
 `IOS 15.x`.
 Nelze jejich podporu nebo aktivati ovlivnit konfiguračně jinak, než právě *Named* režimem.
 Wide metriky jsou preferované, ale pokud router zjistí, že sousedí s routerem, který podporuje pouze klasickou metriku, bude s ním komunikovat na základě klasických metrik.
 
 Vzhledem k tomu, že po výpočtu *WD* může vyjít číslo dejší, než `32-bit` (maximální velikost metriky u RIB), samotná hodnota, která se zapisuje do RIB se dělí, defaultně 128, dělitele lze přenastavit pomocí `metric rib-scale`.
 Samotné EIGRP provádí logiku nad celým *WD*, ale do RIB se zapisuje zmenšené číslo.

### Throughput

Jedná se o analogickou hodnotu k [[#Bandwith]] u *Classic* metriky.
Pouze samotné číslo je jinak vypočítáno:

$$
T_{min}=\frac{65536 \times 10^7}{Bandwith(kbps)}
$$

Přičemž Bandwith je opět nejnižší přeposílaná hodnota z celé cesty.

Tato úprava umožňuje započítávat linky až do propustnosti `655.36 Tbps`.

### Latency

Jedná se o analogickou hodnotu k [[#Delay]] u *Classic* metriky.
Pouze samotné číslo je jinak vypočítáno:

$$
La_s=\sum{}{\frac{65536 \times Delay_{Interface}}{10^6}}
$$

Protože použitelná defaultní latence je pouze u interfaců s menší propustnostní, než `1Gbps`, tak výpočet $Delay_{Interface}$ se dělí do několika podmínek:

#### Propustnost do `1Gbps`

$$
Delay_{Interface}=DefaultDelay_{Interface}[picosec]
$$

Jedná se o defaultní, IOS, hodnotu interfacu převedenou na pikosekundy.

#### Propustnost nad `1Gbps`

$$
Delay_{Interface}=\frac{10^{13}}{DefaultBandwith{Interface}}
$$

#### Manuálně nastavené `bandwith`

Bez ohledu na nastavený bandwith nebo skutečnou propustnost je $Delay$ defaultní převeden na pikosekundy.

#### Manuálně nastavené `delay`

$$
Delay_{Interface}=10^7 \times ConfiguredDelay_{Interface}
$$

Jedná se o nastavenou hodnotu převedenou na pikosekundy.

### Extended Metrics

Jedná se o připravení pro budoucí rozšíření, v současné době lze zakomponovat *Jitter*, *Energy*, nebo *Quiescent Energy*.

Tato hodnota představuje novou *K-Value* ($K_6$).

Tato metrika nebývá používána ani podporována.

### Výpočet *Wide Composed* metriky

$$
WD=(K_1 \times T_{Min}+ K_2 \times \frac{T_{Min}}{256-Lo_{Max}}+K_3 \times La_s + K_6 \times ExtM)\times (\frac{K_5}{K_4+R_{Min}})
$$

