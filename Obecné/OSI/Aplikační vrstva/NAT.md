# NAT
---

Z důvodu nedostatku IPv4 veřejných adres se rozšířil NAT, který na výstupním prvku sítě překládá Local IPv4 adresy na Public IPv4 adresy a umožňuje tak komunikaci po Internetu.

- Inside Local
  - Lokální adresa uvnitř organizace
- Inside Global
  - Veřejná adresa uvnitř organizace
- Outside Local
  - Lokální adresa jiné organizace, nespadající pod naší kontrolu
- Outside Global
  - Lokální adresa jiné organizace, nespadající pod naší kontrolu

## Static NAT

Překládá jednu *Inside Local* adresu na jednu *Inside Global* adresu. Při cestě zpět překládá i *Outside Global/Local* na *Inside Local*.
Nastavení statických adres je jednoduché, nepotřebujeme každému zařízení v síti dávat veřejnou adresu a přesto má plnou konektivitu v Internetu, ale neřeší nedostatek veřejných IP adres, protože na každou lokální je nutné mít i veřejnou adresu.
Proto je vhodné použít toto nastavení například pro jeden vnitřní server, který má mít plnou konektivitu do Internetu a pro ostatní zařízení použít [[#Dynamic NAT]].

```
R(config)#ip nat inside source static <INSIDE_LOCAL> <INSIDE_GLOBAL>     \\ Přiřazení veřejné adresy lokální adrese
```

```
R(config-if)#ip nat outside     \\ Nastavení interfacu s Inside Global konektivitou
R(config-if)#ip nat inside     \\ Nastavení interfacu s Inside Local konektivitou
```

## Dynamic NAT 

*IP masquerading*

Tento NAT definuje ACL pomocí kterého přiřadí měně Inside Global adres více Inside Local adresám, v praxi klidně celou síť jedné veřejné adrese.

```
R(config)#access-list <NUMBER> permit <IP> <RMASK>     \\ Určení sítě/í
R(config)#ip nat pool <NAME> <SIP> <EIP> <PREFIX>     \\ Určení rozsahu Inside Global adres 
```

```
R(config)#ip nat inside source list <LOCAL_ACL> pool <NAT_POOL>     \\ Přiřazení poolů NAT procesu 
```

```
R(config-if)#ip nat outside     \\ Nastavení interfacu s Inside Global konektivitou
R(config-if)#ip nat inside     \\ Nastavení interfacu s Inside Local konektivitou
```

## PAT
---

*Port Address Translation*, *NAT Overload*

Protože předešlé techniky nějakým způsobem omezovali velikost adresních rozsahů, u [[#Static NAT]] musíme mít stejný počet Inside Local a Inside Global adres a u [[#Dynamic NAT]] se může stát, že nám dojdou adresy z poolu, PAT nevytváří logiku přiřazování adres na základě Inside Local adres, ale na základě portů.

U klasického NAT si router musí pamatovat, že například `192.168.0.25` adresa se mu zrovna překládá na `32.75.87.41` adresu a příchozí komunikaci na tuto adresu zase překládá na `192.168.0.25`.
U PAT si pamatuje, že socket je `192.168.0.25:1005` a ten přeloží na `32.75.87.41:1006`, pokud mu na tuto adresu s tímto portem příjde komunikace, už bude vědět, že port `1006` patří k adrese `192.168.0.25`.

PAT lze používat jak s [[#Dynamic NAT]], tak pouze s IP adresou interfacu, vzhledem k tomu, že logika je založená na portech a ne na IP adresách.

### PAT

```
R(config)#access-list <NUMBER> permit <IP> <RMASK>     \\ Určení sítě/í
```

```
R(config)#ip nat inside source list <LOCAL_ACL> interface <IF> overload   \\ Přiřazení poolů NAT procesu 
```

```
R(config-if)#ip nat outside     \\ Nastavení interfacu s Inside Global konektivitou
R(config-if)#ip nat inside     \\ Nastavení interfacu s Inside Local konektivitou
```

### Dynamic NAT & PAT

```
R(config)#access-list <NUMBER> permit <IP> <RMASK>     \\ Určení sítě/í
R(config)#ip nat pool <NAME> <SIP> <EIP> <PREFIX>     \\ Určení rozsahu Inside Global adres 
```

```
R(config)#ip nat inside source list <LOCAL_ACL> pool <NAT_POOL> overload     \\ Přiřazení poolů NAT procesu 
```

```
R(config-if)#ip nat outside     \\ Nastavení interfacu s Inside Global konektivitou
R(config-if)#ip nat inside     \\ Nastavení interfacu s Inside Local konektivitou
```

### Port Forwarding

Jedná se v podstatě o manuální zadání záznamů do překladové tabulky, řekneme routeru, že cokoli příjde například na port `80` má poslat na specifickou adresu.
Nejčastěji se používá ve scénáři, kde máme pouze jednu veřejnou adresu, ale více serverů, které musí být přístupné zvně sítě, poté nástavíme, že port `443` se má přeposlat na lokální adresu webového serveru, `21` na vnitřní adresu FTP serveru a tak dále...

```
R(config)#ip nat inside source static {tcp|udp} <LOCAL_IP> <LOCAL_PORT> <GLOBAL_IP> <GLOBAL_PORT> [extendable]     \\ Vytvoření statického záznamu
```
