# Route Map
---

Jedná se o nástroj Cisco IOSu a příbuzných systémů poskytující programovací **If/Then/Else** logiku.
Nejčasti se používá ve spojení s [[PBR]], [[ACL]], při redistribuci nebo [[EEM]], prostě tam, kde potřebujeme nějakou logiku pro rozřazování informací.

Samotná route-mapa se skládá, podobně jako ACL, z jednotlivých záznamů, přičemž každý z nich je pomyslná **If** podmínka.
V případě splnění podmínky, což určuje do klauzule připojený ACL, se provede daná akce, nejčastěji, pro *permit*, dojde k redistribuci/akci, a pro *deny*, nedojde k redistribuci/akci.

- Každá route-mapa:
	- Má specifické jméno, pod které spadají všechny klauzule/příkazy
	- Končí, stejně jako ACL, s *deny*
		- Pokud není nastavena specifická klauzule s *permit* bez *match* podmínky
- Každá klauzule/příkaz:
	- Má akci *permit/deny*
	- Má sekvenční číslo
- Příkazy jsou prováděny sekvenčně, od nejnižšího po nejvyšší sekvenční číslo
- Po prvním matchi se nepokračuje
- V případě, že je použito více *match* příkazů v jedné klauzuli, funguje mezi nimi **AND**, musí tedy dojít ke splnění všech

Samotné specifikování shodnosti informací je nejčastěji zprostředkováno pomocí ACL, vzhledem to komu, že samotné ACL mají také možnosti *permit* a *deny* s implicitním *deny* na konci každého ACL, pro splnění záznamu v dané route-mapě musí dojít i ke shodě (*permit*) v ACL.

Logika funguje následovně:
- **route-map** - *deny*
	- **ACL** - *permit*
		- Dochází k povolení akce, dle route-mapy
		- Například v případě redistribuce se síť, která takto splňuje podmínky, se vyřadí z redistribuce
	- **ACL** - *deny*
		- Dochází k zablokování akce, dle route-mapy
		- Například v případě redistribuce se na tuto síť nebude vztahovat *deny* route-mapy a nedojde tedy k jejímu vyřazení, nedojde ale ani k určení sítě pro redistribuci, síť zůstane nevyhodnocena a bude pokračovat k dalším pravidlům
- **route-map** - *permit*
	- **ACL** - *permit*
		- Dochází k povolení akce, dle route-mapy
		- Například v případě redistribuce se síť, která takto splňuje podmínky, bude redistribuována
	- **ACL** - *deny*
		- Dochází k zablokování akce, dle route-mapy
		- Například v případě redistribuce se na tuto síť nebude vztahovat *permit* route-mapy a nedojde tedy k jejímu určení pro redistribuci, nedojde ale ani k vyřazení z redistribuce, síť zůstane nevyhodnocena a bude pokračovat k dalším pravidlům

## Konfigurace

Konfigurace je specifická pro akci, kterou chceme udělat, ale obecně:

```
R(config)#route-map <NAME> <NUMBER> <permit|deny>     \\ Vytvoření route-mapy
R(config-route-map)#match <INFO> <ACL>     \\ Nastavení If
```

Takto se jednodušše vytvoří **If** podmínka, se kterou se nadále pracuje v rámci dalších konfigurací.