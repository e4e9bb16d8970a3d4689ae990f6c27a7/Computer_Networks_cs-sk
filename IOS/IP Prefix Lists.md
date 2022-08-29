# IP Prefix Lists
---

Jedná se o nástroj podobný [[Route-Map|Route-Map]] a [[ACL]], umožňuje filtrovat informace na základě předem určených hodnot a dle toho vracet `true`/`false` do jeho aplikace, například na řízení redistribucí.

Umožňuje filtrovat na základě 2 hodnot:
- Prefix cesty (Adresa subnetu)
- Délka prefixu (Maska subnetu)

```
R(config)#ip prefix-list <NAME> <NUMBER> <deny|permit> <NETWORK> {ge} {le}
```

Problémové bývají prefix listy při konfiguraci s hodnotami *network/lenght*, *ge*, *le*.
Defaultně totiž prefix list používá do jeho podmínky zapsaný parametr *network/lenght*, například pro `10.0.0.0/8` se prefix list splní pouze, pokud ho porovnáme přímo s cestou `10.0.0.0/8`, například cesta `10.0.0.0/24`, i přesto, že je z dané sítě, se nesplní, protože má jinou délku masky, v tomto ohledu jsou prefix listy velmi implicitní.

Pro porovnání ostatních sítí nám nabízí nástroje *ge* a *le*.
- **ge** 
	- Používá se pro určení minimální délky masky
	- `10.0.0.0/8 ge 9`
		- Všechny sítě, které spadají do `10.0.0.0/8` sítě a mají délku masky `9 - 32`
		- `10.0.0.0/9` - `10.255.255.255/32`
- **le**
	- Používá se pro určení maximální délky masky
	- Pokud není použit s *ge*, za minimální délku se bere délka určená v síti
	- `10.0.0.0/8 le 24`
		- Všechny sítě, které spadají do `10.0.0.0/8` sítě a mají délku masky `8 - 24`
	- V případě spojení s *ge* se za minimální hodnotu bere ona hodnota
		- `10.0.0.0/8 ge 16 le 24`
			- Všechny sítě z `10.0.0.0/8` prefixu s maskou `16 - 24`
			- `10.0.0.0/16 - 10.255.255.255/24`

Specifické případy jsou s `0.0.0.0/0`:
- `0.0.0.0/0`
	- Toto znamená "pokud je ze sítě 0.0.0.0/0 a má délku sítě 0"
	- V praxi je toto *deny all* podmínka
- `0.0.0.0/0 le 32`
	- Toto znamená "pokud je ze sítě 0.0.0.0/0 a má libovolnou délku sítě"
	- V praxi je toto *permit all*
