# Popis

Defaultně Cisco používá *autonegotiation* pro určení duplex a speed linky.

## Speed

### FLP

*Fast Link Pulses* je metoda, která se využívá pro určení rychlosti linky.
Toto určení funguje i pokud je na jednom konci *autonegotiation* vypnuto.
Lince lze vnutit rychlost pomocí příkazu `SW(config-if)#speed <BW>`, ale to může vést k nefunkčnosti linky.

## Duplex

Detekování duplexu lze pouze pomocí zapnuté autonegotiace, a to na obou koncích, pokud z nějakého důvodu nelze duplex zjistit, použije se defaultní nastavení.

- **Half-Duplex**
	- *10 Mbps*
	- *100 Mbps*
- **Full-Duplex**
	- *1Gbps <*
