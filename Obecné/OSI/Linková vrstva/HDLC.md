# HDLC

   

## Popis

Jeden z nejjednodušších L2 protokolů, původně pochází z SDLC (IBM verze).

Jeho hlavním záměrem je kontrola chyb a správa doručování na L2.

## DLC vs cHDLC

cHDLC je Cisco verze HDLC, kterou ovšem podporuje většina výrobců, původní verze podporovala pouze jeden vyšší protokol, například pouze IP, Cisco přidalo Control pole, které identifikuje vyšší protokol.

## Rámec

-   **Flag**
	-   Identifikuje začátek framu
		-   `01111110`
		-   Posílá se i přesto, že nejsou posílána spojem žádná data, synchronizují se tak hodiny
-   **Address**
	-   1 Byte
	-   2 typy adres
		-   Multicast (`8F`)
		-   Unicast (`0F`)
-   **Control**
	-   Určuje typ rámce
		-   ***0 - Information frame (I-Frame)***
			-   Obsahuje informace vyšší vrstvy
		-   ***01 - Supervisory frame (S-Frame)***
			-   Používají se například pro Flow nebo Error Control
		-   ***11 - Unnumbered frame (U-Frame)***
			-   Konfigurace linky
-   **Protocol - cHDLC**
	-   2 Byte
	-   Určuje protokol vyšší vrstvy
-   **FCS**
	-   CRC kód
-   **Flag**
	-   Identifikuje konec framu
	-   `01111110`
![[hdlc.gif]]