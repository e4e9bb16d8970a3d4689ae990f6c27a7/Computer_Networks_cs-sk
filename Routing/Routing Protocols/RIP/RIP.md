[[Routing]] [[Distance-Vector Routing Protocol]] [[Interior-Gateway Routing Protocol]]
# Vlastnosti

Jedná se o jeden z nejjednodušších [[Routing]] , podporují ho i některé "levnější" zařízení, ovšem v dnešních sítích už nemá moc využití.
Router dostane Paket s cestami do dalších sítí, tu implementuje a ze své aktualizované RIB následně sestaví nový Response paket, to je jeden z důvodů, proč konvergence trvá velmi dlouho.

- Jedná se o [[Distance-Vector Routing Protocol]] [[Interior-Gateway Routing Protocol]]
- Metrika je založená na počtu Hopů (1 - 15), 16 je bráno jako nedosažitelné
- Protokol je postaven nad [[UDP]]:520
- ##### Advertisements
	-  **Request**
		- Dotázání po RIB
	-  **Response**
		- Odpověd s RIB
		- Rozesílá každých 30s nebo při změně topologie
- ##### Timers
	- Update
		- Čas za který se pošle další Response Advertisement
		- 30s
	- Invalid
		- Určuje čas, po který může být cesta v RIB
		- Po uplynutí se cesta označí jako Unreachable (16)
		- 180s
	- Hold Down
		- Začíná po uplynutí Invalid Timeru
		- Po tuto dobu může ještě být cesta obnovena, ale nemůže být změněna
		- 180s
	- Flush
		- Po uplinutí tohoto času je cesta odstraněna z RIB
		- Začíná hned po získání Response Advertisementu, takže odstranění cesty z RIB trvá 240s
		- 240s
	![[RIPTimers.jpg]]

## RIPv1

- Přeposílá pouze třídy adres
- Nepodporuje VLSM ani CIDR
- Neposílá masku sítě
- Advertisements rozesílá na `255.255.255.255` broadcastu

## RIPv2

- Podporuje VLSM a CIDR
- Posílá i masku sítě
- Advertisements rozesílá na `224.0.0.9`
- Defaultně nesumarizuje 
- Podporuje Plain-text a MD5 autentizaci

## RIPng

- RIP pro [[IPv6]]
- Nastavuje se pro Interface, tím odpadá potřeba příkazů jako `network` a `passive-interface`
- Používá [[UDP]]:521
