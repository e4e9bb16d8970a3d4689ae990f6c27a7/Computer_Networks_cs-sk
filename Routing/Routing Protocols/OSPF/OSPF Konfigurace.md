# OSPF Konfigurace
---

## Základní konfigurace
---

```
R(config)#router ospf <PID>     \\ Vytvoření procesu, ID se nemusí shodovat, je interně signifikantní
R(config-router)#router-id <IP>     \\ Nastavení RID, volitelné, nutno nastavit před network příkazem!
R(config-router)#network <IP> <WC_MASK> area <ARN>     \\ Zapnutí interfaců v určité arei
```

> Process ID je sice lokálně signifikatní, ale hraje roli ve výběru cesty v případě, že jsou 2 shodné:
	>> Pokud máme 2 procesy na jednom routeru a z obou nám přichází stejná cesta, typicky defaultní, pak se vybere ta z procesu s nižším ID.

```
R#show ip protocols     \\ Zobrazení stavu routovacích protokolů
R#show ip ospf neighbor     \\ Zobrazení sousedů a jejich stavů
R#clear ip ospf proccess     \\ Restartování OSPF
```

```
R(config-router)#passive-interface <IF|default>     \\ Nastavení passive-interfacu
```

Vzhledem k tomu, že `network` příkazem se specifikují interfacy, na kterých se má OSPF zapnout, je možné OSPF zapnout i přímo na interfacu. 

```
R(config-if)#ip ospf <PID> area <ARN>     \\ Zapnutí interfacu v určité arei
```

```
R(config-router)#shutdown     \\ Vypnutí OSPF procesu
R(config-if)#ip ospf shutdown     \\ Vypnutí OSPF procesu
```

### Sousedství & DR/BDR

```
R1(config-if)#ip ospf network <point-to-multipoint non-broadcast|non-broadcast>     \\ Přepnutí typu sítě
R1(config-router)#neighbor <IP>     \\ Nastavení souseda
```

Manuální nastavení souseda umožňuje přepnout defaultní multicastovou komunikaci na unicastovou. Toto nastavení ale není možné na defaultním typu ethernetové sítě (broadcast), manuálně nastavit sousedy lze pouze na *NBMA* a *P-t-MP NB*, přičemž *NBMA* používá DR/BDR a *P-t-MP NB* ne.


```
R(config-if)#ip ospf priority <PR>     \\ Nastavení priority pro DR/BDR volbu
```

#### R1

```
R1(config-router)#neighbor <R2_IP> priority 0     \\ Nastavené souseda s 0 prioritou
R1(config-router)#neighbor <R3_IP> priority 1     \\ Nastavené souseda s prioritou
```

#### R2

```
R2(config-router)#neighbor <R1_IP> priority 0     \\ Nastavené souseda s 0 prioritou
R2(config-router)#neighbor <R3_IP> priority 0     \\ Nastavené souseda s 0 prioritou
```

#### R3

```
R2(config-router)#neighbor <R1_IP> priority 1     \\ Nastavené souseda s prioritou
R2(config-router)#neighbor <R2_IP> priority 0     \\ Nastavené souseda s 0 prioritou
```

Toto nastavení, dle [[OSPF Adjacency#Neighbor|Neighbor]], zajistí, že spolu navážou DR/BDR pouze `R1` a `R3`.

### Metrika

```
R(config-if)#bandwith <kbps>     \\ Přenastavení bandwith hodnoty pro výpočet metriky interfacu
```

```
R(config-if)#ip ospf cost <C>     \\ Manuální přiřazení hodnoty, tento příkaz má vždy přednost
```

Minimální hodnota metriky je 1, vzhledem k defaultnímu výpočtu [[OSPF Areas & LSAs#OSPF Path Selection|metriky]] jsou všechny linky nad `100mbps` shodné, proto je doporučeno referenční hodnotu zvýšit.

```
R(config)#router ospf <PID>     \\ Přepnutí se na OSPF proces
R(config-router)#auto-cost reference-bandwith <mbps>     \\ Změna referenční hodnoty
```

```
R(config)#router ospf <PID>     \\ Přepnutí se na OSPF proces
R(config-router)#neighbor <IP> cost <C>     \\ Nastavení metriky od souseda
```

### Časovače

```
R(config-if)#ip ospf hello-interval <sec>     \\ Nastavení intervalu mezi Hello pakety
R(config-if)#ip ospf dead-interval <sec>     \\ Nastavení doby, po kterou router čeká na hello paket
```

Pro co možná nejrychlejší konvergenci lze nepřímo nastavit *hello timer* až na 250ms:

```
R(config-if)#ip ospf dead-interval minimal hello-multiplier <M>     \\ Nastavení hodnot    
```

Tento příkaz nastaví *dead timer* na nejnižší hodnotu (1s) a *hello timer* pomocí dělení této hodnoty, *hello-multiplier* udává, kolikrát je *dead timer* větší, než *hello*, při nejnižší hodnotě se jedná o `1/3s`.

Toto nastavení je velmi rychlé, oproti normálnímu, ale může být velmi nestabilní a náročné na systémové prostředky.

## Zabezpečení
---
### Autentizace

- Prvotně OSPF podporoval *type 0* (*null*), *type 1* (*Clear text*) a *type 2* (*Cryptographic*) autentizaci, později se k nim přidala i podpora `SHA`.
- Autentizace se povoluje per-interface pomocí příkazu `ip ospf authentication`.
- Defaultní typ autentizace je *type 0*
- Defaultní typ autentizace lze změnit pod `area authentication` v `router ospf` módu.
- Klíče jsou vždy konfigurovány per-interface.
- `MD5` klíčů je možné mit v *key-chainu* více, umožňuje to OSPF vyměnit autentizační hesla i bez rozvázání sousedství. Pro podepsání *odchozích* paketů se používá poslední přidaný klíč, bez ohledu na číslo. Pro autentizaci *příchozích* se používá číslo klíče, které je indikované v paketu.
	- Pokud router zjistí, že pro autentizaci příchozích používá jiný klíč, než pro podepisování odchozích, spustí *OSPF key rollover*, jedná se o procedůru, při které sousedovi pošle hello paket pro každý uložený klíč, díky tomu se sousedé domluví na jednotném klíči.

#### Plain text

```
R(config-if)#ip ospf authentication     \\ Zapnutí autentizace, určení typu
R(config-if)#ip ospf authentication-key <PSSWD>     \\ Nastavení hesla
```

#### SHA/MD5 (Key-chain)

```
R(config)#key chain <NAME>     \\ Vytvoření key-chainu
R(config-keychain)#key <ID>     \\ Vytvoření klíče
R(config-keychain-key)#key-string <PASSWD>     \\ Vytvoření hesla
R(config-keychain-key)#cryptographic-algorithm <ALG>     \\ Určení algoritmu
```

```
R(config-if)#ip ospf authentication key-chain <NAME>     \\ Přiřazení key-chainu a spuštění autentizace   
```

Key-chain konfigurace je trochu specifická i v dalších věcech:

- Každý klíč musí mít specifikovaný kryptografický algoritmus
- Pokud existuje více použitelných klíčů, OSPF používá ten s *nejvyšší* ID, což je opak od [[EIGRP]] a [[RIP]]
- *OSPF key rollover* procedura se u key-chain nepoužívá
	- Odchozí pakety OSPF podepisuje klíčem s nejvyšším ID
	- Příchozí pakety autentizuje s klíčem, který je indikovaný v paketu
	- Oproti non-key-chain metodě nedochází k synchronizaci a domluvě na jednom klíči
- Tuto *Extended Cryptographic OSPF Authentication* nelze povolit skrze [[#Area]] konfiguraci, ale jím authentizovat [[#Virtual Link]]

#### MD5

```
R(config-if)#ip ospf authentication message-digest     \\ Spuštění MD5 autentizace
R(config-if)#ip ospf message-digest-key <ID> md5 <PASSWD>     \\ Vytvoření hesla
```

#### Area

Toto nastavení slouží pouze pro zapnutí autentizace na interfacech uvnitř arei, hesla se musí poté nastavit per-interface.

```
R(config-router)#area <ID> authentication {message-digest}     \\ Zapnutí autentizace
```

#### Virtual Link

Protože *Virtual Link* není fyzický interface, na který by se mohli nastavit přístupové údaje, je nutné je nastavit v `router ospf` módu.

```
R(config-router)#area <ID> virtual-link <router_ID> authentication <NASTAVENÍ>     \\ Zapnutí autentizace
```

V poli `<NASTAVENÍ>` se nadále zadávají klasické autentizační příkazy.

### TTL Security Check

Jako další bezpečnostní funkci OSPF nabízí kontrolu TTL pole IP paketu. Tato funkce je postavená na premise, že sousedská komunikace musí pocházet ze stejného segmentu, krom Virtual-Link a Sham-Link, router zkontroluje TTL paketu a pokud není rovné 255, je jasné, že nepochází ze stejného segmentu sítě a je zahozeno.
Tato funkce tak předchází útokům, při kterých koncové zařízení generuje OSPF provoz, mimo segment, a může tak způsobit DOS.

#### Per-Proccess

```
R(config-router)#ttl-security all-interfaces {hops}     \\ Zapnutí funkce na všech interfacech
```

```
R(config-if)#ip ospf ttl-security disabled     \\ Vypnutí funkce na jednotlivých interfacech
```

#### Per-Interface

```
R(config-if)#ip ospf ttl-security {hops}     \\ Zapnutí funkce na interfacu, nastavení počtu hopů
```

#### Virtual-Link/Sham-Link

```
R(config-router)#area <ID> <virtual-link/sham-link> <IP> ttl-security hops <HOPS>
```

Normální nastavení neovlivňuje Virtual/Sham-Linky, ty mají speciální nastavení.

## Area
---

### Stub

```
R(config-router)#area <AREA_NUMBER> stub     \\ Nutno nastavit na všech routerech v arei
```

### NSSA

```
R(config-router)#area <AREA_NUMBER> nssa     \\ Nutno nastavit na všech routerech v arei
```

### Totally Stub

```
R(config-router)#area <AREA_NUMBER> stub <no-summary> \\ Na všech routerech v arei stub + na ABR no-summary
```

### Totally NSSA

```
R(config-router)#area <AREA_NUMBER> nssa <no-summary> \\ Na všech routerech v arei stub + na ABR no-summary
```

### Virtual Link

```
R1(config-router)#area <TRANSIT_AREA_NUMBER> virtual-link <R2_RID>     \\ Připojení na R2
```
```
R2(config-router)#area <TRANSIT_AREA_NUMBER> virtual-link <R1_RID>     \\ Připojení na R1
```

```
R1#show ip ospf virtual-links     \\ Zobrazení informací
```

Jako číslo arei se zadává tranzitní area a cíl linky je RID.

## Filtrování & Sumarizace
---
### Filtrování

Protože nelze upravovat LSA, protože OSPF a SPF musí mít zaručeno, že věechny zařízení v arei budou mít shodné LSDB, jedná se o filtraci cest, nikoli LSA.
Pro filtraci se využívají *distribute* listy.

Příkaz pouze filtruje to, co se zabuduje routeru, na kterém je distribute list nastaven, do RIB, tento příkaz nemá, a ani nemůže mít, vliv na rozesílání LSA´s nebo LSDB/SPF.

- **inbound** distribute list se aplikuje pouze na RIB
- **outbound** distribute list se aplikuje pouze v případě, že se jedná o redistribuci
- V případě specifikování příchozího interfacu, tento interface se bere v potaz jako egress interface pro danou cestu, nikoli, jako interface, na který přišlo LSA

Způsobů, jak nastavit pravidla je, jako vždy, několik.

#### ACL

```
R(config)#ip access-list extended <NAME>     \\ Vytvoření ACL
R(cconfig-ext-nacl)#{seq} deny ip <IP> <W_MASK> any     \\ Vytvoření pravidla pro blokování
R(cconfig-ext-nacl)#{seq} permit any     \\ Vše ostatní se musí povolit
```

```
R(config-router)#distribute-list <ACL> in <IF>     \\ Specifikace výstupního interfacu
```

Pokud nechceme specifikovat výstupní interface, lze propojit ACL s route-map, poté není potřeba.

#### Prefix-list

```
R(config)#ip prefix-list <NAME> {seq} deny <IP/PREFIX> {le/ge}     \\ Specifikace prefixu pro blokaci
R(config)#ip prefix-list <NAME> {seq} permit 0.0.0.0/0 le 32     \\ Povolení všeho ostatního
```

```
R(config-router)#distribute-list prefix <NAME> in <IF>     \\ Provázání s OSPF
```

### ABR type 3 LSA filtering

Na ABR routeru, vzhledem k tomu, že ten funguje jako [[Routing#Distance-Vector|Distance-Vector]] protokol, je možné filtrovat LSA informace.
Díky tomu, že ABR generuje LSA´s, je možné zaručit jak to, že zařízení nemůže LSA upravovat, pouze přeposílat, tak to, že uvnitř arei musí mít všechny zařízení stejnou LSDB, filtrace na ABR totiž probíhá pro celou areu.
ABR tyto cesty bude na dále mít, ale zařízení uvnitř arei ne.

- Při nastavení **in** se filtruje vstup do arei
- Při nastavení **out** se filtruje výstup z arei

```
R(config)#ip prefix-list <NAME> {seq} deny <IP/PREFIX> {le/ge}     \\ Specifikace sítě pro blokaci
R(config)#ip prefix-list <NAME> {seq} permit 0.0.0.0/0 le 32     \\ Povolení všeho ostatního
```

```
R(config-router)#area <NUMBER> filter-list prefix <NAME> [in|out]     \\ Propojení s OSPF
```

### ABR no-advertise

Pomocí tohoto příkazu lze udělat hned několik věcí:
- Nastavit specifickou metriku nějaké cestě
- Manuálně sumarizovat
- Zamezit rozesílání určité cesty

Tento příkaz na ABR se specifikuje jako **out**, vztahuje se tedy pouze na jeho vlastní cesty, které redistribuuje z arei ven, nikoli z venčí do arei!

```
R(config-router)#area <NUMBER> range <IP> <MASK> [advertise|cost|not-advertise]     \\ Zapnutí pravidla
```

- **advertise**
	- Tento příkaz je defaultní, jedná se o prosté přeposílání dané sítě
	- Pomocí tohoto příkazu lze nastavit i sumarizaci
		- Pokud máme sítě `10.40.0.0/24`, `10.40.1.0/24` a `10.40.2.0/24`, můžeme nastavit následující příkaz:
			- `R(config-router)#area <NUMBER> range 10.40.0.0 255.255.0.0 advertise`
		- ABR následně bude přeposílat `/16` síť, kterou jsme specifikovali
- **cost**
	- Tento příkaz nám umožní nastavit specifickou metriku pro danou síť, místo té defaultní
- **not-advertise**
	- Zde je nástroj pro filtraci, tento příkaz blokuje rozesílání dané sítě

### ASBR

V případě ASBR existuje jiný příkaz, který umožňuje sumarizovat externí cesty:
```
R(config-router)#summary-address <ADDR> <MASK>
```

## Funkce
---

### SPF Throttling

```
R(config-router)#timers throttle spf <start> <hold> <max-wait>     \\ Nastavení SPF Throttling časovačů
```

#### LSA Throttling

```
R(config-router)#timers throttle lsa <start> <hold> <max-wait>     \\ Nastavení LSA Throttling časovačů
```

##### LSA Arrival

```
R(config-router)#timers lsa arrival <msec>     \\ Nastavení doby rozestupu mezi přijímanými LSA
```

### Incremental SPF

```
R(config-router)#ispf     \\ Zapnutí Incremental SPF kalkulace
```

### Prefix Suppression

```
R(config-router)#prefix-suppression     \\ Zapnutí funkce na všech OSPF-enabled, non-passive interfacech
```

```
R(config-if)#ip ospf prefix-suppression [disable]     \\ Vypnutí/Zapnutí na interfacu
```

### Stub Router

```
R(config-router)#max-metric router-lsa {on-startup <wait-for-bgp|sec>} <LSA_TYPE>
```

### NSF

```
R(config-router)#nsf [ietf|cisco|enforce]
```

### Graceful Shutdown

```
R(config-router)#shutdown
```

```
R(config-if)#ip ospf shutdown
```

# OSPFv3
---

```
R(config-if)#ipv6 ospf <PID> area <AID> {instance}     \\ Zahrnutí interfacu, sítě, do OSPFv3
```

## Autentizace

### IPSec

```
R(config-if)#ipv6 ospf encryption ipsec spi <SPI> esp <ALG> <BIT> <KEY> [sha-1/md5] <key>
```

```
R(config-if)#ipv6 ospf encryption ipsec spi 1024 esp aes-cbc 256 9D271CCE3B6EE8774809985B4205CF2012953772D99800A774687A6E51AE8984 sha1 17F8E414270E5B6B1CD8A81E7F77259456C55516
```

### Key-Chain

```
R(config-if)#ospfv3 authentication key-chain <KC>
```