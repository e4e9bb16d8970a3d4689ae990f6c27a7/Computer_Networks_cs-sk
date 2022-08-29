# IS-IS Operation over Different Network Types
---

## Network Types

[[Routing/Routing Protocols/IS-IS/IS-IS|IS-IS]] má 2 typy sítí:
- **Broadcast**
- **Point-to-Point**

Automaticky zde fungují téměř všechny technologie, Ehternet, HDLC, PPP, GRE, P-t-P Frame Relay nebo sub-interface ATM.  
Problém nastává u  *Partially meshed data link layer* technologií jako Hub and Spoke Frame Relay, v takových situacích má IS-IS problém s navazováním spojení a dochází k vytváření nekompletních RIB.

## Adjacency

- **Down**
	- Defaultní stav
	- Nedostal žádné *IIH* od souseda
- **Initializing**
	- Dostal *IIH* od souseda, ale není potvrzena obousměrná komunikace
- **Up**
	- Plně fungující spojení

Způsob ověření obousměrného spojení se liší na typu sítě:
 
 ### Three-way handshake
 
 Je založen na posílání [[IS-IS Pakety#Hello paket|IIH]] paketu s *adjacency TVL*:
 - **Adjacency three-way state**
	 - Stav adjacency, který má posílající rotuer
- **Extended Local Circuit ID**
	- Lokální Extended Circuit ID
	- Použito pro volbu Designated IS
	- Použito pro identifikaci interfacu pro souseda
- **Neighbor System ID**
	- System ID routeru, od kterého dostal router IIH
	- Určen pro ověření správného routeru
- **Neighbor Extended Local Circuit ID**
	- Extended System ID sousedního interfacu
	- Určen pro ověření správného interfacu
 
 #### Původní implementace (p-t-p)
 
 Původní implementace *Three-way handshaku* obsahovala pouze *Extended Local Circuit ID* a *Adjacency three-way state*, ověření obousměrné komunikace bylo založeno na tom, že když `Router A` dostal *IIH* s *Adjacency three-way state* `down`, je jasné, že `Router A` nezná `Router B`, na to poslal `Router B` *IIH* `routeru A` s *Adjacency three-way state* nastaven na `Initializing`, dle toho bylo jasné, že `Router B` slyší `Router A`, takže `Router A` poslal *IIH* s *Adjacency three-way state* na `Up`.

Tento způsob používá Cisco na p-t-p linkách a lze ho nastavit pomocí `if#isis three-way-handshake cisco`.

#### Nová implementace (broadcast)

Kvůli technologiím jako Frame Relay a ATM, u kterých se může, bez vědomí IS-IS, vyměnit virtual circuit a spoj se může přesunout na jiné zařízení, nebo u Fiber, kde tx vlákno může vést jinam, než rx, je nutné ověřit, že adjacency je navázáno na správném systému a interfacu, od toho se přidalo *Neighbor System ID* a *Neighbor Extended Local Circuit ID*.

Zde nejprve `Router A` pošle *IIH* s vlastním *System ID* a *Extended Circuit ID* s `Down` stavem, na čež `Router B` odpoví s vlastním *System ID*, *Extended Circuit ID* a stavem `Initialization` + přidá sousední hodnoty *system ID* a *Extended Circuit ID*, `Router A` tyto hodnoty ověří a pokud se shodují, odešle stejnou *IIH*, s hodnotamy souseda, se stavem `Up`, načež, pokud se hodnoty shodují, `Router B` odpoví s *IIH* se stavem `Up`.
Pokud se některá z hodnot při kontrole neshoduje, router IIH zahodí bez jakékoliv odpovědi.

Tato metoda je defaultně používána na broadcast a lze ji nastavit pomocí `if#isis three-way-handshake ietf`.
 Mezi oběma způsoby funguje zpětná kompatibilita.
 
 ### Local Circuit ID
 
 Jak známo z [[OSI Network Layer|OSI Network Layer]], IS přiřazuje každému interfacu Local Circuit ID.
 Na obou typech sítí se používá pro identifikaci správného interfacu na sousedním zařízení, na broadcast sítích navíc je používáno jako Pseudonode ID.
Z tohoto důvodu musí na broadcast sítích být Local interface ID unikátní.

 Další problém je, že Local circuit ID je `1B` číslo, umožňuje mít tedy pouze $256$ interfaců, proto byla později uvedena hodnota *Extended Local Circuit ID*, která je `4B` pro maximum $2^4$ interfaců, tato hodnota je automaticky určena pro každý interface routeru a je používána pouze u three-way handshake.
 
 Pro povolení Extended Curcuit ID je nutné zadat příkaz `if#isis three-way-handshake ietf`.
 
 Cisco vnitřně odlišuje u Extended Local Circuit ID zda se jedná o broadcast nebo p-t-p spoj, u p-t-p je odpočítáváno od 256, začíná tedy `0x100`, u broadcast je odpočítáváno od 0, začíná tedy na `0x0`, přeposílán je ale vždy pouze poslední bajt.
 
 ### Point-to-Point

 Na P-t-P spoji očekává IS-IS pouze jednoho souseda.
 IS-IS po vytvoření adjecency vymění kompletní LSDB a dále pouze udržuje spojení.
 
 Prvotní mechanismus navazování spojení nenabízel kvalitní ověření obousměrné komunikace a správného sousedství, proto v [RFC 3373](https://datatracker.ietf.org/doc/html/rfc3373), dnes [RFC 5303](https://datatracker.ietf.org/doc/html/rfc5303), přišel do IS-IS mechanizmus *Three-Way Handshake*, který dnes používá většina implementací.
  Po navázání spojení pomocí Three-way handshake má dojít ke kompletní synchronizaci LSDB.
 Oba routery označí všechny LSP jako *flood*, určeny pro přeposlání, a začínájí si posílat [[IS-IS Pakety#CSNP|CSNP]] pro všechny [[IS-IS Pakety#Link State PDU|LSP]].
 Pokud router zjistí, že soused již dané LSP má označeno za *flood*, odstraní vlastní označení, aby se jedno LSP neposílalo zbytečně 2x.
 Pokud router zjistí, že mu nějaké LSP chybí, nebo má starou verzi, zažádá si o něj pomocí [[IS-IS Pakety#PSNP|PSNP]].  
 Každý přenos LSP, ať update, nebo purge, musí být acknowledged pomocí PSNP, některé implementace umožňují potvrzovat pomocí CSNP.
 Cisco defaultně synchronizuje LSDB pouze jednou, neposílá periodicky CSNP, lze zapnout jejich posílání pomocí `if#isis csnp-interval`.
 
 ### Broadcast
 
 Spíše se jedná o Multiaccess, protože IS-IS diferencuje sítě na základě počtu připojených sousedů a ne možnostech L2 technologie na multicast a broadcast.
 
IS-IS používá [[MAC|MAC]] a [[LLC|LLC]] s DSAP a SSAP nastavenými na `0xfe`.
- Pro Level 1 používá `0180.c200.0014`
- Pro Level 2 používá `0180.c200.0014`

Pro ověření obousměrnosti používá IIH seznam SNAP adres, od kterých dostal IIH, proces tedy funguje stejně jako u [[OSPF|OSPF]], pokud se soused vidí v daném IIH, přepne se na Up, pokud ne, zůstane v Initialization.

IS-IS volí **Designated IS** dle daných hodnot:
1. Nejvyšší priorita interfacu
	- Oproti OSPF je celý rozsah použitelný
2. Nejvyšší SNPA (Subnetwork Point of Attachment) - MAC u ethernetu, DLCI u Frame Relay ...
3. Pokud nelze porovnat SNPA, pak se používá nejvyšší System ID
	- Toto se používá u Frame Relay na fyzických interfacech a ATM na Multipoint subinterfacu, které jsou považovány za broadcast v IS-IS

Nedochází k volbě *Backup DIS*, oproti OSPF, ale funguje preemtivita.
Oproti OSPF, routery na segmentu s DIS navazují plné spojení každý s každým a může si s každým vyměňovat informace.
Úkolem DIS je reprezentovat LSP multiaccess segmentu sítě pomocí pseudonode, pokud by DIS nebylo, muselo by existovat $n(n-1)$ LSP pro každý multiaccess segment sítě, kde $n$ je počet připojených routerů, takto pouze existuje $2n$ LSP, protože z pohledu LSDB takovýto multiaccess segment funguje tak, že každý router navazuje spojení pouze s Pseudonode routerem, DIS.

$1 \times 10s$ DIS generuje CNPS pro celý obsah LSDB a ostatní rotuery ji porovnávají s vlastní, může nastat několik stavů:
1. LSP jsou stejná
	- Nedochází k další akci
2. Routeru chybí LSP, nebo má starší verzi
	- Router si pomocí PSNP požádá o LSP
3. DIS má starší verzi nebo chybí LSP
	- Router okamžitě všem přepošle svou verzi LSP
	- Tuto akci není potřeba acknowledgovat, protože za dalších 10s znovu pošle DIS CNPS se svou LSDB a pokud LSP znovu chybí, celá akce se bude opakovat

Pseudonode ID je nastaveno na local circuit id interfacu DIS routeru v segmentu, ve kterém je DIS.