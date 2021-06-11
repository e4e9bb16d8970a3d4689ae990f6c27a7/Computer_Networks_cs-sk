# Router IP Traffic Export (RITE)
---
 
Jedná se o funkci téměř shodnou s [[SPAN]] u switchů.
Má za úkol zrcadlit data z jednoho interfacu a přeposlat je na určité zařízení, nejčastěji *Intrusion Prevention System* (IDS), ten je poté má zanalyzovat a provést bezpočnostní vyhodnocení.

## Konfigurace

```
R(config)#ip traffic-export profile <NAME>     \\ Vytvoření profilu
R(config-rite)#mac-address <MAC>     \\ Specifikování IDS MAC
R(config-rite)#interface <INT>     \\ Specifikování výstupního interfacu
R(config-rite)#<bidirectional | incoming | outgoing>     \\ Nastavení typu provozu, který se bude duplikovat, lze použít i poměry a ACL
```

```
R(config)#interface <IF>     \\ Přepnutí na interface, ze kterého chceme zrcadlit data
R(config-if)#ip traffic-export apply <NAME>     \\ Spojení RITE s interfacem
```
