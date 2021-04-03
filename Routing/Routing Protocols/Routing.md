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
## IGP - Interior Gateway Protocol

Jedná se o routing uvnitř AS (Autonomního systému), souboru sítí.
Routovací protokoli jsou tomu uspůsobeny

### Distance-Vector

- Routery udržují RIB s informací o vektoru, vzdálenosti, do dané sítě
- Periodicky posílají svou tabulku sousedům, ti si ji upraví a pošlou dále
- Jednoduché, nevytváří vztahy  ([[RIP]])
- Problémem jsou routovací smyčky, to řeší
	- TTL
	- Split Horizon - Neposílání tabulky na rozhraní, ze které přišlo
	- Hold-down timer - Čekací interval, než je síť stabilní, prodlužuje konvergenci

- [[RIP]]
- [[EIGRP]]
- [[IGRP]]

### Link-State

- Routery udržují komplexní mapu topologie vytvořenou pomocí LSA
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

Jedná se o upravený typ [[Routing#Distance-Vector|Distance-Vector]]

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




