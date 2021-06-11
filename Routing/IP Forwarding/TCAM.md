# Ternary CAM
---

## Vlastnosti

  
*Ternary Content Addressable Memory* dovoluje porovnat packet na více než jednom poli.

TCAM je rozšíření CAM pro lepší upper-layer processing jako identifikace L2/3 SRC/DST adresy, protokolu nebo QoS.

Na rozdíl od CAM, která je binární, TCAM umožňuje 3 výsledky, `0`, `1` a `X` pro *do not care*.

Díky tomu se nemusí vstup přesně shodovat a bude správný, protože ne vždy potřebujeme přesný záznam, například maska sítě, nepotřebujeme záznam pro každou síť, pokud stejně máme pouze jednu defaultní cestu.

Záznamy jsou uloženy ve formátu Value, Mask a Result (VMR).

-   Value
	-   Přesně uložená hodnota
-   Mask
	-   Ono X, nebo maska, tato hodnota se nemusí shodovat
-   Result
	-   Ukazuje na akci, která se má provést, pokud dojde ke shodě
	-   Akce jako povolení, shození, přeposlání QoS policeru

![[tcam.gif]]

## SDM Templates 

TCAM mají omezenou velikost, která se při každém startu alokuje službám a pokud dojde místo, musí se provádět Process Switching, což má vážný dopad na rychlost.

Na některých switchích lze nastavit prioritu pro službu, pomocí příkazu

```
SW(config)#sdm prefer [vlan | advanced]
```

Nutno mít na paměti, že IPv6 záznam zabírá 2x více místa, než MAC a IPv4...

![[tcamSize.png]]