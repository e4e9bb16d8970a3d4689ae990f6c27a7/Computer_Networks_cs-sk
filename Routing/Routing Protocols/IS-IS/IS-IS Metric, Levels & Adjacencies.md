# IS-IS Metrika, Levely & Adjacencies 
---

## Metrika

[[Routing/Routing Protocols/IS-IS/IS-IS|IS-IS]] přisuzuje metriku linkám, nabízí několik typů metrik, vyšší = horší:
- **Default**
	- Defaultní metrika, kterou musí každá implementace podporovat
- **Delay**
	- Informuje o spoždění na lince
- **Expense**
	- Informuje o peněžní ceně posílání dat přes danou linku
	- Výhodné u WAN
- **Error**
	- Informuje o *error rate*

Reálné chování záleží na implementaci, Cisco podporuje pouze *Default* metriku, cost linky je defaultně 10, není upravován na základě bandwithu, administrátor ho musí ručně změnit.
Ostatní metriky Cisco umožňuje nastavit, ale jsou posílány jako *unsopported* a není na ně brán zřetel.

Nad každou metrikou je vypočítáván vlastní SPF strom, znamenaje, že normálně by mělo IS-IS nabízet 4 routovací tabulky.

Původní metriky, *narrow metric*, přidělují `6b` každému interfacu pro metriku a `10b` pro sumární metriku cesty, to se ukázalo jako nedostatečné, a tak v [RFC 3784](https://datatracker.ietf.org/doc/html/rfc3784), dnes [RFC 5305](https://datatracker.ietf.org/doc/html/rfc5305), IS-IS používá *wide metric*, která nabízí `24b` pro interface a `32b` pro celou cestu spolu s podporou pro *MPLS Traffic Engineering*.

## Levely

IS-IS je hierarchicky založeno na [[OSI Network Layer#OSI Routing|Level Routing]], funguje na L1 a L2.
Level 2 je označován za *backbone*, protože má za úkol směrovat mezi areami.
Level 1 je označení pro jednotlivé arei.

## Adjacencies

Routery fungují v IS-IS odděleně pro každý level, navazují oddělené sousedství, vedou oddělené LSDB a posilájí speciální LSP.
Router může navázat zařízení pouze s vlastním levelem, pokud se jedná o L1, tak pokud se nachází ve stejné arei, pokud sousedící routery podporují L1 i L2, navážou separátní adjacencies pr okaždý level.
Defaultní nastavení pro Cisco je L1/2.

| 1. IS Level  | 2. IS Level  |               Adjacency                |
|:------------:|:------------:|:--------------------------------------:|
| Level 1 only | Level 1 only |     Level 1, pokud je shodná area      |
| Level 1 only | Level 1 + 2  |     Level 1, pokud je shodná area      |
| Level 1 only | Level 2 only |                  Bez                   |
| Level 1 + 2  | Level 1 + 2  | Level 1, pokud je shodná area; Level 2 |
| Level 1 + 2  | Level 2 only |                Level 2                 |
| Level 2 only | Level 2 only |                Level 2                 |


NSAP adresa je přiřazena nódu a ze své podstaty může být pouze jedna originální v aree a doméně.
NA routeru v IS-IS mohou být v jednu chvíli až 3 NSAP adresy, pokud sdílí stejné System ID a liší se tedy pouze v arei/doméně.
I přes více adres, a tím teoreticky více areí, router nevytváří další LSDB, ale všechny informace spojuje do jedné, díky tomu IS-IS umožňuje velmi důležité funkce jako:
- Spojování areí
- Přenastavování Area ID

V těchto případech můžeme prostě nejprve nastavit nové NSAP adresy a, protože sdílejí jednu LSDB, po odstranění starých nedojde k narušení provozu

Více NSAP adres je v IS-IS doporučeno používat pouze v těchto specifických situacích.