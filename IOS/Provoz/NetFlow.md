# NetFlow
---

Jedná se o Cisco technologii, která umožňuje sledovat konkrétní provoz na síti, aby na něj adminitrátor mohl reagovat.
Tento nástroj je v Cisco zařízeních již dlouhou dobu a nachází se v několika hlavních verzích: 1, 5, 9.
Následně Cisco tento nástroj přejmenovalo na *Cisco Flexible NetFlow*, nejedná se pouze o změnu jména, ale i funkce, která je popsaná v [[#Flexible NetFlow FNF]].
Netflow může být velkou zátěží na systémové prostředky, zejména na operační pamět!

NetFlow vytváří statistiky na základě *flows* v síti, jednosměrném provozu, který má společné rysy:

- Zdrojová a Cílová IP
- Zdrojový a Cílový port
- L3 typ protokolu
- ToS (QoS)
- Vstupní interface

Samotná instance se skládá z několika věcí:

- Records
  - Jedná se o, u klasického NetFlow, o předem definované seznamy, dle kterých se data seskupují a ukládají.
  - U Flexible NetFlow si je uživatel může nadefinovat sám.
- Cache
  - Jedná se o lokální místo v operační paměti, do kterého se ukládají data lokálně.
- Exporter
  - Přeposílají obsah Cache do nějakého centrálního prvku, serveru, na kterém běží analyzer.
- Flow Monitor
  - Jedná se o spojení Records, Cache a dle nastavení i Exporteru, s interfacem ze kterého se data sbírají.
- Flow samplers
  - Pro snížení systémových nároků lze určit poměr paketů, které se mají analyzovat.
  - Od 1:2 po 1:32768
- Flow Collector
  - Jedná se o aplikaci běžící na serveru, která analyzuje data.
  - Cisco DNA Center, Cisco Prime Infrastructure, PRTG a další...

## Konfigurace

### Základní

```
R(config)#ip flow-export version 9     \\ Určení verze
R(config)#ip flow-export destination <IP> <PORT> {udp | scp}     \\ Určení místa, kam se mají data posílat
```

```
R(config)#interface <IF>     \\ Přepnutí na interface, ze kterého chceme sbírat data
R(config-if)#ip flow ingress     \\ Sběr příchozích dat
R(config-if)#ip flow egress     \\ Sběr odchozích dat
```

### Verifikace

```
R#show ip flow interface     \\ Zobrazení interfaců a jejich nastavení (in, eg)
R#show ip flow export     \\ Zobrazení informací o exportování
R#show ip cache flow     \\ Zobrazení samotných dat
```

### Ostatní

Top-talkers je velmi šikovné nastavení, pokud chceme rychle zjistit, kdo má největší zátěž na síti.

```
R(config)#ip flow-top-talkers     \\ Přepnutí do nastavení top-talkers
R(config-flow-top-talkers)#sort-by <bytes | packets>     \\ Dle čeho se mají řadit
R(config-flow-top-talkers)#top <NUMBER>     \\ Kolik se jich má zobrazovat    
```

```
R#show ip flow top-talkers
```

```
R(config)#ip flow-cache entries <1024-524288>     \\ Určení velikosti lokální paměti
R(config)#ip flow-cache timeout <active | inactive> <min>     \\ Určení času, po kterém se záznam smaže, defaultně 60m
```

# Flexible NetFlow (FNF)
---

Jedná se o vylepšení původního systéme, který nabízel pouze předdefinované sety, dle kterých sbáral a třídil informace, Flexible NetFlow umožňuje administrátorovi nakonfigurovat si pravidla sám, specificky pro jeho síť.

## Cache

Flexible NetFlow představuje 3 typy cache:

### Normal
- Normální cache používající timeout timery pro čištění
- U NetFlow cache je nejnižší active timer 60s, zde 1s, což umožňuje posílat data kolektoru téměř realtime
- `active` timer určuje čas, po kterém se smažou aktivní *flows*, defaultně 30m
- `inactive` timer určuje čas po kterém se smažou neaktivní *flows*, tedy ty, které například u TCP ukončily spojení, defautlně 15s
- Na konci `active` i `inactive` timeru se exportuje cache na určený server, je li
- V případě, že data do cache přicházejí rychleji, než se jich je schopna zbavovat, NetFlow služba zapojí heuristiku a určí, které záznamy exportuje na server a smaže

```
R(config)#flow monitor <NAME> 
R(config-flow-monitor)#cache timeout <active | inactive> <1-604800s>
```

### Permanent
- Nedochází k obměně *Flows*
- Pokud dojde místo v paměti, další záznamy se neukládají
- `update` interval určuje čas, za který se posílají informace na určený server

```
R(config)#flow monitor <NAME> 
R(config-flow-monitor)#cache type permanent
R(config-flow-monitor)#cache timeout update <1-604800s>
```

### Immediate
- Okamžitě po vytvoření *Flow* se zase smaže, což má za následek, že každý paket má vlastní *Flow*
- Lze použít v případě, že očekáváme krátké komunikace a požadujeme nízkou latency pro analýzu

```
R(config)#flow monitor <NAME> 
R(config-flow-monitor)#cache type immediate
```

### Synchronized 
- Tato cache má shodné timery active a inactive a tudíš se exportují data pouze v jednom timeru 
- Nastavuje pouze jeden interval, ve kterém se mažou všechna data a exportují na server

```
R(config)#flow monitor <NAME> 
R(config-flow-monitor)#cache type synchronized
R(config-flow-monitor)#cache timeout synchronized 	<1-86400s>
```

## Nastavení

### Record

```
R(config)#flow record <NAME>     \\ Vytvoření pravidla
R(config-flow-record)#description <POPIS>     \\ Nastavení popisu pro lepší vyhledávání
R(config-flow-record)#match <FUNKCE>     \\ Nastavení informace, dle které se identifikuje flow
R(config-flow-record)#collect <FUNKCE>     \\ Nastavení informací, které se mají sbírat z provozu
R#show flow record <NAME>
```
### Exporter

```
R(config)#flow exporter <NAME>     \\ Vytvoření exporteru
R(config-flow-exporter)#description <POPIS>
R(config-flow-exporter)#destination <IP>     \\ Nastavení adresy kolektoru
R(config-flow-exporter)#export-protocol netflow-v9     \\ Nastavení verze, záleží na kolektoru
R(config-flow-exporter)#transport udp <PORT>     \\ Nastavení portu
R(config-flow-exporter)#source <IF>     \\ Nastavení zdrojového interfacu a jeho IP
```

### Monitor

Tady se spojí Record s Exporterem

```
R(config)#flow monitor <NAME>     \\ Vytvoření monitoru
R(config-flow-monitor)#description <POPIS>
R(config-flow-monitor)#record <R_NAME>     \\ Provázání s pravidly
R(config-flow-monitor)#exporter <E_NAME>     \\ Provázání s exporterem
```

```
R(config)#interface <IF>     \\ Přepnutí na interface, ze kterého chceme získávat data
R(config-if)#ip flow monitor <M_NAME> <input | output | ...>     \\ Nastavení módu
```

```
R(config-if)#ip flow monitor <M_NAME>
```

```
R(config-flow-monitor)#cache <active | inactive> timeout <sec>     \\ Nastavení maximáního stáří záznamu
R(config-flow-monitor)#cache entries <16-1048576>     \\ Nastavení velikosti cache
```

### Sampler

```
R(config)#sampler <NAME>     \\ Vytvoření pravidla pro sample
R(config-sampler)#mode random 1 out-of <2-32768>     \\ Nastavení poměru
```

```
R(config)#interface <IF>     \\ Přepnutí na interface, ze kterého chceme získávat data
R(config-if)#ip flow monitor <M_NAME> sampler <S_NAME> <input | output | ...>     \\ Propojení s interfacem
```

### Verifikace

```
R#show flow record <NAME>
R#show flow exporter <NAME>
R#show flow monitor <NAME>
```

```
R#show flow monitor <M_NAME> cache     \\ Zobrazení lokálních záznamů
R1#show flow exporter statistics     \\ Zobrazní informace o exportování
```