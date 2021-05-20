# First-Hop Redundancy Protocols (FHRPs)

V dnešních sítích je krom propustnosti další klíčová vlastnosti stabilita a rezilience, ideální síť by měla být v provozu 100% času, na L2 to řešíme přidáváním L2 Switchů a spojů, na L3 to opět řešíme přidáváním L3 zařízení a spojů.
Oproti L2 je ale na L3 problém s IP adresami, není možné mít 2 IP adresy jako Default Gateway a není možné mít 2 IP adresy na jednom zařízení.

FHRP protokoly tuto problematiku řeší přidělením jedné virtuální IP adresy a mnohdy i MAC adresy několika zařízením najednou a o správné přeposílání se stará L2.
V tomto případě pak nevadí, že nám jeden ze skupiny L3 zařízení vypadne, jestliže máme další, které tyto virtuální adresy sdílí. 

### Object Tracking

Rozhodování o aktivním prvku ze skupiny lze dělat na základě sledování stavů uvnitř routeru, typicky nemá smysl využívat jeden z routerů, jestliže ztratil konektivitu do *WAN*...

```
R1(config)#track <NUMBER> ip route <ROUTE> {reachability | metric}    \\ Nastavení trackingu na základě cesty
```

```
R1(config)#track <NUMBER> interface <IF> {line-protocol | ip}     \\ nastavení trackingu na základě stavu interfacu
```

```
R1#show track     \\ Zobrazení nastavení
```

## Hot Standby Router Protocol (HSRP)
---

Jedná se o cisco proprietární protokol, který vytváří virtuální IP a MAC adresu mezi členy HSRP skupiny. Jednotlivý členové musí mít L3 konektivitu.
Jedno zařízení jedná jako **active**, to se stará o zpracovávání provozu, a ostatní jako **standby**, které vyčkávají na selhání aktivního prvku.
**active** L3 zařízení rozesílá multicastové UDP-based Hello packety na port `1985` každé 3s (Hello timer) a při nedostání po 10s (Hold Timer) se přepne do **active** režimu L3 zařízení s nejvyšší prioritou, v případě, že je priorita defaultní (100), pak se rozhoduje na základě nejnižší IP adresy v daném subnetu.
V případě, že není nastavené *preemption*, pak po znovu naběhnutí primárního L3 zařízení se stav **active** nevrátí, ale zůstává u stávajícího zařízení.

Tento protokol zatím nepodporuje přímý load-balancing, ale lze ho nastavit podobně jako u [[STP]], tím, že nastavíme rozdílné priority pro různé zařízení v různích VLANách, takovému nastavení se říká *Multiple HSRP (MHSRP)*.
Umožňuje maximálne 16 instancí.

### Timers

|Timer|Použití|
|:-:|:-:|
|Active|Používá se pro určení funkčnosti aktivního zařízení, shodné s Hold Timer|
|Standby|Používá se pro určení funkčnosti standby zařízení, shodné s Hold Timer|
|Hello|Interval, ve kterém každé zařízení posílá Hello pakety|
|Hold|Jedná se o timer pro vypršení, používá se pro vypršení stavu Active a Standby|

### HSRPv1 vs HSRPv2

|Oblast|HSRPv1||HSRPv2|
|:-:|:-:|:-:|
|Timers|Nepodporuje milisekundové intervaly|Podporuje milisekundové intervaly|
|Group Range|0-255|0-4095|
|Multicast adress|224.0.0.2|224.0.0.102|
|MAC|0000.0C07.ACxy|0000.0C9F.F000 - 0000.0C9F.FFFF|
|IPv6|/|FF02::66;0005.73A0.0000 - 0005.73A0.0FFF; 2029(UDP)|

### Stavy

|Stav|Popis|
|::|:-:|
|Initial|HSRP zatím neběží|
|Learn|Očekává Hello z active zařízení, nezná virtuální adresy, nedostal autentizační paket|
|Listen|Zná virtuální adresy, ale není active ani standby, očekává hello pakety od těchto zařízení|
|Speak|Posílá hello pakety a účastní se volby active/standby zařízení|
|Standby|Funguje jako kandidát na active zařízení, konečný stav|
|Active|Jedná se o zařízení, které obstarává virtuální adresy, a tak i provoz|

### Konfigurace
---

#### Základní

```
R1(config)#interface <IF>     \\ Přepnutí se na interface, který má participovat v HSRP
R1(config-if)#ip address <IP> <MASK>     \\ Zařízení musí mít L3 konektivitu
R1(config-if)#standby version <VER>     \\ Nastavení verze, doporučeno nastavit 2, což je i defaultní
R1(config-if)#standby <GROUP> ip <VIP>     \\ Nastavení virtuální IP adresy, naše Default Gateway
R1(config-if)#standby preempt     \\ Nastavení preemptivního režimu, vždy předá řízení zařízení s nejvyšší prioritou
R1#show standby [IF] [brief]     \\ Zkontrolování nastavení
```

#### Vedlejší

##### Timers

```
R1(config-if)#standby <GROUP> timers {msec} <HELLO> <HOLD>
```

##### Autentizace

```
R1(config-if)#standby <GROUP> authentication md5 key-string <PASSWD>
```

##### [[#Object Tracking]]

```
R1(config-if)#standby <GROUP> track <TR_NUMBER> {decrement <0 - 255> | shutdown}     \\ Spojení s Object-trackingem
```

##### Další

```
R1(config-if)#standby <GROUP> priority <PR>     \\ Nastavení priority
```

```
R1(config-if)#standby <GROUP> mac-address <MAC>     \\ Manuální nastavení VMAC
```

## Virtual Router Redundancy Protocol (VRRP)
---

Jedná se o standartizovanou verzi HSRP a tudíš ji podporují i non-Cisco výrobci, základní funkcionalita je shodná, ale pár drobností je změněných.

|Věc|VRRP|HSRP|
|:-:|:-:|:-:|
|Počet instancí|16|255|
|Active/Standby|1 Active, 1 Standby, ostatní kandidáti|1 Master, ostatní backup|
|VIP|Musí se lišit od reálně|Může být shodná s reálnou|
|Multicast address|224.0.0.2(.102)|224.0.0.18|
|Časovače|1s,10s|1s,3s|
|Autentizace|Ano|Ne v RFC, Cisco ji podporuje|
|Preemptivita|Defaultně vypnuto|Defaultně zapnuto|

### VRRPv2 vs VRRPv3

Jsou vzájemně nekompatibilní.
VRRPv3 navíc podporuje:
- IPv6
- Časovače v msec

VRRPv3 má navíc změněnou *preemption* logiku, už o ní nerozhoduje IP adresa, ale **pouze** priorita.

Pro MAC IPv4 používá 0000.5E00.01xy a IPv6 0000.5E00.02xy, kde xy je číslo instance, stejně, jako u HSRP.

### Konfigurace
---

#### Základní

```
R1(config)#interface <IF>     \\ Přepnutí se na interface, který má participovat v VRRP
R1(config-if)#ip address <IP> <MASK>     \\ Zařízení musí mít L3 konektivitu
R1(config-if)#vrrp <GROUP> ip <VIP>     \\ Nastavení virtuální IP adresy, naše Default Gateway
R1#show vrrp [IF] [brief]     \\ Zkontrolování nastavení
```

#### Další

```
R1(config-if)#vrrp <GROUP> description <DCT>     \\ Nastavení popisu
```
```
R1(config-if)#vrrp <GROUP> authentication md5 key-string <PSWD>     \\ Nastavení autentizace    
```

```
R1(config-if)#vrrp <GROUP> timers advertise {<NUMBER> | msec <NUMBER>}     \\ Nastavení Hello timeru
R1(config-if)#vrrp <GROUP> timers learn {<NUMBER> | msec <NUMBER>}     \\ Nastavení Hold timeru
```

```
R1(config-if)#vrrp <GROUP> priority <1 - 254>     \\ Nastavení priority
```

##### [[#Object Tracking]]

```
R1(config-if)#vrrp <GROUP> track <TR_NUMBER> decrement <0 - 255>     \\ Spojení s Object-trackingem
```

#### IPv6

V případě [[IPv6]] je nutné nejprve přepnout verzi VRRP na v3:

```
R1(config)#fhrp version v3
```

A následně nastavit adresní rodinu:

```
R1(config-if)#vrrp <GROUP> address-family {ipv4 | ipv6}
```


## Gateway Load Balancing Protocol (GLBP)
---
Jedná se o Cisco proprietární protokol, novější než HSRP, který mimo běžných funkcí nabízí i load-balancing.
Posílá Hello pakety na `224.0.0.102:3222(UDP)`

Ten funguje tak, že protokol určuje 2 typy funkcí ve skupině:
- Active Virtual Gateway (AVG)
  - Jeden ve skupině
  - Přiřazuje VMAC
  - Odpovídá na prvotní ARP Request pomocí nějaké AVF virtuální MAC adresy, a tak řídí load-balancing
  - Může být i AVF
- Active Virtual Forwarder (AVF)
  - Stará se o routování komunikace od uživatelů
  - Má shodnou virtuální IP adresu se všemi ve skupině, ale má vlastní MAC adresu
  - `0007.B400.xxyy`, kde xx je číslo skupiny a yy číslo zařízení

Cisco umožňuje konfigurovat až 1024 skupin na interface a do každé 4 zařízení.

### AVG 

|Stav|Popis|
|:-:|:-:|
|Disabled|Není nakonfigurována VIP|
|Initial|Je nakonfigurována VIP, ale není zapnutý interface|
|Listen|Funguje jako záloha pro Standby, po změně stavu Standby zařízení se ukoná volba o nové|
|Speak|Přechodový stav mezi Listen a Active/Standby stavy|
|Standby|Pouze jeden interface, další v pořadí na Active, po vypršení Hold timeru se přepne na Active|
|Active|Zařízení má aktivní roly AVG|

Preemption je defaultně vypnuté, pouze pro přepnutí do Active role, u přepnutí mezi rolemi Standby a Listen se systém chová, jako by byla zapnutá a záleží i na IP adresách, konfigurace se pouze stahuje na Active stav.
Při nastavení Preemption lze nastavit delay, který počká po určitý čas, než předá roli, zabraňuje to rychlému přeskakování rolí.
Stejně jako u VRRP se do Preemption počítá pouze priorita.

```
R1(config-if)# glbp <GROUP> priority <PR>     \\ Nastavení priority pro volbu AVG
```
```
R1(config-if)# glbp <GROUP> preempt {delay minimum <DELAY>}     \\ Nstavení délky čekání před předáním role
```

### AVF 

|Stav|Popis|
|:-:|:-:|
|Disabled|Není nakonfigurována VMAC|
|Initial|Je nakonfigurována VMAC, ale není zapnutý interface|
|Listen|Funguje jako záloha pro Active, po vypršení Hold timeru se přepne na Active|
|Active|Zařízení má aktivní roly AVF|

Preemption je defaultně zapnuté, ale funguje trochu jinak, místo priority funguje na základě vah (weighting), defaultní je stále 100.

- *lower*
  - Defaultně 1
  - Lze nastavit mezi 1 a *weighting - 1*
  - Určuje hranici, pod kterou přestane interface být Active AVF 
- *upper*
  - Defaultně *weighting* (100)
  - Určuje hranici, nad kterou interface přestane být Active
- *weighting*
  - Funguje jako priorita pro preemptivitu
  - Defaultně 100

Tento způsob je velmi šikovný, protože oproti HSRP a VRRP, kdy můžete pomocí [[#Object Tracking]] pouze vypnout interface, v tomto případě ho lze i zapnout.
Může mu být i nastaven delay, stejně jako u AVG, defaultní je 30s.

```
R1(config-if)#glbp <GROUP> weighting <MAX> lower <MIN> upper <UPPER>
```

### Load-Balancing

Algoritmus spravuje AVG, což znemená, že při změně zařízení AVG se může algoritmus změnit.

```
R1(config-if)# glbp <GROUP> load-balancing {host-dependent | round-robin | weighted}
```

#### Round-robin

- Defaultní
- Pro každý ARP Request vždy vybere následující MAC adresu a tak je střídá

#### Host-dependent

- Přiřazuje MAC k hostům a při ARP Requestu vždy navrací určenému hostovi stejnou MAC adresu
- Potřebuje `client-chache`

##### Client Cache

- Jedná se o tabulky s přiřazováním MAC a IP adres koncových zařízení, které použily ARP Request pro VIP
- Obstarává jí AVG
- Defaultně vypnuta, používá se pouze pro Host-dependent

```
R1(config-if)#glbp <GROUP> client-chache maximum <8 - 2000> {timeout <1 - 1440>}
```

- Hodnota 8 - 2000 označuje počet záznamů
- timeout označuje čas, za který se záznam vymaže, defaultně 1440 minut, 1 den

#### Weighted

- Nastavuje poměr mezi AVF
- AVF s vahou 200 bude využíváno 2x často, než AVF s vahou 100

### Redirect Timers

Po selhání AVF v případě, že není žádné další GLBP zařízení, které by převzalo jeho roli, jeden zesoučasných AVF převezme jeho VMAC a bude forwardovat její provoz.
Časem tuto nadbytečnou VMAC AVF odstaraní, ale vzhledem k to,už je ji stále mohou některé koncové zařízení používat jako GW, není možné ji smazat okamžite, od toho se nastavují 2 časovače:
- Redirect
  - Udává čas, po který AVG bude i nadále odpovídat na ARP Requesty s již nadbytečnou VMAC
- Failed Forwarder
  - Udává čas, na který převezme provoz VMAC jiný AVF

```
R1(config-if)# glbp <GROUP> timers redirect <REDIRECT> <FAILED_FORWARDER>     \\ V sekundách
```

### Autentizace

Stejně jako HSRP podporuje autentizaci jak textovou, tak MD5.

```
R1(config-if)#glbp <GROUP> authentication md5 key-string <PASSWD>
```

### Pakety

#### Hello

Je posíláno všemy interfacy pro sdílení informací a stavu spojeném s AVG.

#### Request/Response

Posílají je pouze aktivní AVF a fungují jako *keep-alive*.

### [[#Object Tracking]]

Stejně jako u HSRP je i tady funkce sledování stavu objektů a reagování na jejich změnu.
Tady funguje malá změna, která souvisí s [[#AVF]] váhami.

```
R1(config-if)#glbp <GROUP> weighting track <TRACK> {decrement <VALUE>}
```
