# Funkce

## OSPFv2
---

### SPF Throttling

Po příjmutí nového LSU musí proběhnout nová [[OSPF SPF|SPF]] kalkulace, protože většinou pokud dojde k jedné změně v síti dojde k více změnám, tak LSU s novými LSA přijde více, než jedno, je tedy nevýhodné okamžitě začít s výpočtem. Místo toho OSPF počká nějaký čas, po který je očekáváno, že příjdou i další LSU, a poté je všechny zprocesuje najednou.
Protože LSU mohou být opakované, zejména u chyb v síti typu špatně připojeného kabelu a podobně může být změna v síti generována velmi často, z tohoto důvodu je časovač, po který zařízení čeká sám se zvyšující.
Například na ciscu po prvním LSA se čeká 5s na SPF, pokud po dokončení SPF příjde další čeká se už 10s a tak dále do maxima.

Tyto časovače lze nakonfigurovat právě pomocí funkce **SPF Throttling**.

#### *spf-start*

Jedná se o počáteční hodnotu, kterou SPF čeká na započetí výpočtu po dostání nového LSA.

Defaultně 5s.

#### *spf-hold*

Hodnota mezi jednotlivými SPF výpočty, tato hodnota se po každém průběhu zdvojnásobuje.
Pokud byla síť po tuto dobu od poslední SPF stabilní, příští může čekat pouze *spf-start*, ale *spf-hold* zůstane stejný.

Defaultně 10s.

#### *spf-max-wait*

Jedná se o maximální hodnotu, kterou může *spf-hold* dosáhnout, aby se nedostat do nereálných hodnot, a zároveň definuje čas, po kterém, pokud je síť stabilní, se *spf-hold* vrátí na původní hodnotu a začne se opět od *spf-start* časovače.

Defaultně 60s.

Po prvním příjmutí LSA se SPF kalkulace odehraje po 5s a wait timer se nastaví na *spf-hold*. Po dokončení SPF algoritmu se musí čekat tento wait timer (*spf-hold*), pokud v jeho průběhu nedostane žádné nové LSA, nedochází ke změně a při dalším LSA se bude čekat pouze 5s, ale pokud dostane, SPF kalkulace se odehraje na konci tohoto wait timeru, pokud tedy je wait timer 10s od poslední SPF kalkulace a my od ní dostaneme nové LSA po 7s, další SPF se odehraje za 3s, po této kalkulaci se opět čeká wait timer, který je nyní už (2x *spf-hold*), a tak dále až do uplynutí *spf-max-wait*, po čemž se hodnota wait timeru vynuluje opět na (1x *spf-hold*), pouze pokud po tuto dobu nepříjde nové LSA, jinak se bude udržovat maximální hodnota...
Zároveň, pokud v průběhu wait timeru nepříjde nové LSA, další SPF se může odehrát již po *spf-start* timeru, ale wait timer zůstane nastavený na násobek *spf-hold*.

### LSA Throttling

Tato funkce funguje na stejném principu, jako [[#SPF Throttling]], s tím rozdílem, že ten je pro příjmutí a SPF kalkulaci, LSA Throttling řeší nestabilitu sítě již u zdroje, pokud totiž router vygeneruje změnu, nemůže ji okamžitě poslat, musí se řídit stejným procesem, jako SPF Throttling.
Tato funkce tedy funguje pro každé stejné LSA zvlášť, za stejné LSA je považováno LSA se shodným **ID**, **Type** a **ADV RID**.

#### *start-interval*

Defaultně 0s.

#### *hold-interval*

Defaultně 5s.

#### *max-interval*

Defaultně 5s.

Proces funguje naprosto stejně, jako u [[#SPF Throttling]], s tím rozdílem, že se jedná o odesílání a jména timerů jsou lehce pozměněna...

#### LSA Arrival

Zároveň lze nastavit limit pro příjem stejného LSA, pokud příjde dané LSA vícekrát v nastaveném limitu, další nebude zpracováno.

Defaultní čas je 1s.

### Incremental SPF

V závyslosti na umístění routeru v topologii může být tato funkce více, či méně zrychlující.
Normálně při každé změně topologie router provádí kompletně novou SPF kalkulaci nad celou LSDB, protože věětšinou se jedná pouze o nepatrné změny typu vypadnutí jedné linky, nebo odpojení celého segmentu sítě, je zbytečné propočítávat celý graf, ale stačí přepočítat pouze dotčenou část, tuto změnu právě provede zapnutí *Incremental SPF*.

### OSPFv2 Prefix Suppresison

LSDB obsahuje informace o všech sítích v OSPF doméně, ve většině případů se jedná o tranzitní sítě, tedy sítě mezi routery, typicky `/30` nebo `/31`, bez koncových hostů, přitom finálním důvodem pro OSPF, a protokoly obecně, je zajištění konektivity pro koncové zařízení, tato funkce umožňuje tyto sítě přestat přeposílat a ulevit tak výpočetním a paměťovým prostředkům všech routerů v síti.

Přesný postup je popsán v [RFC6860](https://datatracker.ietf.org/doc/html/rfc6860).

#### Point-to-Point Networks

Jedná se o spoj dvou routerů, typicky `/30` nebo `/31` síť.

##### Advertising

Pro každou p-t-p síť se v LSDB uchovávají 2 linky:
- Type 1 (point-to-point)
	- Popisující sousední router
	- Tento záznam se používá pro SPF
- Type 3 (Stub)
	- Uchovávající informace o subnetu
	- Tento záznam se používá pro generování záznamu do RIB

Samotné LSA poté bude vypadat nějak takto:

```
 LS age = 0                        ;newly (re-)originated
        LS type = 1                       ;router-LSA
        Link State ID = 192.0.2.1         ;RT1's Router ID
        Advertising Router = 192.0.2.1    ;RT1's Router ID
        #links = 2
           Link ID = 192.0.2.2            ;RT2's Router ID
           Link Data = 198.51.100.1       ;Interface IP address
           Type = 1                       ;connects to RT2
           Metric = 10

           Link ID= 198.51.100.0          ;IP network/subnet number
           Link Data = 255.255.255.252    ;Subnet's mask
           Type = 3                       ;Connects to stub network
           Metric = 10
```

##### Hiding

Pro shování této LSA se z router-LSA odstraní Type 3 záznam s IP adresací a síť tedy, i přes její existenci pro SPF, nebude existovat pro RIB.

Následné Type 1 LSA bude vypadat následovně:

```
 LS age = 0                        ;newly (re-)originated
        LS type = 1                       ;router-LSA
        Link State ID = 192.0.2.1         ;RT1's Router ID
        Advertising Router = 192.0.2.1    ;RT1's Router ID
        #links = 1
           Link ID = 192.0.2.2            ;RT2's Router ID
           Link Data = 198.51.100.1       ;Interface IP address
           Type = 1                       ;connects to RT2
           Metric = 10
```

#### Broadcast Networks

Do této sítě je připojeno více zařízení a generuje se z ní Type 2 LSA.

##### Advertising

Type 2 LSA generované z této sítě bude vypadat nějak takto:

```
 LS age = 0                        ;newly (re)originated
        LS type = 2                       ;network-LSA
        Link State ID = 198.51.100.3      ;IP address of the DR (RT3)
        Advertising Router = 192.0.2.3    ;RT3's Router ID
        Network Mask = 255.255.255.0
           Attached Router = 192.0.2.3    ;RT3's Router ID
           Attached Router = 192.0.2.4    ;RT4's Router ID
           Attached Router = 192.0.2.5    ;RT5's Router ID
```

Adresa sítě se následně vypočítá na základě masky a LSID.

##### Hiding

Pro skrytí se nahrazuje původní maska maskou `/32`, čímž se bude o síti vědět z pohledu SPF, protože se bude jednat o IP adresu DR v dané síti, ale pokud router dostane v Type 2 LSA `/32` masku nezabuduje jí do RIB.

Takto se bude chovat OSPFv2, pokud se toto staně zařízení s OSPFv1, do RIB nainstaluje tuto cestu...
Tento stav není ideální a může vytvořit routovací černé díry, ale bude fungovat.

#### NBMA

OSPF obecně na této síti emuluje funkci Broadcast sítě, prefix suppression na tom není jinak a funguje naprosto stejně, jako [[#Broadcast Networks]].

#### Point-to-Multipoint

OSPF tuto síť spravuje jako kolekci point-to-point sítí.

##### Advertising

Router má v této síty v Type 1 LSA Type 1 linky (point-to-point) pro každé přímo připojené zařízení a jednu Type 3 (Stub) linku, ve které posílá vlastní IP adresu a `/32` masku.

Type 1 LSA vypadá tedy nějak takto:

```
 LS age = 0                        ;newly (re-)originated
        LS type = 1                       ;router-LSA
        Link State ID = 192.0.2.7         ;RT7's Router ID
        Advertising Router = 192.0.2.7    ;RT7's Router ID
        #links = 3
           Link ID = 192.0.2.6            ;RT6's Router ID
           Link Data = 198.51.100.7       ;Interface IP address
           Type = 1                       ;connects to RT6
           Metric = 10

           Link ID = 192.0.2.9            ;RT9's Router ID
           Link Data = 198.51.100.7       ;Interface IP address
           Type = 1                       ;connects to RT9
           Metric = 10

           Link ID= 198.51.100.7          ;Interface IP address
           Link Data = 255.255.255.255    ;Subnet's mask
           Type = 3                       ;Connects to stub network
           Metric = 0
```

##### Hiding

Stejně, jako u [[#Point-to-Point Networks]], i zde router prostě vypustí Type 3 (Stub) linku z LSA.

### Stub Router

Tuto funkci definuje [RFC6987](https://datatracker.ietf.org/doc/html/rfc6987), ve skratce zamezuje routeru se stát tranzitním zařízením. Jinými slovy, od tranzitního routeru se očekává, že dále pošle paket dalšímu routeru, oproti tomu stub router mlže forwardovat pakety pouze do přímo připojených sítí.

V praxi při zapnutí router posílá jeho Type 1 LSAs pro tranzitní sítě s infinite metrikou a stub sítě s reálnou metrikou.

Tato funkce se mlže typicky hodit v případě ASBR s BGP spojením, při zapnutí routeru je OSPF konvergence velmi rychlá, ale BGP konvergence může trvat i několik minut, v takovém případě, zejména pokud máme i další, funkční, ASBR v doméně, je kontraproduktivní přeposílání sebe, jako next-hopu do takové sítě, proto lze v OSPF nastavit, aby router počkal na BGP konvergenci a poté vyšel ze Stub stavu.

### Non Stop Forwarding (NSF)

Tato funkce umožňuje zařízením pokračovat ve forwardování i při **Graceful Restart**, tedy oznámenému restartování OSPF procesu.
Tato funkce není pouze Cisco proprietární, ale je i popsaná v [RFC3623](https://datatracker.ietf.org/doc/html/rfc3623), využívá toho, že dnešní routery mají 2 kontrolní centra a zatím co Main-Purpose CPU provádí restart OSPF procesu mohou Line-Cards pokračovat ve forwardování. 
Tomuto přístupu se také říká **routing through a failure**.

Oproti otevřené variantě GR, Cisco vyvinulo vlastní implementaci, kterou nazývá NSF.

Pro správnou funkci musí existovat 2 typy zařízení:

- *restart mode* zařízení
	- Toto zařízení je pod Graceful Restart, ale dále forwarduje dle starých informací
	- V podstatě všechny zařízení
- *helper mode* zařízení
	- Toto je sousední zařízení, které i přes nedostávání Hello paketů stále pokračuje v udržování sousedství
	- Tento mód podporují spíše zařízení vyšších řad typu 7200, 7300, 7600, ASR, CSR

Pro fungování NSF musí být splněno několik podmínek:

- Hardwarová konstrukce musí tuto funkci umožňovat [[CEF]], [[SSO]]
- Router musí provést Graceful Restart, tedy rozeslat Type 9 LSAs, mimo jiné s předpokládanou dobou restartu
- Nesmí dojít ke změně LSDB
- Sousedé musí podporovat Helper mode

Tato funkce je defaultně v IOS zapnuta.

### Graceful Shutdown

Tento nástroj je pro co možná nejrychlejší a nejjednodušší vypnutí OSPF procesu na routeru, bez způsobení velkého problému pro síť.

- Rozpojí všechna OSPF sousedství
- Přepošle všechna jeho LSA s infinite metrikou
- Přepošle Hello pakety s DR/BDR poly nastavenými na `0.0.0.0` a prázdným neighbor listem, čímž si ho všichni přepnou do Init stavu
- Přestane posílat a zpracovávat Hello pakety

Příkaz lze použít i per-interface, v tom případě:

- Rozpojí všechna OSPF sousedství na interfacu
- Přepošle všechna interface LSA s infinite metrikou
- Přepošle Hello pakety s DR/BDR poly nastavenými na `0.0.0.0` a prázdným neighbor listem, čímž si ho všichni přepnou do Init stavu, na daném interfacu


