[[RIP]]

# Základní příkazy
---

```
R1(config)#router rip     \\ Přepnutí na RIP konfigurace
```
```
R1(config-router)#version 2     \\ Přepnutí z RIPv1 na RIPv2
```
```
R1(config-router)#no auto-summary     \\ Vypne sumarizaci, nutno nastavit na každém routeru
```

```
R1(config-router)#network <IP>     \\ Přiřadí síť a interface do RIP routingu
```

```
R1(config-router)#passive-interface <IF>     \\ Přestane posílat Advertisements z interfacu
```

```
R1(config-router)#default-information originate     \\ Přeposílání Defaultní cesty
```

```
R1(config-router)#maximum-paths <Číslo>     \\ Udává maximální počet duplicitních cest do sítě
```

```
R1(config-router)#distance <Číslo>     \\ Změní AD
```

## Ukázková konfigurace

```
R1(config)#router rip
R1(config-router)#version 2
R1(config-router)#no auto-summary
R1(config-router)#passive-interface default     \\ Všechny interfacy přepne do passive módu
R1(config-router)#no passive-interface g0/1
R1(config-router)#no passive-interface g0/2
R1(config-router)#network 10.0.10.0
R1(config-router)#network 10.0.20.0
```

# RIPng
---

```
R1(config)#ipv6 unicast-routing     \\ Zapnutí IPv6 Routingu
R1(config)#interface <IF>
R1(config-if)#ipv6 rip <Název> enable     \\ Zapnutí RIP na Interfacu/Síti
```

Všechny následné informace se nastavují Interface-based.

# Autentizace
---

Defaultně se používá plain-text autentizace.
V případě více platných klíčů IOS vybere klíč s nejnižším číslem.

```
R1(config)#key chain <Jméno>     \\ Vytvoření Key Chainu
R1(config-keychain)# key <Číslo>     \\ Vytvoření klíče
R1(config-keychain-key)# key-string <Heslo>     \\ Vytvoření samotného hesla
```
```
R1(config)#interface <IF>     \\ Přepnutí na Interface, na kterém běží RIP
R1(config-if)#ip rip authentication key-chain <Jméno>     \\ Přiřazení Key Chainu
R1(config-if)#ip rip authentication mode <text|md5>     \\ Nastavení typu autentizace
```

Nastavení musí být naprosto shodné na všech routrech.

## Ukázková konfigurace
```
R1(config)#key chain RIPKEY
R1(config-keychain)#key 1
R1(config-keychain-key)# key-string SomePassword    
```
```
R1(config)#interface g0/1     
R1(config-if)#ip rip authentication key-chain RIPKEY     
R1(config-if)#ip rip authentication mode md5   
```

# Manuální sumarizace
---

```
R1(config)#interface <IF>     \\ Přepnutí na interface, ze kterého se májí posílat sumarizované cesty
R1(config-if)#ip summary-address <IP> <Mask>     \\ Nastavení sumarizované adresy
```

## Ukázková konfigurace

```
R1(config)#interface l0
R1(config-if)#ip address 10.0.0.1 255.255.255.255
R1(config)#interface l1
R1(config-if)#ip address 10.0.1.1 255.255.255.255
R1(config)#interface l2
R1(config-if)#ip address 10.0.2.1 255.255.255.255
R1(config)#interface g0/1
R1(config-if)ip summary-address 10.0.0.0 255.255.0.0
```

# Unicast
---

```
R1(config)#router rip
R1(config-router)#neighbor <IP>     \\ Nastavení souseda, na kterého se budou posílat Unicast zprávy
R1(config-router)#passive-interface <IF>     \\ Zablokování Multicast RIP na interfacu
```

## Ukázková konfigurace

Sousedství se sice nijak nenavazuje, ale je nutné udělat nastavení na obou routerech, abychom odesílali i přijímali Unicast.

### R1
```
R1(config)#router rip
R1(config-router)#neighbor 10.0.0.2 
R1(config-router)#passive-interface g0/1
```

### R2

```
R2(config)#router rip
R2(config-router)#neighbor 10.0.0.1   
R2(config-router)#passive-interface g0/1
```

# Timers
---

```
R1(config)#router rip
R1(config-router)#timers basic <Update> <Invalid> <Holddown> <Flush> {Sleep}
```

# Redistribuce
---

```
R1(config)#router rip
R1(config-router)#redistribute <Protokol> <Instance> {Metric}     \\ Redistribuce z ostatních RP
```

# Filtrování
---

## Network

```
R1(config)#router rip
R1(config-router)#network <IP>     \\ Zadání specifické sítě
```

## Passive-Interface

```
R1(config-router)#passive-interface <IF>
```

## Distribute List

Blokování nebo povolování sítí dle nastavení [[ACL]].

```
R1(config)#ip access-list standard <Jméno|Číslo>
R1(config-std-nacl)#deny <IP> <Mask>     \\ Přidání blokované sítě
R1(config-std-nacl)#permit <IP> <Mask>     \\ Přidání povolené sítě
R1(config-std-nacl)#permit any     \\ Povolení všech sítí
```
```
R1(config)#router rip
R1(config-router)#distribution-list <Jméno|Číslo> <out|in> <IF>     \\ Aplikování na Interface
```

## Offset list

Přidávání bodů metriky sítím dle nastavení [[ACL]].

```
R1(config)#ip access-list standard <Jméno|Číslo>
R1(config-std-nacl)#deny <IP> <Mask>     \\ Odebrání sítě
R1(config-std-nacl)#permit <IP> <Mask>     \\ Přidání sítě
R1(config-std-nacl)#permit any     \\ Přidání všech sítí
```
```
R1(config)#router rip
R1(config-router)#offset-list <Jméno|Číslo> <in|out> <IF>  \\ Aplikování na interface 
```

# Triggered
---

Na Sériových linkách RIP umožňuje změnit nastavení Triggerů pro posílání Advertisements .

```
R1(config)#interface serial <Označení>     \\ Nastavení je dostupné pouze pro Seriové linky
R1(config-if)#ip rip triggered 
```

RIP bude posílad Advertisementy pouze
- Při Requestu - Celá RIB
- Při změně RIB - Část RIB
- Při zapnutí/vypnutí interfacu - Část RIB
- Při zapnutí Routeru - Celá RIB

# Změna AD
---

```
R1(config-router)#distance <AD> {SRC_IP} {Mask}     \\ Přiřazení AD celému procesu, nebo cestě
```

Změna AD je pouze lokální, nepropaguje se nadále RIP doménou.
