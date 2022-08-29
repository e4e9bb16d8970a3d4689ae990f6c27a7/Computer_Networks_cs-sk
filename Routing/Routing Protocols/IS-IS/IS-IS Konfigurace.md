# IS-IS Konfigurace
---

## Základní konfigurace

```
R(config)#router isis {TAG}     \\ Vytvoření IS-IS procesu a přechod do jeho nastavení
R(config-router)#net <NSAP>     \\ Nastavení NSAP adresy
```

```
R(config-router)#is-type <LEVEL>     \\ Nastavení levelu, umístění, routeru
```

```
R(config-router)#metric-style <wide|narrow|transition> {level} {transition}     \\ Nastavení typu metriky
```

```
R(config-router)#log-adjacency-changes {all}     \\ Zapnutí klasického logování, jako OSPF ...
```

```
R(config-if)#ip{v6} router isis {TAG}     \\ Zapnutí iIS-IS na interfacu a sdílení jeho sítě 
```

### Ukázková konfigurace

```
R(config)#router isis
R(config-router)#net 49.0001.5000.0001.0000.00
R(config-router)#is-type level-1
R(config-router)#metric-style wide transision
R(config-router)#log-adjacency-changes all
R(config-router)#interface g0/0
R(config-if)#ip router isis
```

### Passive Interface

```
R(config-router)#passive-interface <IF|default>     \\ Určení pasivního interfacu
```

Oproti ostatním protokolům, nejenom, že na tomto interfacu router nebude posílat IS-IS PDU, ale zároveň přidá jeho IP do IS-IS procesu.

#### Adresa

```
49.<Area ID>.<System ID>.00
```
`System ID` může být nejčastěji [[MAC|MAC]] adresa, nebo i IP adresa, ale musí být, v případě L1 v arei, v případě L2 v celé doméně, jedinečná.
IP adresa se většinou upravuje tímto způsobem:
- `10.1.1.1` -> `010.001.001.001` -> `0100.0100.1001`

Není přímo nutné ji upravovat na `xxxx.xxxx` notaci, i v případě `xxx.xxx` nebo `xxxxxx` notací si router umí adresu přebrat.

## Network
---
```
R(config-if)#isis network  point-to-point     \\ nastavení spoje na P-t-P
```

```
R(config-if)#isis three-way-handshake <cisco|ietf>     \\ Určení způsobu navazování komunikace
```

## Metrika
---
```
R(config-if)#isis metric <default> {{delay} {expense} {error} {level}}     \\ Určení metriky linky
```

## Autentizace
---
### IIH

#### Key Chain

```
R(config)#key chain <KNAME>
R(config-keychain)#key <ID>
R(config-keychain-key)#key-string <PASSWD>
```

```
R(config-if)#isis authentication mode <text|md5>     \\ Určení způsobu předání hesla
R(config-if)#isis authentication key-chain <KNAME> {level}     \\ Přiřazení hesla
```

#### Password

```
R(config-if)#isis password <PASSWD> {level}
```

### LSP/CSNP/PSNP

```
R(config)#key chain <KNAME>
R(config-keychain)#key <ID>
R(config-keychain-key)#key-string <PASSWD>
```

```
R(config-router)#isis authentication mode <text|md5> \\ Určení způsobu předání hesla
R(config-router)#isis authentication key-chain <KNAME> {level} \\ Přiřazení hesla
```

## Sumarizace
---
```
R(config-router)#summary-address <IP> <MASK> {metric|level|tag}     \\ Určení adresy 
```

1. Síť, patřící pod sumární adresu, musí na zařízení existovat.
2. Zařízení musí podporovat *wide* metriku

## Časovače
---
### Hello

 ```
 R(config-if)#isis hello-interval <seconds|minimal> {level}     \\ Nastavení hello timeru
 ```

### Hold

 ```
 R(config-if)#isis hello-multiplier <multiplier> {level}     \\ Nastavení hold timeru
 ```