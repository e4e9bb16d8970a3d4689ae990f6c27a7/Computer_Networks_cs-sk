[[VLAN]] [[Switching/VLAN/Teorie/Terminologie]]
# Popis
---
Pro připojení jiného aktivního prvku, komunikace je tagována a přenáší se vybrané VLANy.
Dnes se využívá i pro připojení serverů, který potřebuje komunikovat do více VLAN.
Pokud bychom zapojili switch do access módu, přenášela by se pouze VLANa nastavena na onen port.

## Metody encapsulace

- 802.1q
	- Standardizovaná metoda
	- Do hlavičky přidá 4B a přepočítá FCS
	- Používá 802.1p[^1] pro QoS
- Cisco ISL
	- Pouze pro Cisco Switche
	- Vezme původní frame a necapsuluje ho do nového framu, přidá 30B
	- Nepodporuje Native VLAN
	- Podporuje i jiné, než IP sítě
	- Podporuje prioritizaci
	- Pokud může, [[DTP]] zvolí ISL

![[Encapsulace.png]]

# Private VLAN Trunk
---
[[Private VLAN]]
![[pVLAN Trunk.png]]
 ## Isolated pVLAN Trunk
 
-   Přepisuje VLAN tag primární VLANy a nahrazuje ho tagem Isolated Secondary VLANy
-   S tagem nijak nemanipuluje, pokud pochází z jiné VLANy, nebo je příchozí
-   Umožňuje rozšířit Isolated VLAN na switche, které nepodporují funkci Private VLAN
-   Díky tomu mohou například komunikovat isolated porty s promiscuous

## Promiscuous pVLAN Trunk

-   Přepisuje VLAN tag sekundární VLANy a nahrazuje ho tagem primární VLANy
-   Pokud frame pochází z jiné VLANy, nebo je příchozí, nedochází k manipulaci
-   Používá se například, pokud chceme mít přístup k více zařízením, jako na promiscuous portu, nebo při použití ROAS, router nemusí znát všechny sekundární VLANy, aby mohl routovat

# Trunk on Router
---

Při konfiguraci ROAS se nastavují jednotlivé interfacy VLAN na sub-interfacy routeru.

## Native VLAN 

Při použití konceptu Native VLAN lze nastavit pomocí příkazu native sub-interface, nebo fyzický interface, jako příjemce pro Native VLAN, znamenaje, že všechny netagované framy budou zařazeny na tento interface a framy vycházející z tohoto interfacu budou netagované.

V případě nastavení Native VLANy na sub-interface je nutné mu nastavit i IP adresu , pokud není, router myslí, že nastavení native je přímo pro fyzický interface.

Pokud je nastavena Native VLAN přímo na fyzický interface, nedovoluje mu to správně rozpoznat 802.1Q tagy v případě, že frame má priority tag.

