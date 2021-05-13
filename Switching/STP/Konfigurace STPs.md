# STP
---

## Mód

Cisco prvky nabízí 3 módy:
- [[Per-VLAN STPs#PVSTPs|pvst]]
- [[Per-VLAN STPs#Verze|rapid-pvst]]
- [[MSTP|mst]]

I přes značení jsou [[Per-VLAN STPs]] verze plusové, rozdíl je pouze v použítí [[STP|Legacy STP]] vs [[Rapid STP]].

```
SW(config)#spanning-tree mode {mst|pvst|rapid-pvst}
```

## Priorita
---

### Manuální

```
SW(config)#spanning-tree vlan <VLAN_RANGE> priority <PRIORITY>     \\ Nastavení manuální priority
```

### Makra

```
SW(config)#spanning-tree vlan <VLAN_RANGE> root primary     \\ Makro, sníží prioritu o 8192 (24576)
```

Pokud je hodnota RBID jiná, než defaultní, tak sníží o 4096 oproti této hodnotě.

```
SW(config)#spanning-tree vlan <VLAN_RANGE> root secondary     \\ Makro, sníží prioritu o 4096 (28472)
```


### Port Cost

V případě, že chceme prioritizovat [[STP Terminologie#Porty#Root|RP]], je možné manuálně určit vyšší cenu ostatních portů.

```
SW(config-if)#spanning-tree vlan <VLAN_RANGE> cost <COST>     \\ Nastavení ceny pro RPC
```

### Port Priority

```
SW(config-if)#spanning-tree vlan <VLAN_RANGE> port-priority <PRIORITY>    
```

## Rozsah

V případě potřeby zmenšení sítě je možné přenastavit diameter, maximální počet hopů, z defaultních 7 až na 2.

```
SW(config)#spanning-tree vlan <VLAN_RANGE> root primary diameter <2-7>   
```

## Funkce
---

### Guards

#### [[STP Funkce#STP Loop Guard|Loop Guard]]

```
SW(config-if)#spanning-tree guard loop     \\ Zapnutí funkce loop guard na portu
```

```
SW(config)#spanning-tree loopguard default     \\ Zapnutí funkce loop guard na všech portech
```

#### [[STP Funkce#Root Guard|Root Guard]]

```
SW(config-if)#spanning-tree guard root     \\ Zapnutí funkce root guard na portu
```

#### [[STP Funkce#BPDU Guard|BPDU Guard]]

```
SW(config-if)#spanning-tree bpduguard <ENABLE|DISABLE>     \\ Zapnutí/Vypnutí funkce bpduguard na portu
```

```
SW(config)#spanning-tree portfast edge bpduguard default  \\ Zapnutí funkce bpduguard na všech portech
```

### [[STP Funkce#BPDU Filter|BPDU Filter]]

```
SW(config-if)#spanning-tree bpdufilter <ENABLE|DISABLE>     \\ Zapnutí/Vypnutí funkce bpdufilter na portu
```

```
SW(config)#spanning-tree portfast edge bpdufilter default  \\ Zapnutí funkce bpdufilter na všech portech
```

### [[STP Funkce#STP Portfast|Portfast]]

```
SW(config-if)#spanning-tree portfast edge      \\ Zapnutí funkcionality na koncovém portu
```

```
SW(config-if)#spanning-tree portfast edge trunk     \\ Zapnutí pro trunk na interfacu
```

```
SW(config-if)#spanning-tree portfast network      \\ Zapnutí v network stavu
```

```
SW(config-if)#spanning-tree portfast disabled     \\ Vypnutí
```

#### Edge vs Network

Edge po dostání BPDU se přepne do běžného módu.
Network nemá tuto funkcionalitu a zůstává v Portfast režimu za všech okolností, a tedy může způsobit smyčku.

Užití varianty network zároveň zapne [[STP Funkce#Bridge Assurance|Bridge Assurance]].

#### Globaly

```
SW(config)#spanning-tree portfast network default
```

Zapne portfast na všech non-edge portech + [[STP Funkce#Bridge Assurance|Bridge Assurance]].

```
SW(config)#spanning-tree portfast normal default 
```

Přepne porty do defaultního režimu, tedy portfast pouze manuálně.

```
SW(config)#spanning-tree portfast edge default
```

Zapne portfast na všech access portech.

### [[STP Funkce#UplinkFast|UplinkFast]]

```
SW(config)#spanning-tree uplinkfast     \\ Zapnutí funkce
```

## Timers

Je doporučené je neměnit, nastavují se pouze na RB, tato změna se projeví do celé topologie.

```
SW(config)#spanning-tree vlan <VLAN_RANGE> forward-time <TIME>
```

```
SW(config)#spanning-tree vlan <VLAN_RANGE> hello-time <TIME>
```

```
SW(config)#spanning-tree vlan <VLAN_RANGE> max-age <TIME>
```

## LinkType

```
SW(config-if)#spanning-tree link-type {point-to-point|shared}
```

# MSTP
---

## Základní konfigurace

```
SW(config)#spanning-tree mode mst     \\ Zapnutí MST
```

```
SW(config)#spanning-tree mst configuration     \\ Přepnutí do MST konfigurace
SW(config-mst)#abort     \\ Odejití bez uložení konfigurace
SW(config-mst)#exit     \\ Odejití s uložením konfigurace
```

```
SW(config-mst)#instance <ČÍSLO> vlan <VLAN_RANGE>     \\ Vytvoření instance
```

```
SW(config-mst)#name <NAME>     \\ Jméno regionu
```

```
SW(config-mst)#revision <ČÍSLO>     \\ Číslo revize nastavení
```

### Nastavení priority

#### Manuálně

```
SW(config)#spanning-tree mst <INSTANCE> priority <PRIORITY>
```

#### Makro

```
SW(config)#spanning-tree mst <INSTANCE> root {primary|secondary}     \\ Funguje stejně jako STP
```

#### Port

```
SW(config-if)#spanning-tree mst  <INSTANCE> port-priority <PRIORITY>    
```

```
SW(config-if)#spanning-tree mst  <INSTANCE> cost <PRIORITY>    
```

## Timers

```
SW(config)#spanning-tree mst forward-time <ČÍSLO>
```

```
SW(config)#spanning-tree mst hello-time <ČÍSLO>
```

```
SW(config)#spanning-tree mst max-age <ČÍSLO>
```

```
SW(config)#spanning-tree mst max-hops <ČÍSLO>
```

## Simulation

```
SW(config)#spanning-tree mst simulate pvst global     \\ Zapnutí/Vypnutí simulace PVSTP na boudary portech
```

```
SW(config-ig)#spanning-tree mst simulate pvst disable     \\ Vypnutí simulace PVSTP na portu
```

```
SW(config-ig)#spanning-tree mst simulate pre-standart     \\ Používání původní cisco MST BPDUs
```