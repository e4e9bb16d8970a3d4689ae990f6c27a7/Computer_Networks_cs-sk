[[Enterprise Network Design]]
# VLAN - Complex

## Zadání

### Rozsah

- 800 aktivních zařízení + 20% rezervy
- Nastavení adresních rozsahů pro *IPv4* a *IPv6*

### Logika sítě

- Rozdělení do *VLAN*
  - Prodejci - 75 + 20%
  - Technici - 150 + 20%
  - Designeři - 25 + 20%
  - HR - 200 + 20%
  - Administrátoři - 50 + 20%

- Rozdělení do *pVLAN*
  - Anonymní zákazníci - 100 + 20%
  - Bežní zákazníci - 200 + 20%

### Požadavky na bezpečnost

#### Možnost komunikace

- Prodejci --> Běžní a Anonymní zákazníci
- Technici --> Bežní zákazníci, Designeři
- Administrátoři --> *Everyone*
- HR --> Prodejci, Technici, Designeři
- Anonymní uživatelé --> *Internet*
- Běžní zákazníci --> Na žádost je možné vytvářet zkupiny, mezi kterými mohou zákazníci mezi sebou komunikovat

Dále je nutné propojit dvě sídla pomocí *MetroE*.

#### Spolehlivost 

- Redundance na klíčových místech
- Dostatečný *Throughput*
- Automatická zpráva *VLAN*

#### Ostatní

- *SSH*
- *OSPF* na *Core* a *Distribution Layer*
- Bannery
- Zakázání *Tenetu*

## Návrh

### Rozsahy

#### IPv4

##### ***`10.0.0.0/16`***

- Administrátoři - `10.0.0.0/25 (10.0.0.0 - 10.0.0.128)`
- Prodejci - `10.0.0.129/25 (10.0.0.129 - 10.0.0.255)`
- Technici - `10.0.1.0/24 (10.0.1.0 - 10.0.1.255)`
- HR - `10.0.2.0/24 (10.0.2.0 - 10.0.2.255)`
- Designeři - `10.0.3.0/26 (10.0.3.0 - 10.0.3.63)`

##### ***`10.20.0.0/16`***

- Běžní uživatelé - `10.20.10.0/23 (10.0.10.0 - 10.0.11.255)`
- Anonymní uživatelé - `10.20.12.0/23 (10.0.12.0 - 10.0.13.255)`

##### ***`10.10.0.0/16`***

- Síťová zařízení


##### Ostatní

- `172.168.255.0/30`

- *Core Layer* - `1.0.0.0/24`
- *Distribution Layer* - `2.0.0.0/24`
- *Access Layer* - `3.0.0.0/24`
- Ostatní - `4.0.0.0/16`

#### IPv6

##### **`2001:DB8::/48`**

- Administrtoři - `2001:DB8:1::/64`
- Prodejci - `2001:DB8:2::/64`
- Technici - `2001:DB8:3::/64`
- HR - `2001:DB8:4::/64`
- Designeři - `2001:DB8:5::/64`

- Bežní uživatelé - `2001:DB8:10::/64`
- Anonymní uživatelé - `2001:DB8:11::/64`
 
- *Core Layer* - `2001:DB8:A1::/64`
- *Distribution Layer* - `2001:DB8:A2::/64`
- *Access Layer* - `2001:DB8:A3::/64`
  
- `2001:DB8:B::/36`


### VLAN

#### Zaměstnanci

- Prodejci - `VLAN 10`
- Technici - `VLAN 20` 
- Designeři - `VLAN 50`
- HR - `VLAN 60`
- Administrátoři - `VLAN 5 - NATIVE`


#### Zákazníci

- Anonymní zákazníci - `VLAN 60 - 70`
- Bežní zákazníci - `VLAN 100 - 200`


## Dokumentace












