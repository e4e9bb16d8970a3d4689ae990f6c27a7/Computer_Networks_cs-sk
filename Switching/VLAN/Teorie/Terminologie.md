[[VLAN]] [[Trunk]]
# Pojmy

## Access

Jedná se o stav portu, ve kterém je port přiřazen pouze jedná VLANě.
Komunikace je na tomto portu netagovaná, protože se nejčastěji jedná o koncový port k PC.
Veškerá příchozí komunikace automaticky spadá pod přiřazenou VLANu.

## Trunk

Pro připojení jiného aktivního prvku, komunikace je tagována a přenáší se vybrané *VLAN*y.
Dnes se využívá i pro připojení serverů, který potřebuje komunikovat do více *VLAN*.
Pokud bychom zapojili switch do *access* módu, přenášela by se pouze *VLAN*a nastavena na onen port.
U *L3* switchů volíme metodu tagování paketů (přiřazení do *VLAN*y).

## Encapsulace

### ISL

*Inter-Switch Link* byl vytvořen v době, kdy neexistoval standard.
Výhodou je, že encapsuluje celý rámec a vloží ho jako obsah jiného, přidá *30B*.
Podporuje prioritizaci a umožňuje i jiné, než *IP* protokoly.

### 802.1q

Také nazýván *trunking protokol*, nebo *dot1q tagging*.
Jedná se o standardizovanou metodu, kterou podporují všechny switche s podporou *VLAN*.
Vezme původní rámec a jeho hlavičku rozšíří o *4B* informace a přepočítá *FCS*. ^9cc932

|16b|3b|1b|12b|
|:-:|:-:|:-:|:-:|
|TPID|T|C|I|
|TPCI|PCP|DEI|VID|

- Tag Protocol Identifier (TPID)
	- Označuje začátek dot1Q tagu, je na stejném místě jako EtherType
	- `0x8100`
- Tag Control Information (TCI)
	- Priority Code Point (PCP)
		- Určuje prioritu pro QoS
	- Drop Eligible Indicator (DEI)
		- Původně CFI pole, ale ve spojení s PCP může označovat framy, které lze zahodit v případě přetížení sítě
	- VLAN Identifier (VID)


## dot1q-tunnel
[[Q-in-Q Tunneling (802.1ad)]]

Umožňuje přenášet naše *VLAN* tagy po síti *ISP*.
Provádí se dvojí tagování, pakety na tunnel portu se doplní o tag *VLAN* providera.


## Native VLAN

Nastavuje se na trunk portu, musí být shodný na obou stranách.
Příchozí provoz, který není tagován se zařazuje do *Native VLAN* a přenos zařazen do *Native VLAN* se netaguje.

[^1]: https://en.wikipedia.org/wiki/IEEE_P802.1p