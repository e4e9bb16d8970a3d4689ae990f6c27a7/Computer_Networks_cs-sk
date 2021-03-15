[[VLAN]]
# VLAN
---

## Základní nastavení

### CLI

Konfigurace se provede až po opuštění prostředí `(config-vlan)`

```
SW1(config)#vlan <Číslo>     \\ Vytvoření VLAN
SW1(config-vlan)#name <Jméno>     \\ Přiřazení jména
SW1(config-vlan)#mtu <Číslo>     \\ Přiřazení MTU pro VLAN
SW1(config-vlan)#shutdown     \\ Vypnutí VLAN
SW1(config)#no vlan <Číslo>     \\ Smazání VLAN
```

### Database

Na starších zařízeních je možné nastavit VLAN pomocí databáze.

```
SW1(config)#vlan database     \\ Přepnutí do VLAN databáze
```

## Mazání

Konfigurace je uložena v running-config a souboru VLAN.dat.

```
SW1(config)#no vlan <Číslo>
SW1#delete vlan.dat     \\ Vymazání souboru VLAN.dat
```

## pVLAN
[[Private VLAN]]
```
SW1(config)#vtp mode transparent     \\ Přepnutí VTP do Transparent, pokud je VTP zavedeno
```
```
SW1(config)#vlan <Číslo>
SW1(config-vlan)#private-vlan <primary|isolated|community>     \\ Určení funkce VLAN
```
```
SW1(config)#vlan <Číslo>     \\ Přepnutí do VLANy, kterou jsme označili jako Primary
SW1(config-vlan)#private-vlan association <Číslo>|<add|remove>     \\ Přiřazení pVLAN do Primary VLANy
```
```
SW1(config)#interface <IF>     \\ Přepnutí na Interface
SW1(config-if)#switchport mode private-vlan host
SW1(config-if)#switchport private-vlan host-association <PrVLAN> <SeVLAN>     \\ Přiřazení Secondary VLAN
```

#### Promiscuous port

```
SW1(config)#interface <IF>     \\ Přepnutí na Interface
SW1(config-if)#switchport mode private-vlan promiscuous
SW1(config-if)#switchport private-vlan mapping <PrVLAN> <SeVLAN>,<SeVLAN>     \\ Přiřazení Primary VLAN
```

### Protected Port (pVLAN Edge)

```
SW1(config-if)#switchport protected     \\ Nastavení portu jako Protected Port
```

### Port Blocking

```
SW1(config-if)#switchport block unicast     \\ Blokuje unkown-unicast
SW1(config-if)#switchport block multicast     \\ Blokuje unknown-multicast
```

[[Inter-VLAN Routing#pVLAN SVI|pVLAN SVI]]

# Trunk
[[Trunk]]

Linka musí splňovat několik předpokladů, aby se stala Trunkem.
1. Musí se jednat o Point-to-Point linku
2. Porty musí mít shodné
	1. Rychlosti
	2. Duplex mode
	3. Metodu encapsulace
	4. Nativní VLAN

### Přepnutí

```
SW(config-if)#switchport trunk encapsulation dot1q    \\ Nastavení metody encapsulace
SW(config-if)#switchport mode trunk    \\ Přepnutí do módu Trunk
```

### Nastavení VLAN

Defaultně jsou povoleny všechny VLANy.

```
SW(config-if)#switchport trunk allowed vlan <VLAN_ID>, <VLAN_ID>, <VLAN_ID>    \\ Přiřazení VLAN 
SW(config-if)#switchport trunk allowed vlan <add|remove|all|none|except> VLAN_ID
```

### Nastavení Native VLAN

```
SW(config-if)#switchport trunk native vlan <VLAN_ID>
```

## ROAS

```
R(config)#interface <IF>.<VLAN_ID>    \\ Přepnutí na sub-interface pro danou VLAN
R(config-subif)#ip address <IP> <Mask>    \\ Přiřazení IP adresy do VLANy
R(config-subif)#encapsulation <dot1q|ISL> <VLAN_ID> {native}    \\ Přiřazení encapsulace, VLANy a Native
```

## pVLAN Trunk
