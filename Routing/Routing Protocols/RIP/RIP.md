# RIP
---
Jedná se o jeden z nejjednodušších [[Routing]] , podporují ho i některé "levnější" zařízení, ovšem v dnešních sítích už nemá moc využití.
Router dostane Paket s cestami do dalších sítí, tu implementuje a ze své aktualizované RIB následně sestaví nový Response paket, to je jeden z důvodů, proč konvergence trvá velmi dlouho.

Čistě teoreticky RIP umožňuje při posílání Response uvést jinou next-hop adresu, než svou, tato možnost je ale čistě teoretická a v IOSu ji nelze ani nakonfigurovat a u RIPng ani není.

- Jedná se o Distance-Vector & IGP
- Metrika je založená na počtu Hopů (1 - 15), 16 je bráno jako nedosažitelné
- Protokol je postaven nad [[UDP]]:520/521
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
		- Po tuto dobu je ještě cesta přeposílána, ale s metrikou 16
		- Cesta v tomto stavu je "uzamčena" a nemůže být znovu přidána v případě updatu, po uplynutí timeru může router opět příjmout nový záznam
		- 180s
		- Tento mechanismus se většinou nevyužívá, v případě, že síť nebo cesta do ní již neexistuje, tak její router ji odešle s metrikou 16, což ji ostatním routerům odstraní z RIB, hold down timer se používá v případě, že se o ní nezmiňovaly žádné předešlé Response zprávy v daném timeru, nejčastěji z důvodu [[ACL]] nebo [[Routing/Routing Protocols/RIP/Konfigurace#Filtrování|Filtrování]]
		- Cesta je přeposílána s metrikou 16, protože umožňuje sousedním routerům odstranit next-hop do dané sítě přes daný router a místo toho příjmou jinou cestu, třeba s vyšší metrikou, protože není jisté, zda cesta do sítě je úplně ztracena, router se tak snaží donutit ostatní, aby našli nějakou možnou další cestu
	- Flush
		- Po uplynutí tohoto času je cesta odstraněna z RIB
		- Začíná hned po získání Response Advertisementu, takže odstranění cesty z RIB trvá 240s
		- 240s
		- Odstraní pouze cestu s vypršeným Invalid timerem!
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
- Defaultně sumarizuje na hranicích classful sítě
	- Například v případě, že posíláme síť `10.0.10.0/16`, tak z interfacu připojeného do sítě `192.168.0.0/24` bude posílat sumarizovanou síť `10.0.0.0/8` a naopak
- Podporuje Plain-text a MD5 autentizaci

## RIPng

- RIP pro [[IPv6]]
- Nastavuje se pro Interface, tím odpadá potřeba příkazů jako `network` a `passive-interface`
- Používá [[UDP]]:521

## Mechanizmy proti vzniku smyček
---

### Counting to Infinity

V případě vzniku smyčky v síti, při přeposílání Response se sítěmi, při každém zabudování do RIB se jejich metrika zvýší o 1, pokud vzikne smyčka po určitém počtu oběhů Response zpráv se metrika zvýší na 16 (Infinity).

Vzhledem k základnímu pravidlu, že cesta s nejnižší metrikou se zapisuje do RIB a Flush timeru by vyřazení takovéto cesty (její přesun z metriky například 3 na 16), trval poměrně dlouho a způsobilo by spoustu škody, *Counting to Infinity* proto přidává jedno pravidlo navíc:

V případě, že ze stejného routeru příjde Response s vyšší metrikou, než naposledy, příjemce tuto cestu okamžitě zabuduje do RIB a přepošle ji dále.

Díky tomu dojde k vyčerpání metriky oné cesty značně rychleji.

### Split Horizon

Router neposílá v Response zprávách kompletní obsah RIB a filtruje cesty, které mají odchozí interface jako next-hop.

V praxi nebude R1 posílat cestu do `10.0.0.0/24` na R2, když mu R2 tuto cestu poslal, ve většině případů by to nebyl problém, protože R2 má tuto síť přímo připojenou a od R1 by přišla s metrikou 2, ale pokud by ta síť na R2 vypadla, pak by si bez Split Horizon R2 vytvořil cestu do `10.0.0.0/24` přes R1, který by zase všechnu komunikaci posílal zpět na R2, tedy vznikla by smyčka.

Funkce je defaultně zapnuta na všech interfacech, krom Frame-Relay a ATM, ale lze ji vypnout pomocí 
`R(config-if)#no ip split-horizon`.

#### Split Horizon with Poisoned Reverse

Router sice posílá v Response zprávách kompletní obsah RIB, ale cesty, které mají odchozí interface jako next-hop, označí s metrikou 16.

### Route poisoning

V případě selhání cesty se stále posílá, ale s metrikou 16, což efektivně znamená, že se data do té sítě přestanou posílat, jinak by se muselo čekat na Flush timer pro efektivní zrušení cesty.

Pokud router dostane od svého souseda síť s metrikou 16, která není připojena přímo k němu, okamžitě ji smaže z RIB. 
I po smazání ji ale dále posílá jako invalid, protože si ji stále nechá uloženou v RIP databázi jako *possibly down*.
Nechá ji uloženou po dobu $Flush-Invalid$ timeru.

### Triggered Update

Posílání Response zpráv krom Update timeru i při změně nastavení.

V dokumentaci jsou někdy označovány jako *flash update*.