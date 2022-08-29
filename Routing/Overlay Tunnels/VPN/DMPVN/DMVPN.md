# Dynamic Multipoint VPN (DMVPN)
---

DMVPN technologie poskytuje řadu výhod:
- **Zero-touch provisioning**
	- Huby nepotřebují další konfiguraci při přidání dalších spoků.
	- Spoky mohou pro konfiguraci využívat templaty.
- **Scalable deployment**
	- Minimální permanentní peering na spoke zařízeních umožňuje vytvářet velké sítě a jednoduše je rozšiřovat.
- **Spoke-to-Spoke tunnel**
	- DMVPN poskytuje *full-mesh* konektivitu, která se generuje automaticky, spoke-to-spoke tunely jsou automaticky navazovány v případě potřeby a ukončovány po pominutí potřeby.
	- Nakonfigurovat je potřeba pouze Spoke-to-Hub tunel
- **Flexible network topologies**
	- DMVPN není rigidně spojeno s jednou overlay topologií, lze jednoduše navrhnout *HA* topologii, tak i vysoce centralizovanou.
- **Multiprotocol support**
	- DMVPN podporuje celou řadu protokolů: IPv4, IPv6 i MPLS jako overlay nebo underlay
- **Multicast support**
	- DMVPN podporuje multicast provoz, což navazuje i například na podporu VXLAN.
- **Adaptable connectivity**
	- DMVPN podporuje zařízení za NAT nebo s použitím DHCP.
- **Standardized building blocks**
	- DMVPN používá otevřené standardy pro navazování spojení, jako [[NHRP]], [[GRE]] a [[IPsec]].

Spoke zařízení iniciují trvalé VPN spojení k Hub zařízení, samotný data-plane tok nemusí ale jít přes hub, DMVPN dynamicky vytváří přímé (spoke-to-spoke) tunely, tak, jak jsou potřeba.
To je velice výhodné pro high bandwith provoz nebo VoIP provoz, protože se nespotřebovává konektivita hub zařízení a zároveň se šetří odezva, protože může jít provoz přímější cestou.

DMVPNDMVPN funguje ve 3 fázích, přičemž každá nadcházející je nadstavbou té předešlé.
Všechny 3 fáze potřebují pouze 1 tunnel interface a celá DMVPN síť by si měla přizpůsobit fungování pro tento interface.
DMVPN spoky mohou používat DHCP nebo NAT pro underlay i overlay sítě, protože zjišťování jejich IP adres probíhá pomocí NHRP.

## Phase 1: Spoke-to-Hub
---

Jedná se o první DMVPN implementaci umožňující *Zero-touch deployment* pro spoky.
VPN tunely jsou vytvářeny pouze mezi Spoke-to-Hub a provoz mezi jednotlivými spoky musí tedy plynout přes Hub.

## Phase 2: Spoke-to-Spoke
---

Druhá implementace DMVPN umožňuje vytváření dynamických on-demand VPN spojení mezi spoky a umožňuje tedy Spoke-to-Spoke komunikaci.

Neumožňuje ale sumarizaci (next-hop preservation), což vede k tomu, že nepodporuje komunikace mezi různými DMVPN sítěmi (*Multilevel Hierarchical DMVPN*).

## Phase 3: Hierarchical Tree Spoke-to-Spoke
---

DMVPN Phase 3 pozměňuje NHRP zprávy a interakci s RIB.
V DMVPN Phase 3 hub posílá NHRP redirect zprávů spoku, který inicioval komunikaci. NHRP redirect poskytuje všechny nutné informace pro spoke pro započetí zjišťování adresy cílového spoku a tím možnost navázání Spoke-to-Spoke tunelu.

Dále NHRP instaluje *shortcut* do RIB pro tunel a sítě, které vytvoří, buď změnou next-hop pro existující sítě, nebo přidáním přesnějších záznamů.
Díky tomu DMVPN Phase 3 podporuje sumarizaci sítí na hubech a poskytování optimálních cest mezi spoky. NHRP shortcuts poskytují vytváření hierarchických/stromových topologií s regionálnímy huby, které jsou zodpovědné za správu svých, regionálních, sítí pomocí NHRP s tím, že Spoke-to-Spoke tunely mohou být vytvářeny napříč regiony.

## Porovnání 
---
### DMVPN 1 vs 2&3
![[Pasted image 20220130110033.png]]
### DMVPN 2 vs 3
![[Pasted image 20220130110054.png]]

## Spoke-to-Spoke komunikace
---

Z prvu komunikace prochází Hubem, jako u Phase 1, dokud není vytvořen Spoke-to-Spoke tunel v obou směrech.
Při prvním paketu příchozím na hub, hub odesílá NHRP redirection, spolu s přeposláním paketu do cíle, a začíná proces hledání nejlepší cesty mezi spoky.

```seqdiag
seqdiag {
activation = none;
Spoke-1 -> Hub [label="Data packet"] Spoke-2 Hub -> Spoke-2 [label="Data Packet"];
Spoke-1 <-- Hub [label="NHRP Redirect"]   ;
Spoke-1 --> Hub [label="NHRP Resolution Request"]Hub --> Spoke-2 [label="NHRP Resolution Request"]Hub <- Spoke-2 [label="Data Packet"] ;
Spoke-1 <- Hub [label="Data Packet"]Hub --> Spoke-2 [label="NHRP Redirect"]  ;
Spoke-1 Hub <-- Spoke-2 [label="NHRP Resolution Request"] Spoke-1 <-- Hub [label="NHRP Resolution Request"]Spoke-1 <-- Spoke-2 [label="NHRP Resolution Reply"];
 Spoke-2;
Spoke-1 --> Spoke-2 [label="NHRP Resolution Reply"];
}
```

1. Spoke-1 provede route lookup pro cílovou síť a najde záznam s next-hop adresou hubu, enkapsuluje paket a přepošle ho přes tunel interface
2. Hub příjme paket a provede vlastní route lookup pro cílovou síť. Zjistí, že paket má přeposlat na Spoke-2, skrze NHRP cache zjistí NBMA IP a přepošle paket (s novou hlavičkou) z tunel interfacu
	- Protože Hub má nastaveno `ip nhrp redirect` a paket odešel ze stejného interfacu, jako ze kterého přišel, Hub pošle Spoke-1 *NHRP Redirect*
3. Spoke-1 příjme NHRP Redirect od Hubu a obratem mu pošle *NHRP Resolution Request* pro cílovou síť.
	- Uvnitř NHRP Resolution Requestu posílá svou Protocol a NBMA adresu
4. Spoke-2 příjme paket a provede route lookup pro odpověď, dojde ke stejnému závěru, jako Spoke-1, a odešle odpověď skrz Hub
	- Hub příjme tento paket a, stejně, jako předtím, přepošle ho na Spoke-1 s tím, že na Spoke-2 pošle NHRP Redirect
5. Spoke-2 pošle NHRP Resolution Request
6. Hub přepošle NHRP Resolution Requesty správným zařízením
7. Zařízení si navzájem odpoví s NHRP Resolution Reply
	- Tento již cestuje napřímo
		- Pokud se používá IPsec, nejprve se vytvoří tunel
	- Obsahuje informace druhé strany (informace z NHRP Resolution Requestu), aby bylo jasné, na které odpovídá

Po výměně obou NHRP Resolution Reply je vytvořen Spoke-to-Spoke DMVPN tunel.

## NHRP Routing Table Manipulation
---

NHRP úzce spolupracuje s RIB a v případě, že dojde ke změně (NHRP Redirect), pak umí přepsat RIB, aby další komunikace podléhala tomuto redirectu.
Sítě takto změněné jsou označeny pomocí `%`, sice v RIB obsahují jako next-hop standardní, nakonfigurovanou, adresu, ale pro routing se přidal nový záznam, který lze zobrazit pomocí `show ip route next-hop-override` a je označen jako `[NHO]`.
Zaroveň s overridem se vkládá NHRP `H` síť odkazující na daný Spoke.

## DMVPN Konfigurace
---

### Hub
---
```
R(config)#interface tunnel <ID>     \\ Vytvoření interfacu
R(config-if)#tunnel source <IP|IF>     \\ Určení zdroje pro interface
R(config-if)#ip address <IP> <MASK>     \\ Nastavení IP adresy
R(config-if)#tunnel mode gre multipoint     \\ Nastavení tunelu na mGRE
```

```
R(config-if)#ip nhrp network-id <ID>     \\ Zapnutí NHRP a lokální identifikace DMVPN sítě
```
> Není nutné, aby se NHRP ID shodovalo s číslem interfacu, ale je lepší je mít shodné, zrovna tak není nutné, aby se shodovali napříč DMVPN sítí, ale jedná se o standard.
```
R(config-if)#tunnel-key <KEY>     \\ (optional)
```
> Pokud více tunnel interfaců používá jeden source interface,musí se použít klíče pro identifikaci provozu, musí se shodovat mezí peery.

```
R(config-if)#ip nrp map multicast dynamic     \\ Zapne podpodu pro multicast (optional)
```

```
R(config-if)#bandwith <BW>     \\ Určení BW (optional)
R(config-if)#ip mtu 1400     \\ Snížení MTU, doporučeno (optional)
R(config-if)#ip tcp adjust-mss 1360     \\ Snížení paylodu u TCP TWH, doporučeno (optional)
```

#### Phase 3

```
R(config-if)#ip nhrp redirect     \\ Pro fungování Phase 3, je nutno zapnout NHRP redirects
```

### Spoke
---

#### Phase 1

```
R(config)#interface tunnel <ID>     \\ Vytvoření interfacu
R(config-if)#tunnel source <IP|IF>     \\ Určení zdroje pro interface
R(config-if)#ip address <<IP> <MASK> | DHCP>     \\ Nastavení IP adresy
R(config-if)#tunnel destination <HUB_UNL_IP>     \\ Určení hubu
```
```
R(config-if)#ip nhrp network-id <ID>     \\ Zapnutí NHRP a lokální identifikace DMVPN sítě
```
```
R(config-if)#ip nhrp nhs <NHS_OVL_IP> nbma <NHS_UNL_IP> {multicast}     \\ Určení NHS
```
> V literatuře se běžně můžete setkat s pojmem *NMBA address* (underlay IP).
> *Multicast* zapíná podporu pro mapování multicast adres v NHRP, tato podpora je nutná pro RIP, EIGRP, OSPF a IS-IS.
```
R(config-if)#bandwith <BW>     \\ Určení BW (optional)
R(config-if)#ip mtu 1400     \\ Snížení MTU, doporučeno (optional)
R(config-if)#ip tcp adjust-mss 1360     \\ Snížení paylodu u TCP TWH, doporučeno (optional)
```
```
R(config-if)#tunnel-key <KEY>     \\ (optional)
```

Příkaz `ip nhrp nhs <NHS_OVL_IP> nbma <NHS_UNL_IP> {multicast}` lze nahradit těmito 3 příkazy:
```
R(config-if)#ip nhrp nhs <NHS_OVL_IP>     \\ Vytvoří záznam pro NHS
R(config-if)#ip nhrp map <OVL_IP> <UDL_IP>     \\ Mapuje danou overlay IP adresu na underlay IP tunelu
R(config-if)#ip nhrp map multicast <<UDL_mIP> | dynamic>     \\ Umožňuje mapování broadcast a multicast
```

#### Phase 3

Jediná změna mezi P1 a P3 je, že se tunel přenastaví z P2P na Multipoint, ostatní konfigurace (krom `tunnel destination`) zůstává.

```
R(config-if)#tunnel mode gre multipoint     \\ Nastavení na mGRE
R(config-if)#ip nhrp shortcut     \\ Povolení NHRP redirectů
```

### Kontrola DMVPN Tunnel stavu
---

Hlavním příkazem při kontrole stavu DMVPN spojení je `show dmvpn {detail}`:

```
R11#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        T1 - Route Installed, T2 - Nexthop-override
        C - CTS Capable
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel1, IPv4 NHRP Details
Type:Hub/Spoke, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 145.31.0.2       192.168.100.31    UP 01:16:26     D
     3 145.41.0.2       192.168.100.41    UP 01:16:22     D
                       192.168.100.100  NHRP    never    SC
                       192.168.100.110  NHRP    never    SC
```

Existuje několik stavů spojení:
- **INTF**
	- *Line protocol* DMVPN tunelu je *Down*.
- **IKE**
	- DMVPN je nakonfigurován s IPsec a ještě nedošlo k navázání IKE SA
- **IPsec**
	- DMVPN je nakonfigurován s IPsec, došlo k navázání IKE SA, ale ještě nedošlo k navázání IPsec SA
- **NHRP**
	- DMVPN spoke ještě není úspěšně zaregistrován
- **Up**
	- DMVPN Spoke je zaregistrován na NHS a dostal *ACK* od Hubu

## DMVPN Failure Detection and High Availability
---

[[NHRP]] mapping zůstává v cachi po dobu *holdtime*, který je defaultně roven 7200s (2 hodiny) (`ip nhrp holdtime`), je doporučeno tuto hodnotu změnit na 600s.
Druhotná funkce NHRP registration paketů je, krom potvrzení registrace záznamu, potvrzení konektivity NHS a NCS, požadavky na registraci jsou posílány každých *timeout* sekund (defaultně), pokud není odpovězeno na první paket, tak se další pošle se spožděním 1s, druhý s 2s a třetí s 4s.
Pokud ani na třetí paket (4) nebylo odpovězeno, pak je NHS prohlášeno za *Down*. (v DMVPN jako NHRP)(interface je stále up up)

pokud je NHS down, tak se NHS pokusí u něj stále registrovat, probe state, delay mezi pakety je 1, 2, 4, 8, 16, 32, 64, delay nejde nad 64, pokud se vrátí odpoved, hub je up

Při vytvoření Spoke-to-Spoke tunelu, ten je taky kontrolován, pokud je tunel stále využíván během posledních 2 minut registration timeoutu, pak se záznam obnoví pomocí NHRP requestu, pokud není, tak dojde k jeho vypnutí.
timeout period je 1/3 holdtime (defaultně 2400s) ip nhrp registration timeout

## DMVPN with [[IPsec]] Protection
---

### IKEv2 Keyring

Jedná se o repozitář ukládající PSK.
Vytvářejí se peery jejichž identifikace je na základě IP adres, je tedy možné vytvořit různé PSK pro různé peery.

```
R(config)#crypto ikev2 keyring <NAME>     \\ Vytvoření keyringu
R(config-ikev2-keyring)#peer <NAME>     \\ Vytvoření peeru
R(config-ikev2-keyring-peer)#address <IP> {MASK}     \\ Specifikace peeru (NBMA)
R(config-ikev2-keyring-peer)#pre-shared-key <PSK>     \\ Určení hesla
```

### IKEv2 Profile

Jedná se o nastavení neoddiskutovatelných bezpečnostních parametrů IKE SA.
Následně se spojuje s IPsec profilem.

```
R(config)#crypto ikev2 profile <NAME>     \\ Vytvoření profilu
R(config-ikev2-profile)#match identity remote address <IP>     \\ Určení peeru(ů)
R(config-ikev2-profile)#identity local address <IP>  \\ Určení lokální identity (optional), nutné pro PKI
R(config-ikev2-profile)#match fvrf <NAME>     \\ V případě nastavení s FVRF
R(config-ikev2-profile)#authentication local <psk|rsa>     \\ Určení autentizace pro ostatní
R(config-ikev2-profile)#authentication remote <psk|rsa>     \\ Určení autentizace pro nás
R(config-ikev2-profile)#keyring local <NAME>     \\ spojení s keyringem
```
```
R#show crypto ikev2 profile     \\ Ověření nastavení
```

### IPsec Transform Set

Následně je nutné určit protokoly pro šifrování provozu.

```
R(config)#crypto ipsec transform-set <NAME> <TS_AUTH> <TS_ENC>     \\ Určení TS
R(cfg-crypto-trans)#mode <transport | tunnel>     \\ Určení funkce 
```

### IPsec Profile

```
R(config)#crypto ipsec profile <NAME>     \\ Vytvoření profilu
R(ipsec-profile)#set transform-set <TS_NAME>     \\ Spojení s TS
R(ipsec-profile)#set ikev2-profile <NAME>     \\ Spojení s IKEv2 profilem
```

### Aplikování na tunel

```
R(config-if)#tunnel protection ipsec profile <NAME> {shared}     \\ Spojení s IPsec profilem
```

`shared` se používá pokud máme vícero DMVPN tunelů na jednom transportním interfacu (tunnel source), v takovém případě dochází ke sdílení *Security Association Database* (*SADB*).
V takovém případě je nutné nastavit jiné Tunnel klíče.

### Ochrany
---
#### IPsec Packet Replay Protection

Cisco IPsec implementace obsahuje *anti-replay* mechanismus, který brání proti duplikaci paketů tím, že každému přiřadí sekvenční číslo a zahazuje staré nebo duplikované pakety.
Router udržuje *sequence number windows size* (defaultně 64 paketů), minimální sekvenční číslo je definováno jako nejvyšší číslo (o kterém zařízení ví) - window size.
Paket je příjmut pokud je mezi těmito hranicemi.

Sekvenční číslo je nastaveno před zašifrováním a QoS, může se tedy stát, že některé pakety příjdou out-of-order, protože byli pozměněny na základě QoS, může se tedy stát, že defaultní okno 64 paketů není dostatečné a musí se upravit:

```
R(config)#crypto ipsec security-association replay window-size <WS>
```

Cisco doporučuje používání největší velikosti, 1024.

#### Dead Peer Detection

V případě vypadnutí DMVPN spojení může dojít, a dochází, k tomu, že IPsec SA je stále aktivní, dokud nevyprší, což vede k blackholed provozu.

IPsec DPD *Dead Peer Detection*, při této funkci před posláním dat skrze tunel, pokud není jisté, že tunel je funkční, se posílají *DPD R-U-THERE* requesty, pokud na ně není odpovězeno pošlou se ještě 5x s rozestupem *Retry* intervalu, pokud ani poté peer neodpoví, vyhlásí se tunel za mrtvý.

```
R(config)#crypto ikev2 dpd <interval> <retry> on-demand     \\ Zapnutí funkce
```

Je doporučeno nastavení interval timeru na dvojnásobek routing protokol hello timeru a retry interval na 5s.

DPD se konfiguruje pouze na spoke, nikoli na hub, routerech, kvůli CPU zdrojům.

#### NAT Keepalives

Jedná se o nástroj, který má udržet záznam u Dynamic NATu pro udržení spojení mezi peery.
NAT Keepalive je UDP paket obsahující nešifrovaný payload 1B.
Když se používá DPD, NAT Keepalives se posílají pokud IPsec peer neposlal nebo nedostal paket v průbehu určitého času.
```
R(config)#crypto isakmp nat keepalive <times>     \\ Zapnutí funkce
```


#### IKEv2 CPU Protection

Vyjednávání jednotlivých IKE SA může být náročné na CPU a v případě přehlcení může dojít nejenom k nevyjednání nových IKE SA, ale narušení stávajících, IKEv2 umožňuje limitovat maximální počet možných IKE SA k navázání:

```
R(config)#crypto ikev2 limit {max-in-negotiation-sa <limit>|max-sa <limit>}[ooutgoing]
```
```
R(config)#crypto ikev2-cookie-challange <NUMBER>     
```

- `max-sa` určuje maximální počet SA, které mohou být navázány
	- Dobré nastavit na 2x předpokládaných spojení, aby se umožňily re-negotiace
- `max-in-negotiation-sa` určuje maximální počet zrovna navazovaných spojení
- `ikev2-cookie-challange` jedná se o ochranu proti DoS útokům, které otevírají IKE SA a nechávají je *Half-Open*, toto nastavení určuje, kolik takovýchto spojení může být před zapnutím coockie challange.