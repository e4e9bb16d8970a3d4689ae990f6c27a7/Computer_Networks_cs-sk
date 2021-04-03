# LLC

## Funkce

-   **Multiplexing/De-Multiplexing Síťové vrstvy**
	-   ***De-Multiplexing***
		-   Je odpovědný za rozpoznání správného L3 protokolu z LLC hlavičky a předání PDU správnému L3 protokolu
	-  ***Multiplexing***
		-   Při zapouzdřování se stará o správné rozpoznání L3 protokolu a předání ho pro LLC hlavičku
-   **Služby spojení**
	-   LLC je teoreticky schopno obstarat spojení pouze na základě spojení L2 vrstvy
	-   **Connectionless Unacknowledge**
		-   Spojení bez navázání spojení nebo oznamování přenosu dat, data jsou poslána napřímo
		-   Může být využita funkce *Flow Control*
		-   Je nejpoužívanějším typem spojení, vzhledem k tomu, že kontrolu přenosu dnes obstarávají protokoly na L4
		-   Je používán na téměř všech vysokorychlostních spojích Ethernetových a Optických
	-   **Connectionless Acknowledge**
		-   Spojení bez navázání spojení, ale framy jsou značkovány pro správné řazení na L2 peeru
		-   Používá se tam, kde náklady na trvalé spojení jsou nepřípustné, například z důvodu zpoždění spojení a zároveň potřebujeme spolehlivost dat
		-   Využíváno například u Bluethoot nebo *Wi-Fi*
	-   **Connection Oriented**
		-   Před posíláním dat se musí vyjednat spojení Peeru za pomocí Supervisory Framů
		-   Lze používat i nepoužívat Acknowledgment
	-   **Connection Oriented service without Acknowledgment**
		-   Bez jakéhokoli značkování dat, o tuto funkci se starají vyšší vrstvy
		-   Využíváno [[HDLC]], [[PPP]], LAPB, PAPD
	-   **Connection Oriented service with Acknowledgment**
		-   Spolehlivost je zabezpečována číslováním framů a znovu posíláním ztracených framů
		-   *Flow Control* zde používá sliding window mechanizmus
		-   Je velmi málo používán, protože se používá [[TCP]]
		-   Je používán například u *Microsoft NetBIOS*
	
## Hlavička
	
### Vlastnosti
	
Ve standardizaci [IEEE 802.2 Logical Link Control](https://en.wikipedia.org/wiki/IEEE_802.2) je použita i LLC hlavička.

V praxi se však používá standardizace [[MAC#Hlavička Ethernet II|Ethernet II]], která vychází spíše z původního DIX standardu, ta LLC hlavičku, pokud nemusí, nevyužívá.

Přesto ji lze u některých protokolů, které ji potřebují, nalézt, typicky u [[STP Terminologie#BPDUs|STP BPDU]] nebo [[CDP]].

#### Popis
-   **DSAP**
	-   Destination Service Access Point
	-   Určuje typ protokolu
-   **SSAP**
	-   Source Service Access Point
	-   Určuje typ protokolu, který vytvořil frame na vyšší vrstvě
-   **Control**
	-   Určuje, zda je komunikace Connection nebo Connectionless
-   **SNAP**
	-   Dělí se na dvě podvrstvy
	-   ***OUI***
		-   Organizationally Unique Identifier
		-   Dnes prakticky nepoužíván
		-   Umožňuje odesílateli určit výrobce NIC
	-   ***Type***
		-   Určuje protokol na L3 a napomáhá tak správnému zpracování