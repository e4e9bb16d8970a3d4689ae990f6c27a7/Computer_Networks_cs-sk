# Virtual Routing and Forwarding
---

někdy také označováno jako *VPN Routing and Forwarding*, jedná se o technologii, která vytváří separátní virtuální routery uvnitř virtuálního routeru, velmi podobně, jako [[VLAN|VLAN]] vytvářejí virtuální switche.

Router s aktivním VRF má pro každou instanci separátní:
- IP adresy na interfacech
	- Díky tomu se mohou shodovat
- RIB a FIB
- Instance routing protokolů

Jednotlivé VRF jsou kompletně oddělená jak na **Data Plane**, tak **Control Plane**.
Defaultně, bez jakékoli konfigurace, spadá celý router do *global/default* VRF.

Většina příkazů nabízí možnost VRF, ať už `show`, nebo *VRF-aware* protokolů, mezi ty spadá například:
- [[RIP|RIP]], [[EIGRP|EIGRP]], [[OSPF|OSPF]], [[IS-IS|IS-IS]], [[BGP|BGP]]
- [[PBR|PBR]]
- [[NAT|NAT]]
a spousta dalších.

## VRF Lite
---

Tento pojem označuje základní možnou funkci VFR, tedy bez MPLS, pouze přiřazení různých interfaců více VRF instancím a více instancí routing protokolů.

## Konfigurace
---

### Základní

```
R(config)#vrf definition <NAME>     \\ Vytvoření VRF
R(config-vrf)#address-family <ipv4 | ipv6>     \\ Určení protokolu
```

```
R(config-if)vrf forwarding <NAME>     \\ Přiřazení interfacu VRF instanci
```

V případě nastavení VRF na interface dojde ke smazání nastavené IP adresy, protože ta je spojena s Global VRF.

V případě smazání VRF instance jako takové dojde ke smazání i s ní spojené konfogurace (IP adres, routing protokolů, ...).