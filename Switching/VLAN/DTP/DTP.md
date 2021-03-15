# Dynamic Trunking Protocol

Jedná se o protokol, který dokáže automaticky nastavit typ portu na Switchi.
Typ automatického módu, který se nachází na Switchi záleží na Cisco a jejich zařazení Switche do Access nebo Distribution/Core Layer.

## Módy

- Dynamic Auto
	- Čeká na vysmlouvání typu, defaultně je access port
- Dynamic desirable
	- Aktivně vysmlouvává Trunk port 
	- Má vyšší prioritu, než auto

## Frame

- Verze
- [[VTP]] doména 
- Trunk status
	- Aktivní status portu
	- Typ DTP módu
- Trunk type
	- Druh encapsulace
	- Negotiated/Set
- Sender ID

## Vypnutí

Framy se neposílají v případech

- Port je manuálně nastaven do Access módu
- Port je manuálně nastaven do Trunk módu s příkazem `no negotiate`
- Port je routed
- Port má příkaz `no negotiate`
- Port je v [[Terminologie#dot1q-tunnel|dot1q-tunnel]] módu