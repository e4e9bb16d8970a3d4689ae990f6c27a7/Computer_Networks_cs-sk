[[STP]]
# Iterace
---

## Druhy

- STP - 802.1D
- Per-VLAN Spanning Tree - PVST
- Per-VLAN Spanning Tree Plus - PVST+
- Rapid Spanning Tree Protocol - RSTP - 802.1W
- Multiple Spanning Tree Protocol - MST - 802.1s (802.1Q-2005)

## Názvy

### Legacy STP

Původní 802.1D

### Rapid STP

Jedná se o přídavek pro původní STP, 802.1w

### Multiple STP

Jedná se o přídavek pro VLANy, 802.1s

### Změna

802.1D - 2004 Už nezahrnuje původní "Legacy STP", místo toho je postaven na RSTP (802.1w) a MSTP (802.1s) je zahrnuto do 802.1Q-2005

# Porty
---

## Role
---
### Root
- Nejlepší cesta k Root Bridge
- Každý switch musí mít právě jeden
- [[#Forwarding]]

### Designated
- Normální port
- Posílá Hello BPDUs
- [[#Forwarding]], pro RSTP dokud nepříjde agreement, tak v [[#Discarding]]

### Non-designated
- Port zablokovaný z důvodu předejití loop v topologii
- [[#Blocking]]

### Alternate
- Jedná se o náhradu za [[#Non-designated]] port
- Tento port funguje jako alternativa pro RP a je schopen okamžitě po vypadnutí RP linky se přepnout a fungovat jako RP
- [[#Discarding]]

### Backup
- Jedná se o port, který poskytuje redundanci linek
- Vytvoří se v případě, že na tento port switch dostane superiorní BPDU s vlastním SBID
- Jedná se o případy, kdy je switch připojen k hubu a vrací se mu komunikace
- [[#Discarding]]

## Typy
---

Na Cisco Catalyst switchích je defaultní non-edge port.

### Point-to-point (P2P) 
- Port je připojen k dalšímu zařízení, PC nebo RSTP switch
### Edge
- Na tomto portu je zapnutý Portfast
- Při jejich změně negenerují TCN
- Udržuje tento status, dokud na něj nepříjde BPDU, poté funguje dle STP, pouze v *runtime*, konfigurace se nezmění
- Okamžitě se po zapnutí přepíná do [[#Forwarding]]
### \*TYPE_Inc 
- Jedná se o chybovou hlášku
- Indikuje mismatch mezi switchi, typicky Access port s Trunkem
### Shared
- Jedná se o připojení k Hubu, musí používat half-duplex
- V případě Rapid STP nebude používat Synchronization Process, ale Timery

## Stavy
---
### Disabled
- Jedná se o administrativné vypnutý port.

### Blocking
- Port je administrativně zapnutý, ale STP ho blokuje, aby zamezil smyčce v topologii
- Port může zpracovávat BPDUs, ale neposílá je 
- Neupravuje CAM

### Listening
- Port se do tohoto stavu přepne z Blocking stavu
- Přijímá i přeposílá BPDUs
- Nefunguje pro normální síťový provoz
- Tranzitní doba je závyslá na STP Forwarding Timeru, poté se přepne do stavu Learning

### Learning
- Port už může upravovat CAM tabulku na základě přijatého provozu
- Nadále přeposílá pouze BPDUs, ne normální síťový provoz
- Tranzitní doba je závyslá na STP Forwarding Timeru, poté se přepne do stavu Forwarding

### Forwarding
- Jedná se o port v normálním provozu
- Tento stav je považován za konečný

### Broken
- Z nějakého důvodu, nejčastěji nějaký STP guard, se port přepne do Broken stavu
- Port maže veškerou příchozí komunikaci
- Pro změnu stavu je nutné port vypnout a znovu zapnout

### Discarding
- Jedná se pro port, který je náhradou pro [[#Disabled]], [[#Blocking]] a [[#Listening]] stavy v RSTP
- Funguje jako [[#Blocking]] port

## STP
---

Následující parametry jsou obsaženy v STP protokolech založených na 802.1D.

### Stavy

- [[#Disabled]]
- [[#Blocking]]
- [[#Listening]]
- [[#Learning]]
- [[#Forwarding]]
- [[#Broken]]

### Porty

 - [[#Designated]]
 - [[#Root]]
 - [[#Non-designated]]

## Rapid STP
---

Následující parametry jsou obsaženy s STP protokolech založených na 802.1w.
 
 ### Stavy
 
 - [[#Discarding]]
 - [[#Learning]]
 - [[#Forwarding]]
 - [[#Broken]]
 
 ### Porty
 
 - [[#Designated]]
 - [[#Root]]
 - [[#Non-designated]]
	 - [[#Alternate]]
	 - [[#Backup]]
 
# BPDUs
---

Všechny BPDUs jsou posílány na MAC adresu `01:80:c2:00:00:00`.

## Configuration BPDU (`0x00`)

|BPDU Pole|Délka|
|:--------:|:------:|
|Protocol ID|2|
|Protocol Version|1|
|BPDU Type|1|
|[[#Flag]]|1|
|[[#Root Bridge ID RBID]]|8|
|[[#Root Path Cost RPC]]|4|
|[[#Sender Bridge ID SBID]]|8|
|[[#Sender Port ID SPID]]|2|
|[[#Message Age]]|2|
|[[#Max Age]]|2|
|[[#Hello Time]]|2|
|[[#Forward Delay]]|2|
| [[#Version 1 length]]| 2 |

Pro STP je Protocol ID nastaveno na `0x0000` a Protocol Version na `0x00`.

## Topology Change Notification BPDU (`0x80`)

|BPDU Pole|Délka|
|:-:|:-:|
|Protocol ID|2|
|Protocol Version|1|
|BPDU Type|1|

Pro STP je Protocol ID nastaveno na `0x0000` a Protocol Version na `0x80`.

## Hodnoty
---

### Bridge ID

Původní Bridge ID se skládalo z 
- 2B - **Priority Field**, konfigurovatelná hodnota, defaultně 32 768 ^Priority
- 6B - **System ID**, MAC adresa, stálá hodnota určená jako tiebreaker

Následně se upravilo pro použití s VLAN
- 4b - **System Priority**, Násobek 4096, defaultně 32 768 ^PriorityVlan
- 12b - **System ID Extension** - Typicky VLAN ID
- 6B - **System ID**, MAC adresa, stálá hodnota určená jako tiebreaker

Používá se Base MAC adresa, kterou lze zjistit pomocí 
```
SW1#show version
```

Při volení na základě Bridge ID ([[STP#Volba Root Switche|Volba Root Switche]], [[STP#Volba Root Portu|Volba Root Portu]], [[STP#Volba superiorního BPDU|Volba superiorního BPDU]]), se nejprve porovnává **Priorita** a jestliže je shodná, vyhrává nižší MAC adresa.

Standart 802.1D vyžaduje originální Bridge ID pro každý switch v topologii, vzhledem k tomu, že kvůly VLANám se jeden switch chová jako více logických, ovšem se stejnou MAC adresou, nevyhovovalo by to standartu. 
Proto se přešlo na implementaci VLAN do Bridge ID, ale někteří výrobci můžou mít implementaci poněkud odlišnou, napčíklad Cisco zabudovává VLAN ID přímo do MAC adresy, při použití původního STP.
Existenci implementace System ID Extension lze ověřit přistupností příkazu 
```
SW1(config)#spanning-tree extended system-id
``` 
Starší switche s většim pamětním prostorem pro MAC adresy umožnují tento příkaz vypnout, ale většina novějších už ne.

#### MST

U MSTP se Bridge ID skládá z:

- 4b - **System Priority**, Násobek 4096, defaultně 32 768 
- 12b - **System ID Extension** - Číslo Instance
- 6B - **System ID**, MAC adresa, stálá hodnota určená jako tiebreaker

### Root Bridge ID (RBID) 

Jedná se o [[#Bridge ID]] zvoleného Root Bridge.

### Root Path Cost (RPC)

Jedná se o součet ceny hopů na cestě k Root Bridgi, v tomto ohledu funguje jako [[Distance-Vector Routing Protocol]].

#### Ceny

|Rychlost portu|802.1D-1998 (Short Mode)|802.1D-2004 (Long Mode)|
|:-:|:-:|:-:|
|10 Mbps|100|2 000 000|
|100 Mbps|19|2 000 00|
|1 Gbps|4| 2 000 0|
|10 Gbps|2|2 000|
|100 Gbps|/|200|
|Definice|$\frac{1Gbps}{Bandwith}$ Následně změněno|$\frac{20Tbps}{Bandwith}$|

I na posledních Catalyst switchích defautlní hodnoty odpovídají 802.D-1998 verzi pro PVST i Rapid PVST, 802.1D-2004 hodnoty jsou používány pro MSTP.
802.1D-2004 hodnoty lmohou být nakonfigurované pomocí příkazu
```
SW1(config)#spanning-tree pathcost method <long|short>
``` 
`long` pro 802.1D-2004 a `short` pro 802.1D-1998, ta je defaultně

### Sender Bridge ID (SBID)

Jedná se o [[#Bridge ID]] switche, od kterého dostal BPDU.

### Sender Port ID (SPID)

Jedná se o ID portu na druhém zařízení.

Skládá se z
- Port Priority - 4b - Defaultně 128
- Port ID - 12b - Jedná se o identifikační číslo portu

Toto číslo je zapisováno v hexadecimální soustavě, takže po převedení se jedná například o `128.5`.

### Reciever Port ID (RPID)

Jedná se o [[#Sender Port ID SPID]] s tím rozdílem, že se jedná o lokální port.

### Message Age

Jedná se o hodnotu, která má zabránit nekonečnému přeposílání BPDUs mezi switchi, kteři nerozumí STP.

BPDU z Root switche odchází s touto hodnotou nastavenou na 0.
Při přijetí BPDU si switch přičte k této hodnotě 1 a pokud zjistí, že je rovna nebo větší Max Age timeru, tak BPDU zahodí.

### Max Age

Čas po který si switch uchová informaci o cizím superior BPDU.
Defaultně se jedná o 20s pro STP a o 6s pro protokoly založené na Rapid STP, ale může být nakonfigurován pomocí
```
SW1(config)#spanning-tree vlan <VLAN> max-age <MaxAge>
```

Pokud Nonroot switch nedostane superior BPDU po tuto dobu, smaže uložené a začne rozesílat vlastní superior BPDUs.
### Hello Time

Jedná se o hodnotu, která určuje, jak často se budou posílat Hello (Configuration) BPDUs.
Defaultní hodnota jsou 2s, ale lze ji přizpůsobit, od 1s do 10s.

```
SW1(config)#spanning-tree vlan <VLAN> hello-time <HelloTime>
```

### Forward Delay

Určuje délku doby změny stavu portů z [[#Listening]] na [[#Learning]].
Defaultní hodnota je 15s, ale lze ji přizpůsobit, od 15s do 30s.

```
SW1(config)#spanning-tree vlan <VLAN> forward-time <ForwardTime>
```

### Version 1 length

Indikuje, že se nejedná o ver. 1 STP protokol.

### Flag

#### STP

- Topology Change (**TC**)
	- Jedná se o bit, který určuje, zda se udála změna topologie
	- Krátkodobě změní Aging Timer CAM tabulky z 300s na Forward Delay Timer
	- Umožňuje rychlejší konvergenci
- Topology Change Ack (**TCA**)
	- Jedná se o zprávu, kterou posílá příjemce TCN BPDU
	- Dává najevo, že dostal zprávu

#### RSTP

- Topology Change (**TC**)
	- Jedná se o bit, který určuje, zda se udála změna topologie
	- Krátkodobě změní Aging Timer CAM tabulky z 300s na Forward Delay Timer
	- Umožňuje rychlejší konvergenci
- Proposal
	- Jedná se o bit, který označuje, že se jedná o žádost o potrvzení superiorního BPDU pro segment
- Port Role
	- 00 - Unknown Port
	- 01 - Alternate/Backup Port
	- 10 - Root Port
	- 11 - Designated Port
- Learning
	- Označuje, zda je port v Learning stavu
- Forwarding
	- Označuje, zda je port ve Forwarding stavu
- Agreement
	- Označuje přijmutí superior BPDU od inferior portu na segmentu
	- Po odeslání tohoto Flagu se dokončuje konvergence
- Topology Change Ack (**TCA**)
	- Jedná se o zprávu, kterou posílá příjemce TCN BPDU
	- Dává najevo, že dostal zprávu
	- U RSTP se nepoužívá, protože se nepoužívá [[#Topology Change Notification BPDU 0x80]]