# OSPFv3
---

## OSPFv2 vs OSPFv3

Tyto 2 protokoly sdílejí řadu konceptů, jako základní operaci, sousedství, arei, typy interfaců, virtual linky, metriku a další.
Ale v některých věcech se i liší:

- Podobně, jako [[RIP#RIPng|RIPng]], je i OSPFv3 konfigurováno per-interface
- OSPFv3 dokáže pracovat s vícero IPv6 adresami na jednom interface, oproti OSPFv2, které nepoužívalo *secondary* adresy
- OSPFv3 generuje stejným způsobem RID, jako v2, ale pokud je používána pouze IPv6, musí se roto `32b` číslo nakonfigurovat manuálně
- OSPFv3 má 3 *flooding scopes*, které korespondují s [[IPv6#Scopes]]:
	- *Link-local scope*
		- Používán pro link-local scope, pro Link LSA
	- *Area scope*
		- Používán po celé arei, pro běžné LSA
	- *AS scope*
		- Používán pro celou OSPFv3 doménu, pro AS External LSA
- *Multiple instances per link*
	- Například pro diferenciaci sousedství OSPFv3 umožňuje dělit linky i do instancí
- OSPFv2 používá termín *network*, ale OSPFv3 *link*
- Krom Virtual Linků, OSPFv3 používá pro komunikaci link-local adresy
- OSPFv3 nativně nepodporuje autentizaci, je řešena pomocí IPv6 protokolů

## Typy LSA´s

|                             LSA Type                             |         Jméno         |                                                                                                         Popis                                                                                                          | Flooding Scope |
|:----------------------------------------------------------------:|:---------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:--------------:|
|                [[OSPF Areas & LSAs#Type 1 2\|1]]                 |        Router         |                                   (Každý router, per-area) Tato zpráva obsahuje RID a IP adresy všech interfaců v dané oblasti, včetně *Stub* sítí, přeposílána pouze v rámci arei.                                    |      Area      |
|                [[OSPF Areas & LSAs#Type 1 2\|2]]                 |        Network        |                                          (DR, per-transit network) Vytvořena DR obsahující informace o subnetu a všech připojených zařízeních, přeposílána v rámci segmentu.                                           |      Area      |
|                 [[OSPF Areas & LSAs#Type 3\|3]]                  |      Net Summary      | (ABR, per-area) Vytvořena ABR obsahující informace o sítích a metrice z jedné arei a přeposílána do další, neobsahuje topologické informace, pouze nabízí sebe, jako next-hop, přeposílána v rámci arei a dále na ABR. |      Area      |
| [[OSPF Areas & LSAs#Type 4 & 5 + External Route Types 1 & 2\|4]] |     ASBR Summary      |                                            (ASBR, per-area) Vytvořena ASBR a  přeposílána ABR obsahující informace o cestě k ASBR, přeposílána v rámci arei a dále na ABR.                                             |      Area      |
| [[OSPF Areas & LSAs#Type 4 & 5 + External Route Types 1 & 2\|5]] |      AS External      |                                                      (ASBR, OSPF doména) Vytvořena ASBR obsahující redistribuované sítě do OSPF, přeposílána v celé OSPF doméně.                                                       |       AS       |
|                                7                                 |         NSSA          |                                                   (ASBR, intra-area) Vytvořeno v NSSA, jedná se o kamufláž pro LSA 5, ABR na hranici NSSA přepošle LSA 7 jako LSA 5.                                                   |      Link      |
|                                8                                 |       Link LSA        |                                                       Oznamuje link-local adresy a prefixy routeru všem ostatním na lince, pouze pokud je na lince více zařízení                                                       |      Link      |
|                                9                                 | Intra-Area-Prefix LSA |                                      Spojuje list IPv6 prefixů s tranzitní sítí identifikovanou s Type 2 LSA a spojuje IPv6 prefixy s routerem identifikovaným pomocí Type 1 LSA                                       |      Area      |


Nové Type 8 & 9 LSA´s řeší fakt, že síťové (IP adresy a masky) informace a topologické informace (Propojené RID), jsou odlišné a OSPFv2 je posílá dohromady v Type 1 & 2 LSA´s, v některých případech, typicky při změně adresy v síti, může dojít k LSU a tím provedení SPF, i přesto, že se změnila pouze adresace, ale ne linkový graf.
OSPFv3 dělí topologické a síťové informace do více LSA´s, přesněji topologické zůstávají v Type 1 & 2 a síťové se přesouvají do Type 9 LSA, s možností sdílet link-local adresy, pro ty je Type 8 LSA.

## Autentizace

- Používá AH pro autentizaci
- Poučívá ESP pro šifrování a autentizaci
- Je postavena na IPSec
- Mezi routery se musí shodovat SPI, AH/ESP módy, algoritmy a klíče

OSPFv3 využívá IPSec a *Authentication Header* (AH) pro autentizaci s *Encapsulating Security Payload* (ESP) pro šifrování.
Tato varianta je běžnější, protože ESP enkapsuluje a šifruje celý paket, ale OSPFv3 podporuje i OSPFv2-like (key chain) autentizaci.

## Graceful Shutdown

Oproti OSPFv2, ve kterém se vypne proces okamžitě, u v3 se čeká *dead interval*, aby se mohla změna dopropagovat síti a všechny sousední zařízení odpojit.