[[VLAN]] [[Routing]]
# Inter-VLAN Routing

## Router on a Stick (ROAS)

Používáno pro routing mezi *VLAN*, v případě, pouze *L2* switchů, je možno tento routing delegovat na router.

```
R1(config-if)#interface g0/0.x     \\ Přepnutí do sub-interface x 
```

```
R1(config-subif)#encapsulation dot1q <VLAN ID>     \\ Nastavení encapsulace a číslo VLANy
```

```
R1(config-subif)#ip address <IP> <Mask>     \\ Přiřazení IP adresy, default gateway
```

Interface musí být zapnutý.

## SVI

Vytváří virtuální interface pro *VLAN*.
Interfacy jednotlivých *VLAN*, které jsou na jednom switchi mezi sebou komunikují bez zásahu, pokud se *SVI* pro *VLAN* nenachází na Switchi, je nutné použít *Routed port*.

```
SW1(config)#interface vlan <VLAN ID>     \\ Přepne se na interface pro VLAN <ID> (SVI), na Cisco Nexus je nutno zapnout (feature interface-vlan)
```

```
SW1(config-if)#ip address <IP> <Mask>     \\ Přiřadí tomuto interfacu IP adresu, default gateway pro koncové zařízení
```

```
SW1(config-if)#no shutdown     \\ Defaultně je vypnutý, je třeba ho zapnout
```


### Routed port

Používá se na *L3* switchi, je rychlejší a není potřeba aktivní prvek navíc.

```
SW1(config)#ip routing     \\ Přepne do L3 módu
```

```
SW1(config)#interface <IF>     \\ Přepne na interface
```

```
SW1(config-if)#no switchport     \\ Vypne switch na tomto interfacu a nastaví L3
```

```
SW1(config-if)#ip address <IP> <Mask>     \\ Přiřadí IP adresu
```

Tento routovaný port se používá pro další směrování ze sítě, například na další router.

## pVLAN SVI

[[Private VLAN]]

Pro případ routování mezi pVLAN je možné udělat SVI na Primary VLAN.

```
SW1(config)#interface vlan <PrVLAN>     \\ Přepnutí na SVI Primary VLAN
SW1(config-if)#ip address <IP> <Mask>     \\ Normální přiřazení sítě, ta je stejná i pro Secondary VLAN
SW1(config-if)#private-vlan mapping <SeVLAN>    \\ Přiřazení Secondary VLAN, které mohou komunikovat
```