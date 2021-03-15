# Rapid Spanning Tree Protocol
---

Hlavním rozdílem oproti standartnímu [[STP]] je, že není založen na časovačích, ale na dohodnutí, oznamování, stavu portů. Dalším je, že všechny switche generují BPDUs.

Konvergence topologie probíha pomocí [[#RSTP Synchronization Process]], který je místo časovačů založen na BPDU s flagy.

## RSTP Synchronization Process
---

[[STP#Volba superiorního BPDU|Volba superiorního BPDU]]

1. Stejně jako u [[STP]] se nový switch považuje za Root Bridge, si všechny porty zvolí, jako [[STP Terminologie#Designated|Designated]]([[STP Terminologie#Discarding|Discarding]]) a odesílá z nich BPDUs s vlastním [[STP Terminologie#Bridge ID|BID]] jako [[STP Terminologie#Root Bridge ID RBID|RBID]] a nastaveným *Purpose* bitem ve [[STP Terminologie#Flag#RSTP|Flag]]
2. Switch na segmentu s inferior BPDU odešle BPDU s nastaveným *Agreement* bitem ve [[STP Terminologie#Flag#RSTP|Flag]] + bity popisující stav portu (*Forwarding & Learning & RP=(2)*) 
3. Po tomto zvolení dochází ke změně topologie, takže switch, který změnil stav interfacu (inferior bridge), rozešle 2x BPDU s *TC* bitem ve [[STP Terminologie#Flag#RSTP|Flag]], oproti [[STP]] už neposílá [[STP Terminologie#Topology Change Notification BPDU 0x80|Topology Change Notification BPDU]], pošle je v rozestupu [[STP Terminologie#Hello Time|Hello Time]]
4.  Následně ostatní linky, krom portů v [[STP Terminologie#Edge|Edge]] módu, nastaví na [[STP Terminologie#Designated|Designated]]([[STP Terminologie#Discarding|Discarding]]) a začně z nich rozesílat, ***ne přeposílat!***, Purpose BPDU s RBID Root bridge, zde můžou nastat následující možnosti
	1.  Na oplátku dostane inferior BPDU a následně Agreement
		1.  Přepne se do stavu [[STP Terminologie#Designated|Designated]]([[STP Terminologie#Forwarding|Forwarding]])
		2.  Protilehlý switch se přepne do stavu [[STP Terminologie#Root|Root]] | [[STP Terminologie#Alternate|Alternate]] | [[STP Terminologie#Backup|Backup]]
			- V případě, že už má lepší RP, přepne se do [[STP Terminologie#Discarding|Discarding]] stavu, může být [[STP Terminologie#Alternate|Alternate]] | [[STP Terminologie#Backup|Backup]], v takovém případě pošle Agreement bez Forwarding & Learning bitů se stavem portu *Alternate or Backup (1)*
	2. Dostane superior BPDU, v tom případě se postupuje opačně dle předešlého postupu u 1. .
	3. Dostane superior BPDU se superior RBID, v tom případě se postupuje od bodu 2. .


## Selhání portů
---

V klasické STP, pokud by na inferior port přišel inferior BPDU, port by musel čekat na vypršení superior PBDU po dobu jeho maxage timeru, v RSTP ho příjme okamžitě.

Jako TC je pouze považována změna non-edge portů z non-forwarding stavu do forwarding, vypnutí portu není TC, protože pokud není na dané adresy náhradní cesta, stejně se k nim nelze dostat a pokud existuje náhradní linka, tak změnu zařídí zapnutí této linky.

### Root Port

V případě selhání RP (*direct link failure*) si switch okamžitě přepne na superior Alternate port, není žádné čekání. Vygeneruje TC a pouze oznámí změnu portu, v tomto případě nedochází k Propose/Agreement domluvě.

V případě, že se jedná o *indirect link failure*, switch čeká 3x Hello timer na prohlášení portu za nefunkční.

### Designated

V případě selhání designated portu ho nelze okamžitě nahradit, po selhání portu na stejném segmentu  se přepnou do designated discarding stavu, po nedostání 3 BPDUs, superiorní z nich zůstave v discarding designated stavu a ostatní se, po dostání BPDU od nově vybraného designated portu, vrátí do backup stavu. 
Tento nový port se z discarding modu přepne do learning a následně, po forward delay timeru, do forwarding režimu.

### Přeměna Designated portu na Root port

V případě, že nám vypadne RP, a tak ztratíme komunikaci s Root Bridgem, pokud se jedná o *direct link failure* okamžitě, pokud o *indirect link failure* po vypršení maxage timeru, začneme považovat samy sebe za Root Bridge a začneme rozesílat BPDU s vlastním [[STP Terminologie#Root Bridge ID RBID|RBID]].

Následně se jede synchronizace dle principu RSTP, protože protilehlý switch mu pošle Purpose s původním RBID...


### Příjmutí BPDU s TC

Pokud switch dostane TC z designated nebo root portu:
- Nastaví [[#tcWhile]] timer na všechny non-edge porty, krom toho, ze kterého BPDU přišlo.
- Okamžitě smaže záznamy v CAM tabulce pro tyto porty.
- Začne rozesílat BPDUs s TC flagem ze všech portů označených tcWhile timerem, rozesílá je do vypršení timeru 1x za Hello timer (defaultně tedy rozešle 3 BPDUs s TC flagem)

#### tcWhile

Jedná se o časovač (timer), který je nastaven na [[STP Terminologie#Hello Time]] + 1s.

Ve starším návrhu RSTP byl nastaven na 2x Hello timer, v některých zařízeních nebo zdrojích se s tím stále můžete setkat.


