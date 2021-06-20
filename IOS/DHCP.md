# DHCPv4
---

*Dynamic Host Configuration Protocol*

Jedná se o protokol pro automatickou konfiguraci zařízení v síti, může konfigurovat například:
- IP adresu
- Masku sítě
- Default Gateway
- DNS
- Doménu
- Vendor Class Identifier
- NetBIOS
- PXE

Je založen na klient-server modelu přičemž běží na UDP portech 67-68.

[Podrobnější informace na](https://www.netmanias.com/en/post/techdocs/5998/dhcp-network-protocol/understanding-the-basic-operations-of-dhcp)

## Zprávy
---

### DHCP Discover

Jedná se o iniciační zprávu, kterou posílá klient. Klient hledá DHCP server v síti, a tak zprávu rozesílá broadcastem.

### DHCP Offer

Jedná se o samotnou zprávu s nastavením pro klienta, vzhledem k tomu, že do této doby klient nemá IP adresu, tak je také broadcastována.
Zpráva obsahuje ID serveru pro identifikaci.

### DHCP Request

Touto zprávou žádá klient konkrétní server o přidělení konfigurace.
Vzhledem k tomu, že v síti může být více DHCP serverů, tak tato zpráva musí být také broadcastována, protože ji musí dostat každý ze serverů, který poslal Offer, to kterému serveru, a tak i nastavení, Request patří se určuje dle MAC adresy uvnitř zprávy.

Před odeslání zprávy klient rozešle ARP Request na přidělenou adresu, aby zjistil, zda již není použitá, pokud není, pošle Request.

#### DHCP Decline

V případě, že na ARP někdo odpověděl, a IP adresa je tak zabraná, klient odešle Decline zprávu a požádá tak o novou adresu.

### DHCP Acknowledgement

Poté, co si server uloží klient ID a přidělenou IP adresu pošle Acknowledgement, čímž potvrdí přiřazení.
I tato zprává je Broadcast.
Po příjmutí této zprávy si klient uloží nastavení, které získal skrze DHCP.

#### DHCP Negative Acknowledgement

Tuto zprávu odešle server v případě, že nemá volnou adresu z poolu, a tak ji nemůže přiřadit.

### DHCP Release

Tuto zprávu odesílá klient serveru a informuje ho tak, že IP adresa, kterou má přiřazenou, je nyní volná a server ji může znovu přiřadit.

### DHCP Inform	

V případě, že klient získal adresu manuálně odešle Inform zprávu o tom, že adresa je zadaná a požádá o další nastavení sítě.

Server odpovídá [[#DHCP Acknowledgement]] zprávou.

## Funkce
---

### Přiřazování IP adresy

1. [[#DHCP Discover]] PC
2. [[#DHCP Offer]] SRV
3. [[#DHCP Request]] PC
4. [[#DHCP Acknowledgement]] SRV

### Obnovení IP adresy

1. [[#DHCP Request]] PC
2. [[#DHCP Acknowledgement]] SRV

### Uvolnění IP adresy

1. [[#DHCP Release]] PC

# DHCPv6
---

Funguje nad vrstvou UDP 546-547.
Typ DHCPv6 závisí na [[IPv6#NDP|NDP]] a jeho [[IPv6#^RA|příznacích]].

## Statefull

Jedná se o DHCP při kterém si klient mimo jiné žádá i o IPv6 adresu.

Po přijímutí [[IPv6#^RA|Router Advertisement]] s těmito flagy:

[[IPv6#^mflag|M]]=1
[[IPv6#^aflag|A]]=0
[[IPv6#^oflag|O]]=0

Klient pošle na IPV6 adresu `FF02::1:2` (DHCPv6) **SOLICIT** zprávu pro objevení všech DHCPv6 serverů v síti.
Server odpoví s **ADVERTISE**, kterou dává na jevo, že je dostupný.
Klient následně pošle **REQUEST**, ve kterém si říká o všechny informace, včetně IPv6 adresy.
Server ukončuje komunikaci posláním informací ve zprávě **REPLY**.

## Stateless

Jedná se o DHCP při kterém si klient vygenreruje IPv6 adresu podle [[IPv6#Generování adres|algoritmu]] samo a zeptá se DHCP na ostatní nastavení.
Po přijímutí [[IPv6#^RA|Router Advertisement]] s těmito flagy:

[[IPv6#^mflag|M]]=0
[[IPv6#^aflag|A]]=1
[[IPv6#^oflag|O]]=1

Klient pošle na IPV6 adresu `FF02::1:2` (DHCPv6) **SOLICIT** zprávu pro objevení všech DHCPv6 serverů v síti.
Server odpoví s **ADVERTISE**, kterou dává na jevo, že je dostupný.
Klient následně pošle **INFORMATION-REQUEST**, ve kterém si říká o všechny informace.
Server ukončuje komunikaci posláním informací ve zprávě **REPLY**.