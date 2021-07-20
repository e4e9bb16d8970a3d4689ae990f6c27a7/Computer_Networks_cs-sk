# Terminologie
---

## Obecné termíny

- Autonomous System - AS
	- Skupina sítí, routerů pod stejnou technickou administrací
	- `1 - 65535`
	- Privátní rozsah `65512 - 65535`
- Administrative Distance - AD
	- Určuje důvěryhodnost protokolu, menší má přednost

|Protocol|AD|
|:--------:|:--:|
|Conected Interface|0|
|Static route|1|
|EIGRP Summary route|5|
|External BGP|20|
|[[EIGRP]]|90|
|IGRP|100|
|[[IS-IS]]|115|
|[[OSPF]]|110|
|[[RIP]]|120|
|EGP|140|
|On Demand Routing (ODR)|160|
|External EIGRP|170|
|Internal [[BGP]]|200|
|Default static via DHCP|254|
|Unknown|255|

- Konvergence
	- Čas potřebný na konverzi routovacího protokolu
	- Konvergence je dosaženo v okamžiku, kdy všechny routery mají všechny routovací protokoly

# Postup
---

- Přesnost
	- Jako první faktor pro zapsání cesty do RIB je délka prefixu
- AD
	- V případě, že prefix pro cestu je stejný, vybírá se protokol dle nejnižšího AD
- Metrika
	- Každý protokol má svou metriku
	- V případě, že i v rámci routovacího protokolu jsou stejné cesty, vybírá se dle nejnižší metriky
Pokud stále je více cest do jedné sítě, router použije typ load-balancingu

# Dělení protokolů
---

## Static

Jedná se o přesné, manuální, určení next-hopu pro určitou síť.
Manuálně musíme určit cesty do všech sítí, což i v malé síti může být problém, proto se upřednostňují dynamické protokoly a statické cesty jsou používány pouze v nějkterých případěch, napříkla jako náhradní cesta nebo defaultní cesta.

```
R(config)#ip route <NET> <MASK> <NEXT-HOP | INTERFACE> {AD}
```

### Directly Attached Static

Při použití *Point-to-point* (P2P) seriového spoje není potřeba pro [[CEF#Adjacency Table]] zjišťovat MAC adresu a jako výchozí cesta se může použít přímo interface.

Z hlediska forwardovací logiky se pak tato cesta bude chovat stejně, jako přímo připojená síť, díky čemuž se zjednodušuje forwardovací proces.

```
R1(config)i#p route 10.0.0.4 255.255.255.252 10.0.0.2
```

```R
R1(config)#do show ip cef
Prefix               Next Hop             Interface
0.0.0.0/0            no route
0.0.0.0/8            drop
0.0.0.0/32           receive              
10.0.0.0/30          attached             Serial1/0
10.0.0.0/32          receive              Serial1/0
10.0.0.1/32          receive              Serial1/0
10.0.0.2/32          10.0.0.2             Serial1/0
10.0.0.3/32          receive              Serial1/0
10.0.0.4/30          10.0.0.2             Serial1/0
127.0.0.0/8          drop
224.0.0.0/4          drop
224.0.0.0/24         receive              
240.0.0.0/4          drop
255.255.255.255/32   receive  

R1(config)#do show ip route

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.0.0/30 is directly connected, Serial1/0
L        10.0.0.1/32 is directly connected, Serial1/0
S        10.0.0.4/30 [1/0] via 10.0.0.2
```

```
R1(config)i#p route 10.0.0.4 255.255.255.252 s1/0
```

```R
R1(config)#do show ip cef                              
Prefix               Next Hop             Interface
0.0.0.0/0            no route
0.0.0.0/8            drop
0.0.0.0/32           receive              
10.0.0.0/30          attached             Serial1/0
10.0.0.0/32          receive              Serial1/0
10.0.0.1/32          receive              Serial1/0
10.0.0.3/32          receive              Serial1/0
10.0.0.4/30          attached             Serial1/0
127.0.0.0/8          drop
224.0.0.0/4          drop
224.0.0.0/24         receive              
240.0.0.0/4          drop
255.255.255.255/32   receive 

R1(config)#do show ip route

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.0.0/30 is directly connected, Serial1/0
L        10.0.0.1/32 is directly connected, Serial1/0
S        10.0.0.4/30 is directly connected, Serial1/0
```

### Recursive Static

Jedná se o cestu, při které je nutné vícenásobně prohledávat RIB pro určení všech informací, typicky po prvním prohledání najdeme IP next-hop, ale poté ještě musíme znovu prohledat RIB pro egress interface.

```
R1(config)#ip route 10.0.0.4 255.255.255.252 10.0.0.2
```

```R
R1(config)#do show ip route

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.0.0/30 is directly connected, Serial1/0
L        10.0.0.1/32 is directly connected, Serial1/0
S        10.0.0.4/30 [1/0] via 10.0.0.2
```

### Fully Specified Static

V případě specifikování výstupního interfacu se usnadní prohledávání RIB.

```
R1(config)#ip route 10.0.0.4 255.255.255.252 s1/0 10.0.0.2
```

```R
R1(config)#do show ip route

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.0.0/30 is directly connected, Serial1/0
L        10.0.0.1/32 is directly connected, Serial1/0
S        10.0.0.4/30 [1/0] via 10.0.0.2, Serial1/0
```

Ovšem ve většině případů se používá [[CEF]], takže tento proces je lehce poupraven a ve výsledku se nejedná o značnou optimalizaci.

### Floating Static

Jedná se o jedno z nejčastějších použití statických cest, manuálně nastavené cestě se zvýší AD nad dynamický protokol a díky tomu bude fungovat jako záloha v případě selhání.

### Static Null

V případě potřeby mazání provozu z určité sítě je možné vytvořit statickou cestu, která bude mířit na `null` interface, což je univerzální logický interface, který maže data bez nutnosti zatěžování CPU, je tedy rychlejší oproti například [[ACL]].

```
R1(config)#ip route 10.0.0.4 255.255.255.252 null0
```

## IGP - Interior Gateway Protocol

Jedná se o routing uvnitř AS (Autonomního systému), souboru sítí.
Routovací protokoli jsou tomu uspůsobeny

### Distance-Vector

- Routery udržují RIB s informací o vektoru, vzdálenosti, do dané sítě
- Periodicky posílají svou tabulku sousedům, ti si ji upraví a pošlou dále
- Jednoduché, nevytváří vztahy  ([[RIP]])
- Problémem jsou routovací smyčky, to řeší jednotlivé mechanizmy protokolů
	- I přes tyto mechanizmy nelze 100% zaručit bezsmyčkovou topologii, krom [[EIGRP]]
- [[RIP]]
- [[EIGRP]]
- [[IGRP]]

### Link-State

- Routery udržují komplexní mapu topologie vytvořenou pomocí LSA
	- Místo posílání připojených sítí, ze kterých příjemce vyhodnotí odesílatele jako next-hop, link-state protokoly posílají objekty s atributy
	- Nejčastější atribut je router, který má atributy v podobě interfaců a připojených sítí, díky tomu se může vypočítat mnohem komplexnější a kvalitnější topologie
	- Objekty ale mohou být i celé AS, mlutiaccess sítě nebo border routery
	- Router, respektive zástupce objektu, pak tyto informace přeposílá	
- Vyměňují si Link State Advertisements - LSA, ta jsou vyvolána nějakou událostí v síti, také Link State Packet - LSP nebo Link State PDU
- Sousedům zasílá Hello packet, ve kterém zasílá informace o sobě
- Rychle reaguje na změnu topologie, ale spotřebovává hodně propustnosti, zejména ze začátku, a zdrojů routeru
- Metrika je komplexně vypočítána pomocí Dijikstrova algoritmu, shortest path first - SFP
- Pro zlepšení vlastností se rozdělují sítě na menší oblasti, hraniční routery posílají sumarizaci sítě, využívají multicast a čísluje LSA
- [[OSPF]]
- [[IS-IS]]

## EGP - Exterior Gateway Protocol

Používá se mezi AS (Autonomní systém).

### Path-Vector

Jedná se o upravený typ [[Routing#Distance-Vector|Distance-Vector]], do jeho logiky přidává navíc další informace o cestě (*path elements*), například u [[BGP]] číslo AS.
Hlavní účel těchto path elements je předcházení smyčce, router nepřidá cestu, která již sdílí stejné údaje o cestě.

# Srovnání protokolů

||[[RIP]]|[[EIGRP]]|[[OSPF]]|[[IS-IS]]|[[BGP]]|
|:-:|:------:|:---------:|:---------:|:--------:|:--------:|
|**Typ**|Distance-Vector|Distance-Vector|Link-State|Link-State|Path-Vector|
|**Class**|Classful/Classless|Classless|Classless|Classless|Classless|Classless|
|**Metrika**|Počet hopů|Delay, BW, Reliability, Load, MTU|100Mb/BW|Ruční|Weight, Local_Pref, Originate, AS_Path, MED|
|**Algoritmus**|Bellman-Ford|DUAL|SPF|SPF|Best Path|
|**Hello pakety**|Nemá|5s|10s|10s|60s|
|**Komunikace**|[[UDP]]:520|IP:88|IP:98|[[IS-IS]]|[[TCP]]:179|
|**Tabulky**|RIB|RIB, Topology, Neighbour|RIB, Topology, Neighbour, LSA|LSA|Atributy, Topology|
|**IPv6**|RIPng|IPv6 EIGRP|OSPFv3|Podporuje|mBGP|
|**Sítě**|/|BMA, NBMA, Pt-Mpt|Pt-Pt, BM, NBMA, Pt-Mpt, Virtual link|/|/|


# Load-Balancing
---

## Equal-Cost Multipathing

*ECMP*

V případě, že protokol podporující ECMP ([[OSPF]], [[RIP]], [[IS-IS]], [[EIGRP]]) dostane více stejných cest do jedné sétě, nainstaluje je všechny do RIB a router mezi nimi load-sharuje na základě [[CEF#Load-Sharing]].

## Unequal-Cost Load Balancing

Tato funkce umožňuje nakonfigurovat protokol, zatím pouze [[Funkce EIGRP#Unequal-Cost Load Balancing|EIGRP]], aby přidal i, z hlediska metriky, nerovné cesty do RIB a load-balancoval s nimi na základě poměru jejich metriky.




