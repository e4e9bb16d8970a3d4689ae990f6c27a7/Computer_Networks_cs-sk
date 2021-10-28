# Typy bezdrátových sítí
---

## Velikostně
---

### WPAN

*Wireless Personal Area Network* (802.15 Bluetooth)
- Menší ve velikosti, 10m, nižší energie, přenosné
- 2.4 GHz

### WLAN

*Wireless Local Area Network* (802.11)
- Více uživatelů, až 100m, větší spotřeba energie
- 2.4 GHz, 5GHz, 6GHz, 60GHz

### WMAN

*Wireless Metropolean Area Network* (802.16 WiMAX)
- Pro větší pokrytí
- Nižší frekvence
- IoT

### WWAN

*Wireless Wide Area Network*
- Pro největší pokrytí
- GSM, LTE

## Topologicky
---

![[WTopology.png|100%]]

### Ad-Hoc Mode

*Independent Basic Service Set* (IBSS)
- Malé množství zařízení
- Zařízení se připojují přímo každé s každým, bez centrálního bodu
- Problémy s kolizemi

### Infrastructure Mode

Tato topoloige je postavena na AP (*Access Point*), centrálním bodu, ke kterému se připojují ostatní zařízení.
AP poskytuje *Basic Service Set* (BSS) uvnitř arei *Basic Service Area*.

- #### Basic Service Set

	V tomto módu je připojeno AP, které se stará o přenost dat mezi zařízeními, zařízení se již nepřipojují každý ke každému, ale připojí se k AP, které funguje jako Hub.

- #### Distribution System

	V této architektuře je AP připojeno k další síti, typicky Ethernetu, AP je pouze bezdrátové spojení do této infrastruktury.
	AP převádí 802.11 framy na jiný protokol.

- #### Extended Service Set

	V této architektuře existuje více AP, které jsou propojeny pomocí DS.

### Bridging

V případě nemožnosti využití fyzických spojů, bezdrátové technologie umožňují využít AP nabo anténu pro jednoduché sdílení konektivity.

- #### Workgroup Bridging
	V tomto případě se nejčasteji jedná o indoor konektivitu.
	Využívá se již existující AP, ne které se připojí Wireless Bridge, který následně tuto konektivitu může sdílet na drátové spoje, čímž se jednoduše přemostí část budovy, kde nejsou kabely.
- #### P2P/P2MP
	Typicky pro propojení budov nebo při poskytování bezdrátového internetu.
	Výhodné pro propojení více budov organizace, levnější, než pronájem bandwithu.
	
![[ISP_P-t-MP.png]]

### Repeaters

Jedná se o architekturu, kdy typicky jedno AP je připojeno do DS další AP, bez drátového připojení k DS, se bezdrátově připojí na první AP a vytváří vlastní síť.
V praxi je tento způsob velmi neefektivní, protože připojení je na stejném kanálu a AP jednak komunikují mezi sebou a jednak s koncovými zařízeními, tím vzniká silné zarušení, navíc je tato technologie nahrazena Mesh sítěmi.

### Mesh

Tato sít má minimálně jeden, ale častěji alespoň 2 a více prvotních AP, které jsou připojeny drátem k DS.
Mesh AP mezi sebou vytvářejí Mesh topologii, podobně, jako [[#Repeaters]], s tím rozdílem, že nekomunikují na jednom Channelu, ale na více nepřekrývajících se Channelech a sdílejí jedno SSID.
Zároveň například Cisco nabízí *Adaptive Wireless Path Protocol* (AWPP), který umí vypočítat nejlepší cestu Mesh síti k DS.
Mesh sítě jsou obecně velmi rezilientní.