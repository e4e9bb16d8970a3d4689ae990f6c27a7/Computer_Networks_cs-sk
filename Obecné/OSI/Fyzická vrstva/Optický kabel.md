# Optický kabel

## Druhy

-  ** SMF - Single Mode Fiber**
	-   Délky až 100 km
	-   Vlákna 9/125 µm
	-   Přenos Laserem na vlnové délce 1310 nm a 1550 nm
	-   Pouze jeden paprsek (vid)
	-   Bandwith až 10tky Tbps
	-   Druhy
		-   ***OS1***
			-   Útlum 1 dB/km
			-   Délka 2 km
			-   Levnější
		-   ***OS2***
			-   Útlum 0,4 dB/km
			-   Délka 10 km
			-   Dražší
-   **MMF - Multi Mode Fiber**
	-   Délky v jednotkách km
	-   Vlákna 62,5/125 nebo 50/125 µm
	-   Přenos diodou na 850 nm a 1300 nm
	-   Několik paprsků (vidů)
	-   Bandwith v řádech 10tek Gbps
	-   Druhy
		-   ***OM3***
				-   Velikost jádra je 50 µm
				-   Vzdálenost 300 m
				-   Bandwith 10 Gbps
				-   Vlnová délka je 850 nm
		-   ***OM4***
				-   Velikost jádra je 50 µm
				-   Vzdálenost 550 m
				-   Bandwith 10 Gbps
				-   Vlnová délka je 850 nm
				-   Je schopen 100 Gbps na 150 metrech
	
## Konektory
	
-   **ST**
	-   Standart
	-   Historicky nejstarší
	-   2.5 mm ferule
-   **SC**
	-   Standart Connector
	-   Druhý nejrozšířenější
	-   2.5 mm ferule
-   **LC**
	-   Lucent Connector nebo Little Connector
	-   Nejrozšířenější
	-   1,25 mm ferule
![[fiberopticconnector.jpg]]

## Spoje

-  **Flat**
	-   Rovný spoj
	-   \-30 dB
-   **PC**
	-   Physical Contact
	-   \-35 dB
	-   Nejčastěji u OM1 - 4
	-   ![[physical-contact.jpg]]
-   **UPC**
	-   Ultra Physical Contact
	-   \-55 dB
	-   Rychle degraduje při zapojování
	-   ![[ultra-physical-contact.jpg]]
-   **APC**
	-   Angled Physical Contact
	-   \-65 dB
	-   Nejlepší vlastnosti odrazu
	-   Největší počet připojení
	- ![[angled-physical-contact.jpg]]



## WDM

*Wavelenght-division Multiplexing*, se používá pro zvýšení přenosové kapacity média.
Máme 2 metody pro směšování vlnových délek, liší se realizací a hlavně okny ve vlnových délkách.

### DWDM - Dense Wavelenght-division Multiplexing
-   20 nm okno
-   Spektrum od 1271 nm po 1611 nm
-   Limitován na délku asi 80 km, nelze ho regenerovat
-   Podporuje 18 kanálů, 8 pro delší vzdálenosti
-   Levnější

### CWDM - Coarse Wavelenght-division Multiplexing
-   0,8/0,4 nm okno
-   Spektrum od 1525 po 1565 (C Band) a 1570 po 1610 (L Band)
-   V podstatě neomezená délka, vzhledem k tomu, že lze regenerovat signál
-   Podporuje až 80 kanálů
-   U 1Gb Ethernetu je asi 3x dražší
![[dwdm-and-cwdm-wavelengths.jpg]]