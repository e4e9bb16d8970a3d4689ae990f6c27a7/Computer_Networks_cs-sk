# Spanning Tree Protocol
---
STP komunikuje mezi switchi pro konverzi sítě na loop-free topologii.
Z toho důvodu některé porty prostě vypne, jinými slovy, převede do *Blocking* stavu. Zbylé porty ve *Forwarding* stavu tvoří loop-free topologii.
Počet maximálníx instancí je, dle platformy, omezen, například na platformách `2960`,`3560` a `3760` je maximální počet instancí 128.

Ke komunikaci mezi switchi využívá [[STP Terminologie#BPDUs|BPDUs]]

## Konvergence
---
Základní princip STP funguje na možnosti porovnávat BPDUs a rozhodnout, které z nich je *superior* a které je *inferior*.

K tomu porovnávají několik hodnot

[[STP Terminologie#Hodnoty|Hodnoty]]

a následující proces.

### Volba superiorního BPDU

1. [[STP Terminologie#Root Path Cost RPC|RPC]]
2. [[STP Terminologie#Sender Bridge ID SBID|SBID]]
3. [[STP Terminologie#Sender Port ID SPID|SPID]]
4. [[STP Terminologie#Reciever Port ID RPID|RPID]]

### Volba Root Switche

V STP může být pouze jeden switch Root Switch, k jeho zvolení se koná election, volba.

Každý switch začíná STP instanci rozesíláním svých vlastních BPDUs, které označují jeho za Root Switch, pokud switch uslyší sueprior BPDU, přestane se prohlašovat za Root Switch a přestane posílat BPDUs. 
Místo toho začne přeposílat superiorní BPDUs nového Root Switch kandidáta.

Stejná volba se koná i poté, co nějaký switch přestane dostávat superior Root Bridge BPDUs, onen switch se opět začne považovat za Root Bridge.

### Volba Root Portu

1. [[STP Terminologie#Root Path Cost RPC|RPC]]
2. [[STP Terminologie#Sender Bridge ID SBID|SBID]]
3. [[STP Terminologie#Sender Port ID SPID|SPID]]
4. [[STP Terminologie#Reciever Port ID RPID|RPID]]

Po zvolení Root Bridge je nutné určit i Root Port (RP), což je port s nejvýhodnější cestou k Root Switchi.

1. Root Bridge generuje Hello BPDU ([[STP Terminologie#Configuration BPDU 0x00|Configuration BPDU]]) každé 2 sekundy.
	- Má nastevené RBID a SBID na Bridge ID Root Bridge, RPC na 0 a SPID na ID výstupního portu.
2. Nonroot příjemce na příslušném portu přidá vlastní RPC hodnotu, což vede k *resultning* BPDU, které zpracuje.
	- Port, na který přišlo superior BPDU se stane Root portem.
3. Nonroot switch následně po updatování RPC, SBID, SPID a Message Age přepošle Hello BPDU ze všech svých Designated Portů.
	- Hello BPDUs získané na jiných portech jsou procesovány, ale nejsou přeposílány.

### Volba Designated portu

1. [[STP Terminologie#Root Path Cost RPC|RPC]]
2. [[STP Terminologie#Sender Bridge ID SBID|SBID]]
3. [[STP Terminologie#Sender Port ID SPID|SPID]]
4. [[STP Terminologie#Reciever Port ID RPID|RPID]]

Posedním portem, který se volí je [[STP Terminologie#Designated|Designated]]

Root Bridge má všechny své porty designated.

Po zvolení Root portu jsou všechny ostatní porty automaticky považovány za Designated, do doby, než na DP přijde superior BPDU, poté se, pro předejití smyčky, přepne port do [[STP Terminologie#Non-designated|Non-designated]].

Každý port si ukládá superior BPDU, i kdyby mělo jít o BPDU souseda.
Pokud má port uložené BPDU od souseda, musí ho dostat znovu do uplinuté Max Age doby, jinak mu vyprší trvanlivost.

## Změna topologie
---

Existuje případ v síti, kdy dochází k vytvoření černých děr, z pohledu L2 se jedná o switch, který má asociaci v CAM tabulce pro port, který již není funkční, defaultně se jedná o  Aging Timer, který je 300s dlouhý. 
To je poměrně dlouhá konvergence a po tuto dobu dochází k chybnému odesílání provozu na port, na kterém nic není, pouze "černá díra".

STP má mechanismus, díky kterému dočasné sníží Aging Timer z 300s na 15s.

Když switch zaregistruje změnu topologie, vypnutí portu nebo jeho přepnutí do Forwarding stavu, začne tuto změnu propagovat do celé sítě.

1. Nonroot switch detekuje změnu topologie
2. Začne rozesílat směrem k Root switchi (z RP) TCN BPDUs ([[STP Terminologie#Topology Change Notification BPDU 0x80|Topology Change Notification BPDU]]), na které každý příjemce po cestě odpoví pomocí Configuration BPDU s nastaveným TCA Flagem [[STP Terminologie#Flag|Flag]] .

Poté, co Root switch získá TCN, začne rozesílat Configuration BPDUs s nastaveným TC Flagem.

Tímto mechanismem dojde k několikanásobně rychlejší konvergenci CAM tabulek.

Informace o poslední konvergenci lze zjistit pomocí

```
SW1(config)#spanning-tree vlan <VLAN> detail
```

### Scénáře změny

#### Direct Link Failure

Jedná se o stav, při kterém se linka přepne do stavu `down`, a tak se vymaže uložená informace o superiorním BPDU, a tak se nemusí čekat na [[STP Terminologie#Max Age|Max Age]].

##### Vypadnutí linky mezi DP a B porty

V takovém případě nejde o změnu topologie, protože na tomto segmentu stejně neprobíhá normální komunikace.
Oba switche rozešlou TCN, což povede ke znovu vytvoření CAM tabulky.

##### Vypadnutí linky mezi Root switchem a switchem s Blocking portem

1. Root switch a Nonroot switch detekují chybu na lince.
2. Root switch vygeneruje Configuration BPDUs s nastaveným TC Flagem.
	- Protože Nonroot switch přišel o RP, nedělá nic
3. Configuration BPDUs s nastaveným TC Flagem se rozšíří po síti, což vede k znovu vytvoření CAM tabulky
4. Nonroot switch, kterému vypadla linka musí čekat, jestli se linka znovu nenahodí, nebo dokud se BLK pork nepřepne do Forwarding stavu, defaultně 30s
	- Pokud se linka přepne do shutdown stavu, tak dojde k vymazání uloženého BPDU a nemusí se čekat Max Age

##### Vypadnutí linky mezi Root switchem a switchem naproti Blocking portu

1. Root switch a Nonroot switch detekují chybu na lince.
2. Nonroot switch začne generovat Configuration BPDUs, ve kterých se považuje za Root switch, protože mu vypadlo spojení s Root switchem.
	- Tyto BPDUs jsou ale inferioriní pro příjemce
3. Root switch začne rozesílat Configuration BPDUs s nastaveným TC Flagem.
4. Max Age timer na BLK portu na Nonroot switchi expiruje a port se začne přepínat do forwarding stavu (20s + 15s + 15s)
5. Původní Nonroot switch, kterému vypadla linka nyní získává superiorní BPDU a přepne port z Designated stavu na RP

#### Indirect Failures

Jedná se o chybu, při které linka jako taková zůstává `up` a tudíš nedochází k vymazání záznamu o superiorním BPDU, ale přenos dat je nějak zkoruptován, například na něj jsou aplikovány filtry.

Tyto chyby se řeší stejně jako Direct Link Failure, ale protože nedochází k vypnutí linky, tak se do konvergence musí přičíst i Max Age Timer.


## Varianty
---

[[Rapid STP]]
[[MSTP]]
[[Per-VLAN STPs]]



