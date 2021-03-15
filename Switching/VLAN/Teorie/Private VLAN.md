[[VLAN]]

# Vlastnosti

## Obecně

Funkce Private VLAN pomáhá šetřit spotřebu VLAN.
Například pokud potřebujeme oddělit jednotlivé zařízení od L2 nefiltrované komunikace, pomocí normálních VLAN bychom museli použít speciální VLAN pro každé zařízení, v případě použití konceptu Isolated VLAN nám stačí 1.
Další výhodou je, že Secondary VLAN sdílí jednu síť, Primary VLAN síť.

## Funkce

- Promiscuous
	- Typicky určeno pro Default Gateway
	- Promiscuous --> Everyone
- Isolated
	- Tradiční L2 port, funguje jako ve vlastní VLAN
	- Isolated --> Promiscuous
- Community
	- Tradiční L2 port, funguje jako ve VLAN, kterou sdíli se svou komunitou
	- Community --> Community & Promiscuous

## VLAN typy

- Primary VLAN
	- Zapouzdřuje celou strukturu pVLAN
	- Obsahuje Promiscuous porty
- Community VLAN
	- Obsahuje Community porty
	- V rámci Primary VLAN může být více komunit
- Isolated VLAN
	- Obsahuje Isolated porty
	- Může být pouze jedna v rámci Primary VLAN

## Podmínky použití

- Pokud používá [[VTP]], tak musí Switch být v Transparent módu
- Jako pVLAN nesmí být nakonfigurován Trunk, Port-channel nebo port v dynamické VLAN
- Nesmí být cílem [[SPAN]], může být zdrojem

## Protected Port (pVLAN Edge)

Zjednodušená obdoba Isolated portu v pVLAN, mezi porty ve stavu Protected neprobíhá L2 komunikace, mezi ostatními porty komunikace funguje.
Nastavuje se v rámci VLAN.

## Port Blocking

Za normálních okolností switch rozešle unknown-unicast na všechny porty, krom příchozího, to může bát bezpečnostní hrozba a Cisco to umožňuje blokovat.

[[Switching/VLAN/Teorie/Konfigurace#pVLAN|Private VLAN]]
[[Switching/VLAN/Teorie/Konfigurace#pVLAN Trunk|Private VLAN Trunk]]
[[Switching/VLAN/Teorie/Konfigurace#Protected Port pVLAN Edge|Protected Port pVLAN Edge]]
[[Switching/VLAN/Teorie/Konfigurace#Port Blocking|Port Blocking]]