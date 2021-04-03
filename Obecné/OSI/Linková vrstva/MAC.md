# MAC

## Funkce

-   **Multiplexing/De-Multiplexing Síťové vrstvy**
	-   ***De-Multiplexing***
		-   Je odpovědný za rozpoznání správného L3 protokolu z MAC hlavičky a předání PDU správnému L3 protokolu
	-   ***Multiplexing***
		-   Při zapouzdřování se stará o správné rozpoznání L3 protokolu a předání ho pro MAC hlavičku
-   **Detekce kolizí, CSMA/CD, CSMA/CA**
-   **Adresace na L2**
-   **802.1Q**
	-   VLAN
	-   QoS
-   **Seskládání framů z pouhých bitů na fyzické vrstvě**
-   **Flow-control**
	-   Ovlivňování objemu posílaných dat na základě propustnosti
-   **Nalézání chyb**

## Hlavička (Ethernet II)

-   Destination MAC Address
	-   Cílová MAC adresa
-   Source MAC Address
	-   Zdrojová MAC adresa
-   Ether Type (2B)
	-   Typ L3 protokolu
-   CRC Checksum
	-   Součet pro Error control

## MAC adresa
-   48-bit dlouhá
-   Prvních 24-bit/3-byte je určeno pro označení výrobce
-   Druhých 24-bit/3-byte je určeno pro označení výrobku
-   Skládá se ze sekvence 12-ti hexadecimálních znaků spojených do 6-ti dvojic
-   **LG/IG**
	-   Jedná se o poslední dva bity prvního bytu - L/G (U/L), I/G
	-   **L/G**
		-   Také U/L
		-   Universal/Local bit
			-   0 - Adresa je přiřazena výrobcem
			-   1 - Adresa je přiřazena nějakou organizací, tato adresa má prioritu
	-   **I/G**
		-   Individual/Group
			-   0 - Adresa je unicastová
			-   1 - Adresa je broadcast nebo multicast
	![[mac.png]]
	