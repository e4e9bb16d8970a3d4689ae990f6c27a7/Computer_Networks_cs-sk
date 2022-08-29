# Generic Routing Encapsulation (GRE)
---

GRE tunelling poskytuje podporu pro celou řadu síťových protokolů tak, že je enkapsuluje a forwarduje přes IP síť.
Původně byl vyvynut pro poskytnutí konektivity pro, v IP sítích, nesměrovatelné protokoly, jako *Internetwork Packet Exchange (IPX)*, dnes je využíván jako overlay pro IPv4 a IPv6.

GRE mají spoustu využití:
- Tunelování provozu přes Firewall/ACL
- Propojení nespojitých sítí
- Oprava špatně navržené sítě
- VPN
- IPv6 -> IPv4 tunely

## Enkapsulace

Při enkapsulaci GRE přidává novou IP hlavičku, dle které se paket forwarduje a GRE hlavičku:

```rackdiag
rackdiag {
  4U;
  1: Data;
  2: Původní IP hlavička;
  3: GRE hlavička;
  4: GRE IP hlavička;
}
```

```packetdiag
packetdiag {
  colwidth = 32;
  node_height = 52;

  0-0: C;
  1-1: R;
  2-2: K;
  3-3: S;
  4-4: s;
  5-8: Recurl;
  9-14:Flags;
  15-18: Ver;
  19-31: Protocol Type;
  32-47: Checksum (Optional);
  48-63: Offset (Optional);
  64-95: Key (Optional);
  96-127: Sequence Number (Optional);
  128-159: Routing (Optional);
}
```
Při příchodu na druhou stranu spoje dojde k odstranění přidané IP a GRE hlavičky.
GRE používá IP protokol 47.

## MTU

GRE přidává minimálně `24B` k hlavičce, a k tomu je potřeba uzpůsobit i MTU na interfacu, samotná hlavička se ale liší na základě konfigurace:

|Tunnel Type|Header Size|
|:-:|:-:|
|GRE|`24B`|
|DES/3DES IPsec (transport mode)|`18-25B`|
|DES/3DES IPsec (tunnel mode)|`38-45B`|
|GRE + DES/3DES|`42-49B`|
|GRE + AES + SHA-1|`62-77B`|

## Možné problémy
---

### Recursive Routing

V případě, že jako cestu do internetu, nebo sítě, přes kterou vytváříme spojení, máme defaultní cestu, nebo jinou cestu, která odpovídá podmínkám níže vyplývajícím, spustíme routing protokol přes tunnel interface a zároveň zahrneme síť k providerovy, tak se nám z druhé strany tunelu vrátí cesta na `tunnel destination`, která odkazuje na tunnel interface.
To samozřejmně není možné a protokoly, například OSPF, toto detekují jako *Recursive Routing*.

### Outbound Interface Selection

Pokud máme, z důvodu dostupnosti, například 2 ISP, oba nám skrz DHCP dávají IP a DG, a každého používáme na jiný DMVPN tunel (jejich interfacy), poté je tedy problém s tím, že router nemusí poslat správný paket ze správného interfacu a spojení nebude fungovat.

### Řešení

#### Front Door Virtual Routing and Forwarding

Výše popsané problémy řeší rozdělení tunelů do VRF instancí, protože VRF si udržují vlastní RIB, pokud vytvoříme speciální VRF, do které přidáme pouze tunnel interface spolu s tunnel source interfacem a nastavíme této instanci defaultní cestu, nebudeme se muset strachovat o problémy s vychozím interfacem či rekurzivním routingem.
VRF instance takto spojeny s tunelem a WAN interfacy se nazývají *Front Door Virtual Routing and Forwarding* (*FVRF*).

## Konfigurace GRE

```
R(config)#interface tunnel <NUMBER>     \\ Vytvoření Tunnel interfacu
```

```
R(config-if)#tunnel source <IP Adress | Interface>     \\ Určení zdroje pro tunel, určení interfacu, na kterém se bude provádět enkapsulace a de-enkapsulace
```

```
R(config-if)#tunnel destination <IP Adress>     \\ Určení druhé strany, na kterou a ze které se posílají GRE pakety
```

```
R(config-if)#ip address <IP> <MASK>     \\ Přiřazení IP adresy interfacu
```

```
R(config-if)#ip mtu <MTU>     \\ Nastavení maximálního MTU
```

### Ostatní

```
R(config-if)#bandwith <BW>     \\ Tunelové interfacy nemají odkud vzít defaultní hodnotu, takže se musí nastavit pro routing protokoly a QoS
```

```
R(config-tun)#keepalive <s>     \\ Určení keepalive timeru pro ověření funkčnosti
```