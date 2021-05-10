# DHCPv4
---
## Základní konfigurace

```
R1(config)#service dhcp     \\ Zapnutí služby DHCP
R1(config)#ip dhcp excluded-address <START_IP> <END_IP>     \\ Vyřazení rozsahu z poolu
R1(config)#ip dhcp pool <Jméno>     \\ Nastavení poolu/relace
R1(dhcp-config)#network <IP> <MASK>     \\ Nastavení sítě pro pool
R1(dhcp-config)#default-router {HOSTNAME|ADDRESS}      \\ Nastavení default-gateway/í
R1(dhcp-config)#dns-server {HOSTNAME|ADDRESS}      \\ Nastavení DNS serveru/ů
```
```
R1(dhcp-config)#domain-name <JMÉNO>     \\ Nastavení jména domény
```
```
R1(dhcp-config)#lease {D [H] [M] | infinite}     \\ Nastavení lease timeru
```

## Manuální přiřazení

Každé manuální přiřazení vyžaduje vlastní pool

```
R1(config)#ip dhcp pool <Jméno>     
R1(dhcp-config)#host <IP> <MASK|PREFIX>     \\ Nastavení statické adresy
R1(dhcp-config)#hardware-address <MAC>     \\ Nastravení MAC adresy, na kterou se vztahuje IP adresa
R1(dhcp-config)#client-name <Jméno>     \\ Nastavení jména klienta
```

Manuálně přiřazovat lze i na základě textového souboru.

```
*time* Jan 21 2005 03:52 PM

*version* <VERZE>

!IP address Type Hardware address Lease expiration

10.0.0.4 /24 1 0090.bff6.081e Infinite
10.0.0.5 /28 id 00b7.0813.88f1.66 Infinite
10.0.0.2 /21 1 0090.bff6.081d Infinite

*end*
```

```
R1(config)#ip dhcp pool <Jméno>     
R1(dhcp-config)#origin file <URL>     \\ Nastavení cesty souboru s bindingama
```

## Relay

V případě, že DHCP Request musí jít přes L3, pak musíme nastavit Relay agenta.

```
R1(config)#interface <DefaultGateway_Interface>     \\ Výchozí brána sítě, pro VLAN SVI
R1(config-int)#ip helper-address <NextHop>
```

# DHCPv6
---
## Router

### Zapnutí

```
R1(config)#interface <IF>
R1(config-if)#ipv6 dhcp server {POOL|automatic}     \\ Přiřazení poolu k interfacu
```

### Statefull

```
R1(config-if)#ipv6 nd managed-config-flag     \\ Zapnutí M-Flagu, a tak i IPv6 adres přes DHCP
```

### Stateless

```
R1(config-if)#ipv6 nd other-config-flag     \\ Zapnutí O-Flagu, a tak DHCPv6 pro ostatní nastavení
```

## Server

```
R1(config)#ipv6 dhcp pool <Jméno>     \\ Nastavení poolu/relace
R1(dhcp-config)#dns-server {HOSTNAME|ADDRESS}     \\ Nastavení DNS serveru/ů
```

## Relay

V případě, že DHCP Request musí jít přes L3, pak musíme nastavit Relay agenta.

```
R1(config)#interface <DefaultGateway_Interface>     \\ Výchozí brána sítě, pro VLAN SVI
R1(config-int)#ipv6 dhcp relay destination <NextHop>
```
