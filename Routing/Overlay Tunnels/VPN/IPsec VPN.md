# [[IPsec]] VPNs
---

|**Funkce a výhody**|**Site-to-Site IPsec VPN**|**Cisco DMVPN**|**Cisco GET-VPN**|**FlexVPN**|**Remote Access VPN**|
|:-:|:-:|:-:|:-:|:-:|:-:|
|**Interoperability**|Multivendor|Cisco-only|Cisco-only|Cisco-only|Cisco-only|
|**Key exchange**|IKEv1 & IKEv2|IKEv1 & IKEv2|IKEv1 & IKEv2|IKEv2|TLS/DTLS + IKEv2|
|**Scale**|P-t-P|Tisíce (Hub-and-Spoke), Stovky (Partially-meshed)|Tisíce|Tisíce|Tisíce|
|**Topology**|Hub-and-Spoke, Mesh|Hub-and-Spoke, On-demand Spoke-to-Spoke partial mesh, Spoke-to-Spoke (spojení jsou automaticky ukončovány bez provozu)|Hub-and-Spoke, Any-to-Any|Hub-and-Spoke, Any-to-Any, Remote Access|Remote Access|
|**Routing**|Nepodporováno|Podporováno|Podporováno|Podporováno|Nepodporováno|
|**QoS**|Podporováno|Podporováno|Podporováno|Nativně podporováno|Podporováno|
|**Multicast**|Nepodporováno|Tunneled|Nativně podporováno přes MPLS a privátní IP sítě|Tunneled|Nepodporováno|
|N**on-IP** protokoly|Nepodporováno|Nepodporováno|Nepodporováno|Nepodporováno|Nepodporováno|
|**Privátní IP adresy**|Podporováno|Podporováno|Podporováno při použití GRE nebo DMVPN|Podporováno|Podporováno|
|**High availability**|Stateless failover|Routing|Routing|Routing IKEv2-based, Server Clustering|Nepodporováno|
|**Enkapsulace**|Tunneled IPsec|Tunneled IPsec|Tunnel-less IPsec|Tunneled IPsec|Tunneled IPsec/TLS|
|**Transport network**|Jakákoliv|Jakákoliv|Private WAN/MPLS|Jakákoliv|Jakákoliv|

## [[#Site-to-Site IPsec Konfigurace|Site-to-Site (LAN-to-LAN) IPsec VPN]]

Jedná se o jednoduché P-t-P spojení pro propojení 2 sítí, dobré pro propojení nespojitých sítí, ale poměrně obtížné na správu při více spojeních.
![How can I help you?](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fwww.makeitsimple.ie%2Fimages%2Fsite-2-site.png&f=1&nofb=1)
## [[DMVPN|Cisco Dynamic Multipoint VPN (DMVPN)]]

Zjednodušuje konfiguraci pro Hub-and-Spoke a Spoke-to-Spoke sítě, zejména, pokud je jich více.
Používá *Multipoint GRE*, *IPsec* a *Next Hop Resolution Protocol* ([[NHRP]]).

![The Network Times: DMVPN Part I. Basic Operation and ...](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2F2.bp.blogspot.com%2F-ddp0jHLYgWw%2FWpqGZataLcI%2FAAAAAAAAATM%2F6DXTsqngGn4_H9M3NnFSpBeNWEI9E_DPgCLcBGAs%2Fs1600%2FFigure-10.JPG&f=1&nofb=1)
## Cisco Group Encrypted Transport VPN (GET VPN)

Vyvynutá pro vytváření *Any-to-Any Tunnel-less VPNs* s použitím originální IP hlavičky přes MPLS síť.

## Cisco Flex VPN

Jedná se o Cisco implementaci IKEv2 standardu s podporou pro Site-to-Site, Remote Access, Hub-and-Spoke a Spoke-to-Spoke direct (partial mesh) topologie.

## Remote Access VPN

Umožňuje připojení vzdálených uživatelů do firemní sítě.
Podporováno na IOS v rámci Flex VPN (IKEv2), ASA a FirePower.

## Site-to-Site IPsec Konfigurace
---

### Site-to-Site GRE over IPsec

Tento způsob zavádí IPsec šifrování do GRE tunelu.

Existují 2 způsoby, jak nakonfigurovat GRE IPsec:
- Pomocí *crypto maps*
- Pomocí *Tunnel IPsec profiles*

Dnes jsou upřednostňovány IPsec profily, protože nahrazují řadu omezení crypto map:
- Crypto mapy nativně nepodporují MPLS
- Jsou poměrně složité na konfiguraci
- Crypto ACL jsou běžně miskonfigurovány
	- Navíc mohou spotřebovávat zbytečné místo v TCAM

#### Crypto maps

Nejprve je nutno vytvořit GRE spojení.

```
R(config)#ip access-list extended <NAME>     \\ Vytvoření ACL pro určení provozu určeného pro IPsec
R(config-acl)#permit gre host <tun-src IP> host <tun-dst IP>     \\ Povolení peerů tunelu
```

```
R(config)#crypto isakmp policy <priority>     \\ Konfigurace ISAKMP pro IKEv1 SA navázání
R(config-isakmp)#encryption <aes | 3des | des> {key_size}     \\ Určení šifrovacího algoritmu
R(config-isakmp)#hash <md5 | sha | sha256/384/512>     \\ Určení hashovacího algoritmu
R(config-isakmp)#authentication <psk | enc| sig>     \\ Určení způsobu autentizace
R(config-isakmp)#group <group>     \\ Určení DH groupy
```

> *priority* identifikuje ISAKMP politiky, nižší má vyšší priority

```
R(config)#crypto isakamp key <passwd> <peer_IP> <peer_MASK>     \\ Určení PSK (pokud vybrán)
```

```
R(config)#crypto ipsec transform-set <NAME> <TS_AUTH> <TS_ENC>     \\ Určení TS
R(cfg-crypto-trans)#mode <transport | tunnel>     \\ Určení funkce 
```

```
R(config)#crypto map <NAME> <SEQ_NUM> ipsec-isakmp     \\ Vytvoření crypto mapy
R(config-crypto-map)#match address <ACL>     \\ Povolení provozu dle ACL
R(config-crypto-map)#set peer <peer_IP>     \\ Určení druhé strany tunelu
R(config-crypto-map)#set transform-set <TS_NAME> [TS_NAME_-6]     \\ Propojení s TS
```

```
R(config)#interface <WAN>     \\ Přepnutí se na tunnel source interface
R(config-if)#crypto map <NAME>     \\ Aplokování konfigurace
```

#### IPsec profiles

```
R(config)#crypto isakmp policy <priority>     \\ Konfigurace ISAKMP pro IKEv1 SA navázání
R(config-isakmp)#encryption <aes | 3des | des> {key_size}     \\ Určení šifrovacího algoritmu
R(config-isakmp)#hash <md5 | sha | sha256/384/512>     \\ Určení hashovacího algoritmu
R(config-isakmp)#authentication <psk | enc| sig>     \\ Určení způsobu autentizace
R(config-isakmp)#group <group>     \\ Určení DH groupy
```

```
R(config)#crypto isakamp key <passwd> <peer_IP> <peer_MASK>     \\ Určení PSK (pokud vybrán)
```

```
R(config)#crypto ipsec transform-set <NAME> <TS_AUTH> <TS_ENC>     \\ Určení TS
R(cfg-crypto-trans)#mode <transport | tunnel>     \\ Určení funkce 
```

```
R(config)#crypto ipsec profile <NAME>     \\ Vytvoření profilu
R(ipsec-profile)#set transform-set <TS_NAME>     \\ Spojení s TS
```

```
R(config)interface tunnel <ID>     \\ Přepnutí se na tunnel interface
R(config-if)#
```

### Site-to-Site VTI over IPsec
---

Zde je konfigurace naprosto shodná, jako v případě s GRE, pouze se přidává jeden příkaz:

```
R(config-if)tunnel mode ipsec <ipv4 | ipv6>
```

V případě, že chceme zvrátit změnu a nastavit tunel na GRE, pak nastavíme:

```
R(config-if)tunnel mode gre <ipv4 | ipv6>
```

Mezi výhody VTI patří:
- Nevyžaduje GRE hlavičky
- Je stále *Up*, pokud je navázán IPsec Phase 2 tunel
- VPN musí být v *Tunnel* módu

Mezi nevýhody:
- Podporuje pouze IP protokoly