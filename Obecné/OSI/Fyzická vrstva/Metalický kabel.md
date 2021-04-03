# Metalický kabel

## Typy

-   **UTP**
	-   Normální kroucený kabel
-   **F/UTP**
	-   Celkové stínění fólií s nestíněnými páry
	-   Jednodušší manipulace
-   **S/UTP**
	-   Celkové stínění opletením s nestíněnými páry
-   **SF/UTP**
	-   Celkové stínění fólií i opletením s nestíněnými páry
-  **S/FTP**
	-   Celkové stínění opletením s páry stíněnými fólií
-   **F/FTP**
	-   Celkové stínění fólií a jednotlivých párů
	-   Využívá se u 10GBaseT Ethernetu
-   **U/FTP**
	-   Bez celkového stínění, pouze jednotlivé páry stíněné fólií
	-   Využívá se u 10GBaseT Ethernetu
-   **Solid**
	-   Vodič je jeden solidní kus mědi
	-   Levnější, horší manipulace
	-   Drží tvar
-   **Stranded**
	-   Vodič se skládá z více lanek mědi
	-   Dražší, lepší manipulace
	-   Navrací se zpět do původního tvaru

## Kategorie

-   **Cat1**
	-   Dříve používán pro telefony a modemy
-   **Cat2**
	-   Dříve používán pro telefony a datové sítě do 4 Mbps
-   **Cat3**
	-   Dříve používán pro datové sítě do 10 Mbps
	-   Dnes používán pro telefonní linky
-   **Cat4**
	-   Token Ring, nepoužívá se
	-   16 Mbps
	-   20 MHz
-   **Cat5**
	-   100BaseTX Ethernet
	-   100 MHz
	-   Dříve využíván pro FastEthernet
-   **Cat5e (extended)**
	-   100BaseTX, 1000BaseT Ethernet
	-   100 MHz
	-   Pro 1 Gbps využívá všechny páry, nejpoužívanější
-   **Cat6**
	-   1000BaseTX, 1000BaseT Ethernet
	-   250 MHz
	-   Do 55 metrů lze využít 10 Gbps
-   **Cat6a**
	-   10GBaseT Ethernet a níže
	-   500 MHz
	-   Lze využít 10 Gbps na plnou vzdálenost
-   **Cat7**
	-   10GBaseT Ethernet a níže
	-   600 MHz
	-   Používá speciální konektor
	-   Pouze stíněný
-   **Cat7a**
	-   100GBaseT Ethernet
	-   1000 MHz
	-   Pouze stíněný
-   **Cat8**
	-   40GBaseT Ethernet
	-   2000 MHz
	-   Pouze stíněný

## Zapojení

Protože zařízení odesílají a přijímají na stejných vodičích, je třeba změnit jejich zapojení, aby nedošlo k rušení, A - B mezi zařízeními na stejné úrovni, dnes ale mají síťové prvky funkci *Auto MDIX*.
Nejpoužívanější typ zapojení je *EIA-TIA 568 B*.

### T-568A
![[RJ45-Pinout-T568A.jpg]]
### T-568B
![[RJ45-Pinout-T568B.jpg]]



### Rollover

Jedná se o kabel pro konzolový přístup, který má prohozené vodiče.