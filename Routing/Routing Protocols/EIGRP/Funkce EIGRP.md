# Další funkce EIGRP
---

## Router ID
---

RID je `4B` číslo označující router, defaultně má každá instance EIGRP vlastní, ale je povoleno použití stejného RID pro více instancí a procesů.
 Původně bylo používáno pro prevenci routing-loops u redistribuovaných cest, ke každé externí cestě bylo připojeno RID routeru, ze kterého byla redistribuována a pokud se připojené RID shodovalo s lokálním, router ji zahodil.
 
 V novějších verzích je ale RID připojeno i v interních cestách, se kterými funguje stejně.
 
 Volba RID je stejná, jako u [[OSPF]]:
 1. Nakonfigurovaná hodnota
 2. Nejvyšší IP adresa loopback interfacu
 3. Nejvyšší IP adresa zapnutého interfacu

Krom manuálního zadání, reinitializace RID probíhá pouze při restartu procesu.
Při přenastavení RID se musí reinitializovat i sousedství.
Teoreticky je možné, že více zařízení bude mít stejné RID, v takovém případě vše funguje, ale tyto zařízení od sebe navzájem nepřijímají cesty. Tento problém se těžko detekuje a lze ho zjistit pouze pomocí `show eigrp address-family ipv4 events`.
Zobrazit současné RID lze pomocí 2 příkazů: `show ip protocols` a `show ip eigrp topology`, ve starších verzích ip protocols nefunguje.

## Unequal-Cost Load Balancing
---

Jedná se o unikátní funkci EIGRP umožňující load-balancovat na základě metriky cest. [[Routing#Unequal-Cost Load Balancing|Unequal-Cost Load Balancing]]

Pro nastavené je nutné mít Feasible Successor/y, protože pouze u nich je možné zaručit loop-free cestu.
Dále je pro zprovoznění potřeba určit *Variance*, neboli poměr, dle kterého se bude load-balancovat, ten se vypočítá následovně:

$$
V=\frac{Feasible\;Successor\;CD}{Successor\;CD}
$$
$$
CD\;via\;Successor < CD\;via\;Feasible\;Successor<V \times CD\;via\;Successor
$$

s tím, že se zaokrouhluje nahoru.
Hodnota 1 znamená, že UCLB není používáno, současnou hodnotu lze zjistit pomocí příkazu `show ip protocols`.
Unequal-Cost paths se započítávají do maximálního počtu cest, a tak je možné, že bude potřeba upravit maximální počet cest pomocí `maximum-paths` příkazu.
V případě více takovýchto cest následně IOS load-balancuje na základě následující rovnice:

$$
Nejvyšší\;CD/CD\;cesty
$$

V tomto ohledu je lepší o Variance mluvit spíše jako o maximální hodnotě poměru metrik, například, pokud máme cesty s metrikou:

1. 131072
2. 156416
3. 2690816

Můžeme nastavit Variance na $\frac{131072}{156416}=8$, v takovém případě se podmínka poměrů splní pouze pro 2. cestu a bude se tedy load-balancovat pouze s prvními 2.
Pokud bychom ale chtěli využít všechny, musíme nastavit variance threshold výše $\frac{2690816}{156416}=18$, poté se využijí všechny cesty.

Samotný load-balancing si následně vypočítá CEF na základě rozdělení pointerů v *load-share table*, přesný způsob je popsaný v ([[CEF#Load-Sharing]]).

```R
R1#show ip cef 1.1.1.1 internal 
1.1.1.1/32, epoch 0, RIB[I], refcnt 5, per-destination sharing
  sources: RIB 
  feature space:
    IPRM: 0x00028000
  ifnums:
    GigabitEthernet0/0(2): 10.0.0.2
    GigabitEthernet0/1(3): 10.0.1.3
    GigabitEthernet0/2(4): 10.0.2.4
  path list 0FA2CC94, 3 locks, per-destination, flags 0x49 [shble, rif, hwcn]
    path 1139E584, share 3/4, type attached nexthop, for IPv4
      nexthop 10.0.0.2 GigabitEthernet0/0, IP adj out of GigabitEthernet0/0, addr 10.0.0.2 1029CDD8
    path 1139E5EC, share 67/67, type attached nexthop, for IPv4
      nexthop 10.0.1.3 GigabitEthernet0/1, IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
    path 1139E654, share 80/80, type attached nexthop, for IPv4
      nexthop 10.0.2.4 GigabitEthernet0/2, IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
  output chain:
    loadinfo 1139F008, per-destination, 2 choices, flags 0002, 5 locks
      flags [for-rx-IPv4]
      16 hash buckets
        < 0*> IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        < 1 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        < 2 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        < 3 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        < 4 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        < 5 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        < 6 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        < 7 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        < 8 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        < 9 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        <10 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        <11 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        <12 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        <13 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
        <14 > IP adj out of GigabitEthernet0/1, addr 10.0.1.3 1029CCA8
        <15 > IP adj out of GigabitEthernet0/2, addr 10.0.2.4 116C7518
      Subblocks:
        None
```

Zde je vidět poměr, $4:67:80$, další 3 pakety půjdou na `10.0.0.2` poté dalších 67 paketů půjde na `10.0.1.3`.

## Add-Path Support
---

V určitých topologiích, typicky [[DMVPN]], může vzniknout situace, ve které ma 2 a více *Spoke* routerů společnou síť, a tak *Hub* router, znaje všechny cesty, může provádět load-balancing, ale vzhledem k tomu, že cestu do určité sítě v EIGRP vždy router sdílí pouze jednu, a to tu na sebe, ostatní spoke routery tedy nevědí o více cestách do jedné sítě a nemohou tedy provádět load-balancing.

IOS `15.3(2)T` přidal do Named módu podporu *Add-Paths*, umožňující routeru sdílet více equal-cost cest do jedné sítě. Tuto funkci ale není možno využívat s UCLB a Variance tedy musí být nastaveno na 1 a zároveň musí být vypnuta funkce Split Horizon.

```
R_HUB(config)#router eigrp <NAME>
R_HUB(config-router)#address-family ipv4 unicast autonomous-system <AS>
R_HUB(config-router)#af-interface <IF (Tunnel)>
R_HUB(config-router-af-interface)#no split-horizon     \\ Vypnutí split horizon
R_HUB(config-router-af-interface)#no next-hop-self {no-ecmp-mode}    \\ Vypne sdílení sebe jako next-hopu
R_HUB(config-router-af-interface)#add-paths <NUMBER>     \\ Přenastaví macimální počet sdílených cest
```

Při použítí více ISP, nebo spojů obecně, je nutné vzít v potaz další eventualitu. Hub se v takovém případě naučí o cestách z více, než jednoho, Tunnel interfacu. Například rovnocenné cesty přes `Tunnel1` $N_{11}, N_{12}$ a přes `Tunnel2` $N_{21},N_{22}$. 

Při použití pouze příkazu `next-hop-self` EIGRP pouze redistribuuje první záznam a kontroluje, zda je daná síť dostupná na výstupním Tunnel interfacu, díky vypnutému split horizon, pokud ano, nebude přeposílat sebe jako next-hop, ale nechá svého vlastního Successora do dané sítě. Ostatní záznamy této kontrole nejsou podrobovány, a tak je možné, že v případě užití příkazu `no next-hop-self` bude tento příkaz v takovém případě ignorován a i nadále bude router nabízet sebe jako next-hop.
Například pokud máme cestu $N_{11}$ přeposílanou přes interface `Tunnel1`, přes který je dostupný, router nebude sám sebe nabízet jako next-hop, ale pokud ho bude dále přeposílat i na interface `Tunnel2`, ze kterého přichází v podstatě stejná cesta ($N_{21}$), původní kontrola již není prováděna a příkaz `no next-hop-self` je tedy ignorován. 
Argument `no-ecmp-mode` tuto optimalizaci vypíná a router tedy pokaždé bude provádět kontrolu, zda je síť na interfacu dostupná.

## EIGRP Stub Routing
---

Jedná se o funkci, která má zvýšit stabilitu a scalabilitu sítě, typicky se využívá ve velkých sítích, nebo v případě připojení po pomalých WAN linkách.
Nejvíce se používá v Hub-and-Spoke topologiích, v takovém případě se nastavuje na Spoke routerech.
Při nastatavení posílá Stub router další TLV v Hello zprávách, kterým se značí jako Stub.

Nastavení Stub routeru pozměňuje řadu funkcí:

- Stub router nikdy nepřeposílá cesty naučené pomocí EIGRP, což ho vyřazuje pro jeho zvolení jako Successor a FS
	- Toto pravidlo má vyjímku, jedná se o [[#leak-map]]
- Stub router pouze přeposílá vlastní připojené cesty na EIGRP-enabled interfacech 
	- Toto lze opět specifikovat pomocí *summary*, *connected*, *static*, *redistributed*, *receive-only*
	- Defaultně přeposílá přímo připojené a sumární cesty
- Stub router nebude zahrnut do diffusing algoritmu, protože mu sousedé neposílají Query, pokud se ale nejedná o sdílené médium, v takovém případě se Query posílají a Stub router si s nimi musí poradit
	- V případě, že na společném segmentu jsou ale i routery bez Stubu, buď se posílají Query unicestem, nebo se využívá funkce [[EIGRP#Reliable Transport Protocol RTP|RTP]] a CR flag.

### Query handling

Vytváření Query v případě výpadku není nijak Stub funkcí pozměněno.

V případě přijímání Quey záleží na síti, v případě, že se jedná o síť výše a níže uvedenými způsoby povolenou (*summary*, *connected*, *static*, *redistributed*, *receive-only*, *leak-map*), Query je zpracováno normálně a router se může i stát Successorem.
V případě, že se jedná o síť ne-specificky povolenou, router odpoví, i pokud ji zná, s *Infinite* metrikou, aby se nemohl stát Successorem.

### Funkce

#### recieve-only

Toto nastavení zakazuje přeposílání všech cest,včetně přímo připojenách, pouze bude přijímat.

#### leak-map

Tato funkce umožňuje nastavit [[ACL]] a specifikovat,které sítě se nají sdílet.

## Route Summarization
---

 Koncept vydávání více jednotlivých sítí za jednu společnou síť schovaje tak změny v jednotlivých sítích zjednodušuje routing a zlepšuje konvergenci i stabilitu sítě, na druhou stranu je potřeba pečlivě navrhnout síťovou topologii.
 
 ### Automatická
 
 Historicky EIGRP podporuje 2 druhy sumarizace: automatickou a manuální.
 Automatická byla používaná v době *classfull* routingu, pokud chceme se zapnutou autosumarizací sdílet síť `10.15.215.0/24`, EIGRP ji bude propagovat jako `10.0.0.0/8`, protože se jedná o `Class A` síť. Tato sumarizace se neaplikuje na externí cesty, pokud není shodná síť i uvnitř sítě.
 EIGRP je ale classledd protokol a funkce autosumarizace byla zabudovaná pouze pro zpětnou kompatibilitu, respektive jednodušší přechod od classfull k classless routingu. Od verze IOS `10.0(1)M` je automaticky vypnutá.
 
 ### Manuální
 
 Manuální sumarizace umožňuje přesně defunovat jaké sítě se mají šířit z vybraného interfacu.
 Oproti [[RIP]] EIGRP podporuje *supernetting*, možnost sumarizovat na menší síť, než je classfull varianta.
 Také je možné nastavit na interface více, i překrývajících se, sumárních adres, EIGRP poté bude používat všechny ty, ke kterým má nějaký záznam v RIB.
 
 S každou sumární cestou instaluje EIGRP i *discard route* do RIB, ta se stará o to, že pokud máme cesty do sítí `10.15.10.0/24` a `10.0.5.0/24` a sumární cesta je `10.0.0.0/8` a příjde nám požadavek do sítě `10.1.0.0/24`, poté by se mohla využít defaultní cesta, protože my cestu do dané sítě nemáme, přeposlání na defaultní cestu by v tomto případě mohlo a s největší pravděpodobností vytvořilo smyčku. 
 Proto se instaluje i cesta se stejnou adresou a maskou, jako sumární, která má egress interface `Null0`, čili zahození.
 Discard route má AD 5, což může v některých případech být problém, například při sumární cestě `0.0.0.0/0` může tato discard routa nahradit naši defaultní cestu, v takovém případě je potřeba AD přenastavit [[EIGRP Konfigurace#Sumarizace]].
 V případě nastavení AD na 255 se u některých starších IOSů vypne instalace cesty do RIB, u novějších se ale i vypne přeposílání sumární cesty, nastavení tak nebude fungovat.
 
 Metrika přeposílaná v sumární cestě je nejnižší metrika ze všech jednotlivých cest, v případě, že sumární adresa spojuje stovky a tisíce síťí může ke změně metriky docházet poměrně často, v takovém případě sumární cesta naopak ztěžuje konvergenci sítě, a proto je možné nastavit metriku staticky.
 
 ### Query
 
 Sumarizace také rozbíjí Query doménu, každá sumarizovaná síť je vlastní doménou, což opět zvyšuje stabilitu a scalabilitu.
 Toto je způsobeno jednoduše tím, že při dostání Query buď router má přesnou cestu, to jest v případě dotazování na cestu `10.0.0.0/24` a my máme přímo cestu `10.0.0.0/24`, pak můžeme poslat Reply, nebo nemáme přímo danou cestu, včetně sumarizovaní, máme například pouze `10.0.0.0/16`, tato cesta se nepočítá, a tak router odešle Reply s nekonečnou metrikou, zabraňujíc tak šíření Query dále v síti.
 
 ## Passive Interface
 ---
 
 Defaultně EIGRP začne rozesílat Hello pakety ze všech interfaců, které mají specifikované sítě, toto je nejenom zbytečně náročné na prostředky, protože do takových sítí spadají i například `Loopback` interfacy, ale i bezpečnostní riziko, protože ne vždy chceme z konkrétního interface sdílet routovací informace.
 
 Stejně, jako u [[RIP]], lze nastavit passive interface [[EIGRP Konfigurace#Základní konfigurace]].
 
 ## Graceful Shutdown
 ---
 
 Když EIGRP dopředu ví, že bude vypnuto na interfacu nebo při vypnutí routeru, rozešle ze zasažených interfaců zprávy, která umožní sousedům okamžitě reagovat na změnu, nemusejí tak čekat na vypršení Hello timeru.
 V praxi tato zpráva je normální Hello paket se všemi K-Values nastavenými na 255.
 
 V Classic módu je možné využít Graceful Shutdown pouze v IPv6 módu použitím příkazu `shutdown`, následně se tato funkce využívá při vypnutí interfacu, přepnutí interfacu na passive, odstranění sítě na interfacu z `network` konfigurace.
 V Named módu lze použív příkaz `shutdown` na několika místech:
 - V `router eigrp` režimu, což způsobí vypnutí celého protokolu
 - V jednodlivých *af*
 - Na jednotlivých interfacech

## Autentizace
---

Již od prvního vydání EIGRP podporuje kontrolu integrity paketů a autentizaci pomocí MD5, od verzí IOS `15.1(2)S` a `15.2(1)T` podporuje i SHA-256.
MD5 lze nakonfigurovat v Classic i Named módu, SHA-256 pouze v Named.

[[EIGRP Konfigurace#Autentizace]]

## Default Route
---

EIGRP nemá speciální příkaz, podobně jako [[RIP]], tato potřeba je řešena [[#Route Summarization]] nebo redistribucí.

V některých materíálech je napsáno, že vytvořit defaultní cestu je možné pomocí příkazu `network 0.0.0.0`, toto však ve většině případů není pravda.
Protože EIGRP defaultně přeposílá pouze directly-connected sítě, tato cesta by musela být nakonfigurována staticky s výstupním interfacem, na kterou RIB pohlíží, jako na directly-connected ([[Routing#Directly Attached Static]]). Nastavení místo next-hopu interface je samo o sobě ve většině případů nedoporučeno ale i EIGRP má s tímto nastavením problém, takto nastavená síť totiž i zapne EIGRP na všech sítích a interfacech, z tohoto důvodu se nejedná o doporučené nastavení.

[[EIGRP Konfigurace#Default Route]]


