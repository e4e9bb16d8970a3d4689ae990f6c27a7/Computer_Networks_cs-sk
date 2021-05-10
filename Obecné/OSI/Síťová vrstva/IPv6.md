# Hlavička
---

- Version
	- Verze IP protokolu
- Traffic class
	- Hodnota pro  [[QoS]]
- Flow label
	- Identifikace paketu z toku dat
	- Určeno pro zamezení předbíhání paketů
- Payload lenght
	- Velikost segmentu
- Next header
	- Určuje typ L4 protokolu, nebo odkazuje na další IPv6 hlavičku
- Hop limit 
	- TTL
- Source address
	- Zdrojová adresa
- Destination address
	- Cílová adresa

 ![[IPv6hlavicka.png]]
 
 ## Extended header
 
 - IPv6 hlavička je co možná nejjednodušší, tak z ní ale mohou vypadnout některé potřebné informace
- V takovém případě jsou připojeny další hlavičky s potřebnými informacemi, například
	- Hop-by-hop Options header
		- 0
		- Paket by měl být přečtený každým zařízením v cestě
	- Routing header
		- 43
		- Obsahuje informace pro routing
	- Fragment header
		- 44
		- Obsahuje informace pro End-to-end fragmentaci
	- Destination Options header
		- 60
		- Paket by měl být přečtený pouze koncovým zařízením
	- Authentication header
		- 51
		- Paket obsahuje informace o autentizaci
	- Encapsulating Security Payload header
		- 50
		- Informace o šifrování

# NDP
---
## Pakety

- Router Solicitation
	- Zařízení v síti se pomocí Router Solicitation snaží najít router
	- Posílají požadavek na `FF02∷2`, což je adresa všech routerů
	
- Router Advertisement ^RA
	- Router ohlašuje svou přítomnost, spolu s informacemi o síti 
	- Adresa sítě, MTU
	- Pro dynamické nastavení obsahuje RA 4 hlavní pole
		- M ^mflag
			- 0 - V síti není DHCPv6
			- 1 - V síti je DHCPv6 a klient by ho měl využít
			- Pokud je nastaven O flag, tak je tato hodnota 0 a ignoruje se
		- A ^aflag
			- 0 - Zařízení se má obrátit na DHCPv6 server
			- 1 - IPv6 adresu si má zařízení vygenerovat samo
		- O ^oflag
			- 0 - V síti není DHCPv6
			- 1 - V síti je DHCPv6 a klient by ho měl využít pro ostatní nastavení (DNS, doména…)
		- L ^lflag
			- 0 - Pro komunikaci s jinými zařízeními v síti je nutné použít router
			- 1 - Všechny adresy s tímto prefixem jsou dosažitelmé po L2
			
- Neighbour Solicitation
	- Náhrada za ARP, získávání L2 adresy
	- Požadavek je odeslán pouze na jednu adresu, Solicited-Node Multicast Address, kterou zná, není tak nutné využívat multicast, oproti ARP u [[IPv4]]
	- Jako MAC adresu používají
		- `33:33:` + posledních 8 znaků z Solicited-Node Multicast Address
		
- Neighbour Advertisement
	- Reakce na zprávu s MAC adresou
	
- Redirect
	- Změna first-hop adresy
	- Pochází z routeru pro hostitele

## Funkce

- Router discovery
	- Zjištění směrovače v síti
- Prefix discovery
	- Zjištění adresy sítě
- Parametr discovery
	- Zjištění parametrů linky, například MTU
- Address resolution
	- Mapování L3 a L2 adres
- Next-hop determination
	- Stanovení lepší first-hop adresy
	- ICMPv6 Redirect
- Neighbour Unreachability Detection (NUD)
	- Zjištění, že soused již není dosažitelný
- Duplicate Address Detection (DAD)
	- Zjištění duplikátní adresy
	- Zařízení se pokusí poslat Neighbour Solicitation na svou IPv6 adresu, pokud dostane odpověď, jde o duplikaci

## SLAAC

Díky NDP a Modified EUI-64 si může host nastavit přístup do sítě bez jakéhokoli zásahu administrátora, nebo DHCPv6

# Adresy
---
## Generování adres

### Modified EUI-64

- Způsob automatického získání IPv6 adresy z MAC adresy a adresy sítě
- `R1(config-if)#ipv6 address <Network>/<Mask> eui-64     \\ Vygeneruje Interface ID na základě modified EUI-64`
- Tento příkaz vytvoří adresu v síti 2001:db8:0:0::/64 následujícím způsobem
	1. Rozpůlí MAC adresu (`1234:5678:90AB`) a vloží `FF:FE` -->  `1234:56FF:FE78:90AB` 
	2. Oproti EUI-64, modified EUI-64 převrátí 7. bit `1234:56FF:FE78:90AB` --> `1034:56FF:FE78:90AB`
	3. `2001:db80∷1034:56FF:FE78:90AB/64`


### RID

- Kvůli potenciální nebezpečnosti vytváření adres pomocí Modified EUI-64 bylo zavrženo a v [RFC 7212](https://tools.ietf.org/html/rfc7212) bylo doporučeno používat Random Identifier (RID)
- Do funkce $f()$, doporučuje se SHA-1 nebo SHA-512, se zadají hodnoty $f(Prefix, Net_{Int}, Net_{ID},DAD_{Counter},Secret Key)$
	- $Prefix$ - Sít
	- $Net_{Int}$ - Záleží na implementaci, stabilní identifikace rozhraní
	- $Net_{ID}$ - Specifická data sítě, například SSID, nepovinný údaj
	- $DADCouner$ - Počítá DAD konflikty
	- $Secret Key$ - Neznámé pseudo-náhodné číslo, alespoň 128-bit dlouhé
- Z této funkce je jasné, že adresa bude náhodná, tedy bezpečná, pokud se někdo nezmocní Secret Key, a zároveň stejná pro různé podsítě
- Tato implementace je například v Microsoft Windows

## Adresy

### Global Unicast

- Veřejné, registrované a unikátní adresy
- V současné chvíli se rozděluje `2∷/3`
- Jedná se o všechny adresy, které nejdou vyhrazeny jinému užití, například
	- <b><span style="color:#0040FF"> `2001:0DB8:8D00:` </span></b><b><span style="color:#FF8000"> `001`</span></b><b><span style="color:#31B404"> `∷1`</span></b> `/64`
	- <span style="color:#0040FF">48-bit Global routing prefix</span>, přiřazen ISP
	- <span style="color:#FF8000">16-bit Subnet identifier</span>, používán firmou pro subnety
	- <span style="color:#31B404">64-bit Interface identifier</span>, identifikuje interface v síti

### Unique Local

- Privátní, neregistrované adresy
- Původně FC00∷ /7 rozděleno na 2 rozsahy
	- `FC00∷/8`
	- `FD00∷/8`
	- Ale `FC00∷/8` ještě nebyla alokována, je označována za Refused Unique Local
- Každý ISP router paket s takovouto adresou zahodí
- <b><span style="color:#B40404">`FD` </span></b> <b><span style="color:#0040FF">`45:93AC:8A8F:` </span></b> <b><span style="color:#FF8000">`0001`</span></b>  <b><span style="color:#31B404">`∷1`</span></b> `/64`
	- <span style="color:#B40404">8-bit Unique local identifier</span>, identifikuje lokální adresu
	- <span style="color:#0040FF">40-bit Global ID, identifikuje uskupení sítí</span>, měl by být náhodně generován
	- <span style="color:#FF8000">16-bit Subnet identifier</span>, používán firmou pro subnety
	- <span style="color:#31B404">64-bit Interface identifier</span>, identifikuje interface v síti

### Anycast

- Paket na tuto adresu se pošle nejbližšímu zařízení
- Typicky se používá `<Net::/64>`, neboli první adresa v síti
- `R1(config-if)#ipv6 address 2001:DB8::/64 anycast     \\ Nastavení Anycast adresy`


### Link-Local

- Automaticky generovaná adresa na IPv6-enabled interfacech
- Každý router paket s takovouto adresou zahodí, pokud není určen jemu
- `FE80::/64`
- Následně se adresa generuje dle Modified EUI-64

### Solicited-node Multicast Address

- `FF02::1:FF + posledních 6 znaků Global Unicast adresy`
- `2001:DB8::1:F2A:4FFF:FEA3:B1` --> `FF02::1:FFA3:B1`
- Používá se pro Neighbour Solicitation 

### Multicast Addresses

- Používá <b><span style="color:#0040FF">`FF`</span></b> <b><span style="color:#B40404">`0`</span></b> <b><span style="color:#31B404">`0`</span></b> `::/8`
	- <span style="color:#0040FF">Multicast adresa</span>
	- <span style="color:#B40404">Flag</span>
	- <span style="color:#31B404">Scope</span>

#### Scopes
- Určují, jak daleko v síti má být paket poslán
- Interface Local
	- `FF01∷/8`
	- Neopouštějí zařízení
- Link Local
	- `FF02∷/8`
	- Pouze v rámci subnetu
- Admin Local
	- `FF03∷/8`
- Site Local
	- `FF05∷/8`
	- Může být přeposílán v fyzické lokace
	- Neměl by být přeposílán do WAN
	- Záleží na konfiguraci
- Organization Local
	- `FF08∷/8`
	- Může být přeposílán v rámci subnetů firemní sítě, není fyzicky limitován
	- Neměl by mít přístup do Internetu
	- Záleží na konfiguraci 
- Global
	- `FF0E∷/8`
	- Může být routován internetem
	
#### Flag

1. Nepoužívá se
2. R (Randezvous)
	- 0 - Randezvous point nemá multicast adresu
	- 1 - Randezvous point má multicast adresu
3. P (Prefix) 
	- 0 - Multicast adresa není založena na síťovém prefixu
	- 1 - Multicast adresa je založena na síťovém prefixu
4. T (Transient)
	- 0 - Jedná se o trvale přiřazenou, známou, multicast adresu
	- 1 - Jedná se o lokální, dynamickou, multicast adresu

# Přechody
---
## Dual Stack

- V síti funguje [[IPv4]] a [[IPv6]]
- Aplikace bude používat Stack na základě protokolů jiných vrstev, například [[DNS]]

## Tunneling

- Tunely zapouzdřují IPv6 pakety do IPv4 paketů
- Primárně používáno pro IPv6 komunikaci se vzdálenými hosty skrze IPv4 ISP backbone
- Existuje více způsobů
	- 6to4 [[Tunneling#6to4|6to4]]
	- ISATAP
	- Teredo
	- 6PE
	- 6VPE
	- mGRE v6 over v4 [[Tunneling#GRE|GRE]]
- Tunely mohou být konfigurovány manuálně, nebo dynamicky
