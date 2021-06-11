# Remote Monitoring (RMON)
---

Jedná se o rozšíření [[SNMP]], které je zabudováno v standardní MIB.
Lze ho použít na routerech i switchích, jeho hlavní využítí je pro sledování určitého, proměnlivého, stavu například zatížení CPU nebo provozu na interfacu.
Umožňuje sledovat hodnoty různých OID a při překročení nastavených hodnot vygenerovat SNMP Trap.

Tato funkce je vhodná zejména u ne-Cisco zařízení, jelikož na Ciscu je vhodnější použít [[NetFlow]] (pro případ sběru statistik o provozu na interfacu).

Při nastavování monitoru je nutné vybrat jeden ze 2 verzí samplingu:

- Delta
  - Jeho hodnoty konstantně buď rostou, nebo klesají 
  - Například počet CRC errors
- Absolute
  - Jejich hodnota může soupat i klesat, různě kolísá
  - Napříklat zatížení CPU

## Nastavení

Nastavují se 2 věci:

### Alarm

Nastavení detektoru, který v případě překročení hodnot spustí event.

```
R(config)#rmon alarm <NUMBER> <MIB_OID> <SAMPLE_I> <absolute | delta> rising-threshold <rising-threshold> <EVENT> falling-threshold <falling-threshold> <EVENT>
```

### Event

```
R(config)#rmon event <NUMBER> trap <SNMP_C>     \\ Vyšle trap s informací o stavu
```

```
R(config)#rmon event <NUMBER> log     \\ Zaloguje informaco
```

```
R(config)#rmon event <NUMBER> owner <NAME> description <POPIS>     \\ Nastavení pro lepší vyhledávání
```

### Statistics

Tato funkce umožňuje sbírat data, které přicházejí na interface a provádět nad nimi vyhodnocování.

- native
  - Vyhodnocuje pouze data, která jsou určena jemu
- promiscuous
  - Vyhodnocuje všechny data, na která příjde

```
R(config)#interface <IF>     \\ Přepnutí na interface, ze kterého chceme sbírat data
R(config-if)#rmon <native | promiscuous>     \\ Nastavení režimu
R(config-if)#rmon collection <TYP> <ID>     \\ Nastavení co všechno se má vyhodnocovat
```

```
R#show rmon statistics     \\ Zobrazení hodnot na interfacu
```