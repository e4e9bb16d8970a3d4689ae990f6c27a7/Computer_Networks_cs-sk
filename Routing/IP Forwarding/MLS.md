# Multilayer Switching (MLS)
---

*MLS* je termín označující funkci switche, který je normálně pouze L2, která využívá i vyšší vrstvy ISO/OSI, nejčastěji L3.

V případě L3 switchingu se jedná o [[VLAN]] interfacy (SVI), které poskytují L3 interface přístupný po [[Trunk|Truncích]] nebo portech v [[Switching/VLAN/Teorie/Terminologie#Access|Access]] módu.

## SVI

Z pohledu MLS logiky SVI propojuje interní router v MLS s konkrétní VLANou, což je L2 specifikace.
MLS posílá nebo přímá data v konkrétní VLANě přes SVI a jsou tak v MSL Routing Table označené jako egress interface.
Při forwardování L3 provozu MLS funguje klasickým [[CEF]] procesem s tím rozdíle, že jako poslední informaci musí z CAM tabulky na základě next-hop [[MAC]] adresy zjistit egress interface.
Z toho důvodu se veškerá nastavení týkající se L3, například [[RIP]] passive interfaces, [[DHCP Konfigurace#Relay|DHCP Relays]] a podobně musí nastavovat na SVI, ne na fyzické interfacy.

Vzhledem k tomu, že se jedná o normální L3 interface, má i stejné stavy, ale vzhledem k tomu, že s ním MLS pracuje trochu jinak, problémy mohou mít trochu jiný původ:

- `Administratively down, line protocol down`
	- SVI interface je *shutdown*
- `Down, line protocol down`
	- Neexistující nebo nefunkční VLAN
	- VLAN ve stavu *suspend* nebo *shutdown*
- `Up, line protocol down`
	- Není k VLANě přiřazen FUNKČNÍ L2 interface
	- Může být i zablokovaný pomocí [[VTP#VTP Pruning|VTP Prunningu]] nebo [[MSTP#Další#Možná nefunkčnost komunikace|MSTP]]
- `Up, line protocol up`
	- Vše funguje jak má

## Routed Port

U Cisco prvků se velmi jednoduché udělat z L2 portu L3 (`no switchport`).
Uvnitř switche přiřadí takto nastavenému portu MLS speciální VLANu, která nepůjde nikdy jindy použít a je specifická pouze pro tento port a deaktivuje *L2 Control Plane Protocols* ([[STP]], MAC Address learning...).
Cisco tento proces dělá automaticky pomocí jednoho příkazu, ale u jiných výrobců je nutné tuto konfiguraci udělat manuálně.

Manuálně to lze nastavit i na Cisco zařízeních:

```
SW(config)#vlan <VLAN_ID>
SW(config)#no spanning-tree vlan <VLAN_ID>
SW(config)#no mac address-table learning vlan <VLAN_ID>
SW(config-if)#switchport mode access
SW(config-if)#switchport access vlan <VLAN_ID>
SW(config-if)#switchport nonegotiate
SW(config-if)#no vtp
SW(config)#interface vlan <VLAN_ID>
SW(config-if)#ip address <IP> <MASK>
```

Takové VLANy se nazývají *Internal ussage VLANs*.
A jsou postupně brané z rozšířeného rozsahu v zevstupné podobě (ascending), tedy od 1006 výše.
Toto nastavení lze změnit pomocí příkazu:
```
SW(config)#vlan internal allocation policy <ascending | descending>
```

```
SW#show vlan internas usage     \\ Zobrazení informací o těchto VLANách
```

Tyto VLANy nejsou uložené ve `VLAN.dat` a nesdílejí se pomocí [[VTP]], pokud například používáme VLAN1006
a její nastavení ním příjde pomocí [[VTP]], switch ho jednodušše nepříjme.
Stejně tak, pokud ji chceme nastavit manuálně, switch napíše chybovou hlášku:
```
VLAN(s) not avaible in Port Manager.
```
