# Point-to-Point Protocol

## Popis
L2 protokol používaný pro komunikaci mezi dvěma uzly.

## Funkce
-   Ověřování pomocí hesla
-   Podporuje více vyšších protokolů
-   Používá LCP (Link Control Protocol)
-   Umožňuje kompresi dat CCP (Compression Control Protocol)
-   Šifrování pomocí ECP
-   Multilink

## Rámec

-   **Flag**
	-   1 B
	-   `01111110`
-   **Destination Address**
	-   1 B
	-   Typicky FF, na PPP je pouze Broadcast
-   **Control**
	-   1 B
	-   Používáno v [[HDLC]], nepoužíváno u PPP
-   **Protocol**
	-   1 - 2 B
	-   Identifikuje vyšší protokol
		-   [[PPP#LCP|LCP (Link Control Protocol)]]
		-   IP
		-   [[PPP#NCP|NCP (Network Control Protocol)]]
		-   …
-  **FCS**
	-   2 B
	-   Kontrola chyby
-   **Flag**
	-   Ukončuje frame

![[ppp.gif]]

## LCP & NCP

### NCP

Je soubor protokolů, které formují PPP komunikaci

-   **Internet Protocol Control Protocol (IPCP)**
	-   Konfiguruje IP na PPP
	-   Nastavuje IP adresy
	-   Zapíná / Vypíná IP protokol
-   **OSI Network Layer Control Protocol (OSINLCP)**
	-   Odpovědný za zapínání / vypínání vyšších protokolů OSI
-   **IPv6 Control Protocol (IPV6CP)**
	-   Konfiguruje [[IPv6]] adresu
	-   Zapíná IP protokol na PPP
-   Jsou i další, méně využívané

### LCP

   

Stará se o navázání, konfiguraci, testování, obstarávání a ukončování spojení na PPP.

Vyjednává o možnostech PPP spojení.

#### Funkce

-   Kontroluje identitu
-   Určuje velikost MTU
-   Identifikuje nekompatibilní konfiguraci
-   Stará se o linku a testuje ji
-   Ukončuje spojení, pokud nefunguje správně

#### Framy

-   LCP Configuration
-   LCP Termination
-   LCP Maintenance
	-   Code
		-   1 B
		-   Určuje druh framu
	-   ID
		-   1 B
		-   Určuje, zda se jedná o request nebo reply
	-   Lenght
		-   2 B
		-   Obsahuje velikost LCP framu

## Konfigurace 

### Multilink

Umožňuje spojit rychlosti více sériových linek.
Oproti [[Port-Chanel|Port-Chanelu]], který využívá *load-balancing*, PPP Multilink doslova spojuje sériová spojení v jedno, každý frame, poslaný tímto spojen je totiž fragmentován a každá linka přenese stejnou část framu.

```
R1(config-if)#encapsulation ppp \\ Nastavení encapsulace
R2(config-if)#encapsulation ppp
```
```
R1(config)#interface multilink <číslo> \\ Vytvoření multilink interfacu
R1(config-if)#ip address <IP> <Maska> \\ Nastavení IP, které se nastavuje na multilink interface
```

```
R2(config)#interface multilink <číslo>
R2(config-if)#ip address <IP> <Maska>
```
```
R1(config)#interface Serial <if_number> \\ Přiřazení interface do multilinku
R1(config-if)#ppp multilink group <číslo>
R1(config)#interface Serial <if_number>
R1(config-if)#ppp multilink group <číslo>
```
```
R2(config)#interface Serial <if_number>
R2(config-if)#ppp multilink group <číslo>
R2(config)#interface Serial <if_number>
R2(config-if)#ppp multilink group <číslo>
```

### Authentizace
```
R1(config)#username jméno password heslo \\ Nastavení přístupových údajů
R2(config)#username jméno password heslo
```
#### PAP

PAP posílá heslo jako plain-text.
```
R1(config-if)#ppp authentication pap  \\ Nastavení požadování ověření

R2(config-if)#ppp pap username <Jméno>  \\ Nastavení ověření
R2(config-if)#ppp pap password <Heslo>
```
#### CHAP

CHAP posílá heslo jako MD5 hash, Cisco proprietární
```
R1(config-if)#ppp authentication chap  \\ Nastavení požadování ověření

R2(config-if)#ppp chap username <Jméno> \\ Nastavení ověření
R2(config-if)#ppp chap password <Heslo>
```


# PPPoE

V jednoduchosti vytvoří PPP vrstvu nad Ethernetem.

Jedním z hlavních důvodu pro PPPoE je bezpečnost, při DSL technologii je spojení realizováno ne samostatně, ale často ve formě jakých si ringů, kvůli tomu bylo dříve možné odposlouchávat i provoz sousedů.

Dnes je tato technologie využívána zejména pro autentizaci internetového připojení.
![[dsl.jpg]]
## Konfigurace

### Server

```
Server(config)#username <Jméno> password <Heslo>  \\ Jedná se o heslo pro autentizaci
Server(config)#bba-group pppoe global \\ BroadBand Access group, která je používaná pro PPPoE session
Server(config-bba-group)#virtual-template <číslo> \\ Virtual-template, jedná se o virtuální interface
```
```
Server(config)#interface virtual-template <číslo>
Server(config-if)#ip address IP Maska
Server(config-if)#mtu 1492 \\ Protože PPP a PPPoE přidává dalších 8 Bytů do hlavičky
```
```
Server(config-if)#ppp authentication chap pap \\ Nastavení typů autentizace
Server(config-if)#peer default ip address pool <Jméno> \\ Lokální NCP služba, přiřadí adresu
Server(config)#ip local pool <POOL> <Start_IP> <End_IP>
```
```
Server(config)#interface <if>
Server(config-if)#pppoe enable group global
```

### Klient

#### PPPoE Dial-Up

```
Client(config)# interface <if>
Client(config-if)# pppoe-client dial-pool-number <číslo> \\ Spojení interfaceu s virtuálním pro dial-up
```

```
Client(config)# interface dialer1 \\ Na tomto virtuálním interfacu nastavíme PPP
Client(config-if)#dialer pool <číslo> \\ Spojení s interfacem
```

```
Client(config-if)# mtu 1492 \\ Protože PPP a PPPoE přidává dalších 8 Bytů do hlavičky
Client(config-if)# ip tcp adjust-mss 1452
Client(config-if)#ip address negotiated \\ Automaticky získá IP adresu
Client(config-if)# encapsulation ppp
Client(config-if)# ppp chap hostname <Jméno> \\ Samotná autentizace
Client(config-if)# ppp chap password <Heslo>
```




