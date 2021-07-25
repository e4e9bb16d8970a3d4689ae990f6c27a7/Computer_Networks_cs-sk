# Classic Konfigurace
---

## Základní konfigurace
---

```
R(config)#router eigrp <AS>     \\ Zapnutí EIGRP procesu pod AS
R(config-router)#network <ADDR> {MASK}     \\ Nastavení sítě a interfacu, pro kterou se má EIGRP zapnout
```

```
R(config-router)#neighbor <IP> <EGINT>     \\ Nastavení souseda, vypnutí multicast na interfacu
```

```
R(config-router)#passive-interface <IF|default>     \\ Vypnutí EIGRP na interfacu, celého
```

### IPv6

```
R(config)#ipv6 unicast-routing     \\ Zapnutí IPv6 routingu
R(config)#ipv6 router eigrp <AS>     \\ Přepnutí se na konfiguraci EIGRP
R(config-rtr)#network <ADDR> {MASK}     \\ Nastavení sítě a interfacu, pro kterou se má EIGRP zapnout
R(config-rtr)#no shutdown     \\ Zapnutí EIGRP procesu
```

## Autentizace
---
```
R(config)#key chain <NAME>     \\ Vytvoření key-chain
R(config-keychain)#key <NUMBER>     \\ Vytvoření klíče
R(config-keychain-key)#key-string <PASSWD>     \\ Uložení hesla
```

```
R(config-if)#ip authentication mode eigrp <AS> md5     \\ Zapnutí autentizace
R(config-if)#ip authentication key-chain eigrp <AS> <KEY_CHAIN>     \\ Přiřazení key-chainu
```

## Stub
---
```
R(config)#router eigrp <AS>     \\ Přepnutí se na EIGRP proces
R(config-router)#eigrp stub {ROUTE}     \\ Zapnutí Stub, možnost vybrání typů cest, které fungují
```

### leak-map

Toto je nástroj, který umožňuje, i přes stub, sdílet konkrétně vybrané sítě.
Nejprve je potřeba definovat logiku, pravidla, dle které se určují sítě, které mají být "leaknuté". Toto lze určit pomocí ACL nebo Prefix listu.
Samotná leak-mapa je ve formě route-mapy, ve které se spojí permit logika PBR s permit logikou ACL nebo Prefix listu.

```
R(config)#access-list <NUMBER> permit <NETWORK> <PREFIX>     \\ Specifikace sítě
```

```
R(config)#route-map <NAME> permit     \\ Vytvoření leak-mapy
R(config-route-map)#match ip address <ACL|prefix-list>     \\ Spojení route-mapy s logikou
```

```
R(config)#router eigrp <AS>     \\ Přepnutí se na EIGRP proces
R(config-router)# eigrp stub leak-map <NAME>     \\ Přiřazení leak-mapy
```

## Sumarizace
---
```
R(config)#interface <IF>     \\ Přepnutí se na hraniční interface
R(config-if)#ip summary-address eigrp <ADDR/PREFIX>     \\ Specifikace sítě, kterou bude posílat
```

### leak-map

Toto je nástroj, který umožňuje, i přes sumarizaci, sdílet konkrétně vybrané sítě.
Nejprve je potřeba definovat logiku, pravidla, dle které se určují sítě, které mají být "leaknuté". Toto lze určit pomocí ACL nebo Prefix listu.
Samotná leak-mapa je ve formě route-mapy, ve které se spojí permit logika PBR s permit logikou ACL nebo Prefix listu.

```
R(config)#access-list <NUMBER> permit <NETWORK> <PREFIX>     \\ Specifikace sítě
```

```
R(config)#route-map <NAME> permit     \\ Vytvoření leak-mapy
R(config-route-map)#match ip address <ACL|prefix-list>     \\ Spojení route-mapy s logikou
```

```
R(config)#interface <IF>     \\ Přepnutí se na sumarizovaný interface
R(config-if)#ip summary-address eigrp <AS> leak-map <NAME>     \\ Aplikování leak-mapy
```

## Route Filtering
---
EIGRP umožňuje fultrovat příchozí a odchozí cesty na základě několika typů pravidel:

- [[ACL]]
- [[PBR|Route map]]
- Prefix lists

### ACL

 Nastavení přes ACL je poměrně neflexibilní, ale za to jednoduché a rychlé.

```
R(config)#access-list <NUMBER> <permit|deny> <IP> <MASK>     \\ Vytvoření pravidel
R(config)#router eigrp <NUMBER>     \\ Přepnutí se na EIGRP proces
R(config-router)#distribute-list <ACL_NUMBER> <in|out> <IF>     \\ Připojení pravidel k interfacu
```

### Route-map

Nastavení přes route-map může být složité, dlouhé a pomalé, ale nástroj je to velmi rozsáhlý a flexibilní i pro řadu dalších vlastností, než jenom samotná adresa/maska.

```
R(config)#route-map <NAME> <permit|deny> <NUMBER>     \\ Vytvoření mapy a přejití na číslo řádku
R(config-route-map)#match <VLASTNOST>     \\ Vytvoření pravidla
R(config-route-map)#router eigrp <NUMBER>     \\ Přepnutí se na EIGRP proces
R(config-router)#distribute-list route-map <NAME> <in|out> <IF>     \\ Propojení mapy a procesu
```

### Prefix list

Tato možnost je nejjednodušší pro čisté filtrování na základě adres/mask, protože struktura listu je připravena na tento formát.

```
R(config)#ip prefix-list <NAME> <permit|deny> <NETWORK/PREFIX> {ge|le}     \\ Vytvoření pravidla
R(config)#router eigrp <AS>     \\ Přepnutí se do EIGRP
R(config-router)#distribute-list prefix <NAME> <in|out> <IF>     \\ Propojení mapy a procesu
```

## Redistribuce
---

```
R(config)#router eigrp <NUMBER>     \\ Přepnutí se na EIGRP proces
R(config-router)#redistribute <RP> {route-map} metric <METRIKA>     \\ Nastavení redistribuce
```

# Named Konfigurace
---

## Základní konfigurace
---

```
R(config)#router eigrp <NAME>     \\ Zapnutí EIGRP procesu
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Připojení AS
R(config-router-af)#network <ADDR> {MASK}     \\ Nastavení sítě a interfacu, pro kterou se má EIGRP zapnout
```

```
R(config-router-af)#neighbor <IP> <EGINT>     \\ Nastavení souseda, vypnutí multicast na interfacu
```

```
R(config-router-af)#af-interface <IF>     \\ Přepnutí na nastavení interfacu
R(config-router-af-interface)#passive-interface     \\ Vypnutí EIGRP na interfacu, celého
```

### IPv6

```
R(config)#ipv6 unicast-routing     \\ Zapnutí IPv6 routingu
R(config)#router eigrp <NAME>     \\ Přepnutí se do EIGRP konfigurace
R(config-router)#address-family ipv6 autonomous-system <AS>     \\ Přepnutí se na AF
R(config-router-af)#network <ADDR> {MASK}     \\ Nastavení sítě a interfacu, pro kterou se má EIGRP zapnout
```

## Autentizace
---
```
R(config)#key chain <NAME>     \\ Vytvoření key-chain
R(config-keychain)#key <NUMBER>     \\ Vytvoření klíče
R(config-keychain-key)#key-string <PASSWD>     \\ Uložení hesla
```

U Named módu lze nastavit autentizaci na všechny interfacy najednou, v případě použití default.
U `md5` je nutné nastavit heslo přes key-chain, u `sha-256` lze nastavit přímo heslo.

```
R(config)#router eigrp <NAME>     \\ Přepnutí se do EIGRP konfigurace
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí se na AF
R(config-router-af)#ad-interface <default|if>     \\ Přenutí se na interface/nastavení všech
R(config-router-af-interface)#autentication mode md5     \\ Zapnutí autentizace
R(config-router-af-interface)#autentication key-chain <NAME>     \\ Přiřazení key-chainu
```

```
R(config)#router eigrp <NAME>     \\ Přepnutí se do EIGRP konfigurace
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí se na AF
R(config-router-af)#ad-interface <default|if>     \\ Přenutí se na interface/nastavení všech
R(config-router-af-interface)#autentication mode hmac-sha-256 <PASSWD>     \\ Zapnutí autentizace, heslo
R(config-router-af-interface)#autentication key-chain <NAME>     \\ Přiřazení key-chainu
```

V případě, že je pomocí `default` parametru nastaven key-chain nebo `md5` autentizace a na jednotlivém interfacu pak chceme nepoužívat key-chain nebo `md5`, musí se to specificky nastavit.

```
R(config-router-af)#interface <IF>
R(config-router-af-interface)#autentication mode hmac-sha-256 <PASSWD>     \\ SHA256 má přednost MD5
R(config-router-af-interface)#no autentication key-chain     \\ Odpojení key-chainu
```

### Časově omezené klíče

Z důvodu bezpečnosti lze nastavit platnost klíčů na základě času.

```
R(config)#key chain <NAME>     \\ Vytvoření key-chain
R(config-keychain)#key <NUMBER>     \\ Vytvoření klíče
R(config-keychain-key)#key-string <PASSWD>     \\ Uložení hesla
R(config-keychain-key)#accept-lifetime <hh:mm:ss> <date> <month> <year> <duration | infinite | stop_time>
R(config-keychain-key)#send-lifetime <hh:mm:ss> <date> <month> <year> <duration | infinite | stop_time>
```

Ukázka vytvoření klíčů na rok:

```
R(config)#key chain EIGRP_KEYS     \\ Vytvoření key-chain
R(config-keychain)#key 1     \\ Vytvoření klíče
R(config-keychain-key)#key-string CCIE     \\ Uložení hesla
R(config-keychain-key)#accept-lifetime 14:30:00 10 APRIL 2019 duration 31536000
R(config-keychain-key)#accept-lifetime 14:30:00 10 APRIL 2019 duration 31536000
```

V případě více nakonfigurovaných, platných, klíčů se EIGRP chová tak, že pro podepisování vždy vybírá klíč s nejnižším ID a pro ověřování si vybere klíč, který odpovídá podpisu.
Díky tomu lze poměrně jednoduše provést výměnu klíčů:

1. Nejprve vytvořme nový klíč s nižším, než stávajícím, ID
2. Nadále můžeme začít nastavovat `send-lifetime` starého klíče do minulosti, to způsobí, že klíč již nebude používán pro podepisování, ale bude se používat nový klíč, nedojde tedy k výpadku, protože tento nový klíč již všechny okolní routery mají mít
3. Podé, co je toto nastaveno v celé síti, již všechny zařízení používají nové klíče a můžeme staré vymazat

## Stub
---
```
R(config)#router eigrp <NAME>     \\ Přepnutí se do EIGRP konfigurace
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí se na AF
R(config-router-af)#eigrp stub {ROUTE}     \\ Zapnutí Stub, možnost vybrání typů cest, které fungují
```

### leak-map

Toto je nástroj, který umožňuje, i přes stub, sdílet konkrétně vybrané sítě.
Nejprve je potřeba definovat logiku, pravidla, dle které se určují sítě, které mají být "leaknuté". Toto lze určit pomocí ACL nebo Prefix listu.
Samotná leak-mapa je ve formě route-mapy, ve které se spojí permit logika PBR s permit logikou ACL nebo Prefix listu.

```
R(config)#access-list <NUMBER> permit <NETWORK> <PREFIX>     \\ Specifikace sítě
```

```
R(config)#route-map <NAME> permit     \\ Vytvoření leak-mapy
R(config-route-map)#match ip address <ACL|prefix-list>     \\ Spojení route-mapy s logikou
```

```
R(config)#router eigrp <NAME>     \\ Přepnutí se do EIGRP konfigurace
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí se na AF
R(config-router-af)#eigrp stub leak-map <NAME>     \\ Přiřazení leak-mapy
```

## Sumarizace
---
```
R(config)#router eigrp <NAME>     \\ Přepnutí se na proces EIGRP
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí na AS
R(config-router-af)#af-interface <IF>     \\ Přepnutí se na hraniční interface
R(config-router-af-interface)#summary-address <ADDR/PREFIX>     \\ Specifikace adresy, kterou bude posílat
```

### leak-map

Toto je nástroj, který umožňuje, i přes sumarizaci, sdílet konkrétně vybrané sítě.
Nejprve je potřeba definovat logiku, pravidla, dle které se určují sítě, které mají být "leaknuté". Toto lze určit pomocí ACL nebo Prefix listu.
Samotná leak-mapa je ve formě route-mapy, ve které se spojí permit logika PBR s permit logikou ACL nebo Prefix listu.

```
R(config)#access-list <NUMBER> permit <NETWORK> <PREFIX>     \\ Specifikace sítě
```

```
R(config)#route-map <NAME> permit     \\ Vytvoření leak-mapy
R(config-route-map)#match ip address <ACL|prefix-list>     \\ Spojení route-mapy s logikou
```

```
R(config)#router eigrp <NAME>     \\ Přepnutí se na proces EIGRP
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí na AS
R(config-router-af)#af-interface <IF>     \\ Přepnutí se na hraniční interface
R(config-router-af-interface)#summary-address <ADDR/PREFIX> leak-map <NAME>    \\ Aplikování leak-mapy
```

## Route Filtering
---
EIGRP umožňuje fultrovat příchozí a odchozí cesty na základě několika typů pravidel:

- [[ACL]]
- [[PBR|Route map]]
- Prefix lists

### ACL

 Nastavení přes ACL je poměrně neflexibilní, ale za to jednoduché a rychlé.

```
R(config)#access-list <NUMBER> <permit|deny> <IP> <MASK>     \\ Vytvoření pravidel
R(config)#router eigrp <NAME>     \\ Přepnutí na EIGRP proces
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí na AS
R(config-router-af)#topology base     \\ Přepnutí na nastavení topologie
R(config-router-af-topology)#distribute-list <ACL_NUMBER> <in|out> <IF>     \\ Připojení pravidel k interfacu
```

### Route-map

Nastavení přes route-map může být složité, dlouhé a pomalé, ale nástroj je to velmi rozsáhlý a flexibilní i pro řadu dalších vlastností, než jenom samotná adresa/maska.

```
R(config)#route-map <NAME> <permit|deny> <NUMBER>     \\ Vytvoření mapy a přejití na číslo řádku
R(config-route-map)#match <VLASTNOST>     \\ Vytvoření pravidla
R(config)#router eigrp <NAME>     \\ Přepnutí na EIGRP proces
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí na AS
R(config-router-af)#topology base     \\ Přepnutí na nastavení topologie
R(config-router-af-topology)#distribute-list route-map <NAME> <in|out> <IF>     \\ Propojení mapy a procesu
```

### Prefix list

Tato možnost je nejjednodušší pro čisté filtrování na základě adres/mask, protože struktura listu je připravena na tento formát.

```
R(config)#ip prefix-list <NAME> <permit|deny> <NETWORK/PREFIX> {ge|le}     \\ Vytvoření pravidla
R(config)#router eigrp <NAME>     \\ Přepnutí na EIGRP proces
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí na AS
R(config-router-af)#topology base     \\ Přepnutí na nastavení topologie
R(config-router-af-topology)#distribute-list prefix <NAME> <in|out> <IF>     \\ Propojení mapy a procesu
```

## Redistribuce
---

```
R(config-router)#address-family <ipv4|ipv6> autonomous-system <AS>     \\ Přepnutí na AS
R(config-router-af)#topology base     \\ Přepnutí na nastavení topologie
R(config-router-af-topology)#redistribute <RP> {route-map} metric <METRIKA>     \\ Nastavení redistribuce
```
