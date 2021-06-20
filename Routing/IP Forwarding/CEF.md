# Cisco Express Forwarding
---

## Popis
---
Je to defaultní metoda přepínání paketů pro všechny Cisco zařízení od 90. let, je defaultní metodou pro zařízení, které využívají *Application-specific integrated circuits* (*ASICs*) a *Network Processing Units* (*NPUs*).

Při inicializaci se přesune obsah RIB do speciální tabulky, *FIB* (*Forwarding Information Base*).

Podoba samotné FIB záleží na platformě, většinou se jedná o speciální hardware, díky kterému se zásadně zvyšuje rychlost.

Pokud na zařízení není dostatek místa pro FIB, změní se metoda z CEF na Fast Switching.

Vzhledem k tomu, že nejnáročnější proces pri routingu je zjištování L2 informací z RIB a L2 tabulek, na základě kterých se vytvoří L2 Hlavička, tento CEF optimalizuje.

Ve většině případů může mít sice RIB stovky až stovky tisíc záznamů, ale next-hop L2 a L3 adres má většinou omezené množství, nemá tedy smysl prohledávat celou RIB a pro každý záznam vytvářet novou L2 hlavičku, ale je lepší vytvořit tabulku síti (FIB), ze které záznamy odkazují na tabulku s L2 informacemi (*Adjacency Table*), které jsou již předpřipravené a uložené.

Další důležité zrychlení oproti RIB je, že RIB není svou strukturou navržena pro rychlé prohledávání, jedná se o tabulku obsahující jednoznačné routing informace postavené nad různými protokoly, ale obsahuje spoustu nadbytečných informací, jako AD, Metriku a podobně, a naopak některé informace nemusí na první prohledání obsahovat, routa může obsahovat pouze next-hop IP adresu a interface, na základě kterého má router zjistit L2 informace, musí zjistit až při dalším prohledávání. Tím, že FIB obsahuje pouze síť a odkaz na Adjecency table značně urychluje proces.

Z hlediska relačních databází by se pole **Adjacency** u FIB dalo považovat za *Foreing Key* odkazující na pole **IP Address** u Adjacency Table (*Primary Key*).

#### Vyjímky

CEF proces nedokáže pracovat s některými druhy provozu:

- Manipulace s IP hlavičkou
- Přeposílání na *Tunnel* Interface
- Překročení MTU
- [[IGMP]] přesměrování

### FIB

|Network|Prefix|Adjacency|
|:-:|:-:|:-:|
|10.0.0.0|/24|10.0.0.254|
|192.168.0.0|/16|attached|
|0.0.0.0|/0|172.16.0.1|

Jedná se o upravenou verzi RIB obsahující pouze síť a odkaz na Adjecency Table.

Je organizovaná do stromové struktury a případě Hardware CEF je postavena na [[TCAM]], což jí také oproti RIB zrychluje.

Jiné protokoly mohou mít různé FIBs, například [[MPLS]] využívá LFIB, která je kopií LIB.

### Adjacency Table

|IP Address|Next Hop MAC|Interface|Egress MAC|
|:-:|:-:|:-:|:-:|
|10.0.0.254|0062.ec9d.c546|Gi0/0/1|0042.ec9d.c515|
|172.16.0.1|0062.ec9d.c546|Gi0/0/1|0184.ac54.b4154|

Obsahuje informace o L2 next-hop adresách.
V některých případech obsahuje, krom normálních dat, i speciální záznamy:
- *Null Adjacency*
	- Paket je směrován na *Null* interface (smazán)
- *Glean Adjacency*
	- Router nezná L2 adresu, dojde k ARP Requestu
- *Punt Adjacency*
	- Z nějakého důvodu, nejčastěji bezpečnostního, paket musí být poslán do [[IP Forwarding#Process Switching|Process Switching]]
- *Discard Adjacency*
	- Pakety jsou zahazovány z důvodů [[ACL]], [[PBR]] a podobně
- *Drop Adjacency*
	- Pakety na tuto L3 adresu jsou zahazovány z důvodu kompatibility, protokolu, enkapsulace ...

![[fib.png]]

V Cisco IOS `show` příkazu se tyto tabulky spojují:

```R
R#show ip cef     
Prefix               Next Hop             Interface
0.0.0.0/0            10.0.0.1             GigabitEthernet0/0
0.0.0.0/8            drop
0.0.0.0/32           receive              
10.0.0.0/24          attached             GigabitEthernet0/0
10.0.0.0/32          receive              GigabitEthernet0/0
10.0.0.1/32          attached             GigabitEthernet0/0
10.0.0.43/32         attached             GigabitEthernet0/0
10.0.0.135/32        receive              GigabitEthernet0/0
10.0.0.255/32        receive              GigabitEthernet0/0
10.0.1.0/24          attached             GigabitEthernet0/1
10.0.1.0/32          receive              GigabitEthernet0/1
10.0.1.1/32          receive              GigabitEthernet0/1
10.0.1.2/32          attached             GigabitEthernet0/1
10.0.1.255/32        receive              GigabitEthernet0/1
10.0.2.0/24          10.0.1.2             GigabitEthernet0/1
127.0.0.0/8          drop
224.0.0.0/4          drop
224.0.0.0/24         receive              
240.0.0.0/4          drop
255.255.255.255/32   receive              
```

## Módy
---
![[dcef.png]]

### Central CEF (Default)

Jedná se o jeden proces operující s celou komunikací, většinou bývá implementován jako Software CEF.

### Distributed CEF (dCEF)

Jedná se o více procesu v rámci linecards, implementovaný jako Hardware CEF.

### Software CEF

Výše popsaný proces CEFu je implementovaný v pámci operačního systému a tabulky FIB a Adjacency jsou vytvořené v operační paměti, do které se dotazuje CPU v rámci *interrupt handler*, což napravuje problém Process Switchingu, u kterého se proces spouští periodicky, zde je spuštěn vždy při příchodu paketu.

### Hardware CEF

Router pro CEF procef využívá *ASIC*, které jsou oproti *Main Purpose CPU*, ale i *NPU* zásadně rychlejší pro výpočty. FIB tabulka se uložena v [[TCAM]] podobě, což umožňuje ještě více snížit latenci.
Zároveň je jeho implementace většinou dCEF, takže provoz je rozložen mezi více linecarts.

## Load-Sharing
---

CEF nabízí 2 způsoby Load Sharingu.

### Per-Packer

Každý příchozí paket pošle na jiný interface.
Tato metoda sice mnohem lépe utilizuje všechny možné cesty, ale může způsobovat zpoždění určitých paketů a tím pádem jejich re-ordering...

Na některých zařízeních ani není podporována a není doporučována.

### Per-Destination (Flow/Session)

Vytváří *sessions* na základě síťových informací, defaultně SRC a DST IP adresy, nad těmito informacemi spočítá hash, který jednoznačně označuje flow.
Mezi těmito sessionami pak load-sharuje.

Jedná se o defaultní a doporučené nastavení.

V případě potřeby lze do výpočtu hashe, který označuje session, započítat i jiné informace, například L4 porty, z různých důvodů, nejčastěji například z důvodu [[NAT]].

Algoritmy se mohou lišit dle platformy, často MLS s Hardware CEF nepodporují změny, zatímco Software-based CEF ISR routery ano.

Realný load-sharing se provádí tak, že mezi FIB a Adjacency Table je vložena nová tabulka, *load-share table*, která obsahuje až 16 pointerů odkazujících na Adjacency Table, tyto pointery jsou přesně rozdělené v poměru *route cost* , což znamená například, že pokud máme 3 cesty se stejnou metrikou, pak se bude rozdělovat 15 pointerů a každá cesta jich dostane 5, pokud budou 2 cesty, každá z nich dostane 8 pointerů.
Mezi těmito pointery pak přeskakují jednotlivé hashe.

#### Algoritmy

##### Original

Původní algoritmus, který je pouze na základě Cílové a Zdrojové IP adresy.7

##### Universal

Rozšířený algoritmus používající *seed number* (*Universal ID*) spolu s Origina algoritmem a zabraňuje tak [[#CEF Polarization]].

##### Tunnel

Jedná se o vylepšený Universal algoritmus, který je určený pro síť s rozšířeným používáním tunelů, což vede k relativně nízkému počtu SRC-DST IP párů.

##### L4 Port

Jedná se o vylepšený Universal algoritmus, který krom SRC-DST IP počítá i s porty.
Hodí se ho použít například za [[NAT|NATem]].

#### CEF Polarization

Z logiky nyní vyplývá, že pokud vytvoříme hash na základě SRC a DST IP adresy, které se v průběhu přenosu nemění, tento hash bude po celou dobu přenosu stejný.
Tady nastává problém, jestliže load-sharujeme mezi 2 routery například 4 sessiony, pak každý z nich dostane 2, load-sharing zatím funguje, ale pokud i oni mají 2 cesty mezi kterými chtějí load-sharovat a využívají stejný algoritmus, jako my, jejich kompletní komunikace skončí se stejným hashem a tedy i se stejným next-hopem, load-sharing přestane fungovat.
Tomuto fenoménu se říká *CEF Polarization*.

Aby se tomuto v síti vyhlo, do výpočtu hashe se nepromítá jenom nakonfigurovaná část paketu, defaultně SRC a DST IP, ale i náhodně vybrané 4B dlouhé číslo,***Universal ID***, což zaručuje originálnost hashe. 



## Konfigurace
---

```
R(config)#ip cef     \\ Zapnutí CEF na všech interfacech
R(config)#ipv6 cef     \\ Zapnutí CEF pro IPv6
```

Lze mít zapnutý CEF pouze pro IPv4 a ne pro IPv6, ale ne obráceně.

```
R(config-if)#no ip route-cache cef     \\ Selektivní vypnutí CEF na interfacu
```

### Algoritmy

```
R(config)#ip cef load-sharing algorithm <ALGORITMUS>     \\ Globální určení algoritmu pro IPv4
R(config)#ipv6 cef load-sharing algorithm <ALGORITMUS>     \\ Globální určení algoritmu pro IPv6
```

```
R(config-if)#ip cef load-sharing algorithm <ALGORITMUS>     \\ Interfacové určení algoritmu pro IPv4
R(config-if)#ipv6 cef load-sharing algorithm <ALGORITMUS>     \\ Interfacové určení algoritmu pro IPv6
```

Na platformách typu `Catalyst 6500` jsou z důvodu trochu jiného řešení problému polarizace použity jiné příkazy:

```
SW(config)#mls ip cef load-sharing
```

```
SW(config)#mls ip cef load-sharing full \\ Používá L3 a L4 adresy, nepoužívá universal ID
SW(config)#mls ip cef load-sharing simple \\ Používá pouze L3 adresy, nepoužívá Universal ID
SW(config)#mls ip cef load-sharing full simple \\ Používá L3 a L4 adresy, nepoužívá universal ID, všechny cesty mají stejnou hodnotu
```

### Show

```
R#show ip cef summary
```

```
R#show ip cef detail
```

```
R#show cef state
```
