# Areas in IS-IS
---

IS-IS identifikuje areu pomocí [[OSI Network Layer#NSAP|AFI]] + [[OSI Network Layer#NSAP|IDI]] + [[OSI Network Layer#NSAP|HO-DSP]] pole, tedy všechna pole po [[OSI Network Layer#NSAP|System ID]].
Je to logické, protože, pokud mají být zařízení ze stejné arei, musí být i ze stejné domény.

Pro oddělení informací IS-IS používá oddělené LSDB pro L1 a L2 arei.

## [[OSI Network Layer#Level 1 Routing|Level 1]]

L1 router sdílí přímo připojené IP sítě v jeho LSP.
2 L1 routery s jinou areou spolu nikdy nenaváži spojení ([[IS-IS Metric, Levels & Adjacencies#Adjacencies]]).

## [[OSI Network Layer#Level 2 Routing|Level 2]]

Pokud by se používal OSI L3 protokol, routery by si přímo nesdíleli informace o NSAP adresách, ale sdíleli by na této úrovni Area ID pomocí L2 LSP, protože, aby se paket dostal k určitému ES, musí splňovat Area ID (v případě Levelu 3 zase Domain ID), úkolem Level 2 routingu je routeovat mezi temito Areami, lze tedy princip porovnat se sumarizací.
Každý L2 router sdílí své přímo připojené sítě a k tomu ještě L1 sítě z vlastní arei, pokud nějaké má, v takovém případě se jedná o L1/2 router.

I přes to, že se uchovávají oddělené L1 a L2 LSDB, i na L1/2 routerech, tak pro šíření L1 informací do L2 router nejprve vypočítá trasy pro L1, na základě L1 LSDB, a pak tyto informace vkládá přímo do L2 LSP, nikoli do L2 lSDB.

Tento proces ale nefunguje opačně, z L2 do L1 se žádné sítě, pokud nejsou specificky nakonfigurovány, nepronikají, pokud tedy L1/2 router zjistí, že skrze L2 areu má dostupné další L1 arei, začne šířit cestu na sebe jako *default gateway*, IS-IS L1 arei v tomto ohledu tedy fungují jako [[OSPF Areas & LSAs#Totally stubby Area|OSPF Totally Stubby Area]].
L1 routery ale mohou redistribuovat další routovací protokoly, ty se po přepočtení metriky objeví jako vnitřní cesty sítě, funguje zde tedy koncept blíže [[OSPF Areas & LSAs#Totally NSSA|Totally NSSA]].

## ATT/P/O

Při zobrazení IS-IS databáze:

```
IS1#show isis database

IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
IS1.00-00           * 0x0000002F   0x6853        788               0/0/0
IS2.00-00             0x0000002E   0xE319        886               0/0/0
IS2.01-00             0x0000002C   0x2116        746               0/0/0
IS3.00-00             0x0000002B   0x2318        468               1/0/0
IS3.01-00             0x0000002A   0x40F3        1176              0/0/0
```

se v pravo tabulky objevují důležité informace.

- ### ATT
	- *Attached bit*
	- Pokud L1/2 router při SPF kalkulaci zjistí, že může routovat do jiných areí (LSP přenášejí číslo areí), krom své, nastaví na `1`
	- Ostatní routery v arei si nastavují defaultní cestu přes routery s tímto bitem
- ### P
	- *Partition Repait bit*
	- Určuje, zda je router schopen opravit areu přes L2 sub-doménu, funkce je podobná, jako [[OSPF Areas & LSAs#Virtual Link|OSPF Virtual Link]]
	- Tato funkce na Cisco zařízeních není podporována, obecně také ne, a je vždy `0`
- ### O
	- *Overload bit*
	- Defaultně tento bit router nastaví pokud se mu již do LSDB nevejdou nová LSP, jinými slovy, má *overloaded* paměť a pracuje s nekompletní LSDB
	- Způsobí, že ostatní rotuery ho nebudou považovat za možný bod v cestě a nebudou ho zahrnovat do SPF kalkulace
		- Nadále ale budou přes něho routovat pokud není jiná možnost, například na přímo připojené sítě daného routeru
	- Dnes je tento bit používat, protože ho lze manuálně nastavit, k jiným účelům:
		- Při změně konfigurace, vypnutí/restartování zařízení
			- Jedná se o lepší řešení, protože dojde k okamžitému přepočítání nové trasy bez daného zařízení a nemusí se čekat na [[IS-IS Pakety#Hold timer|Hold timer]]
		- Pokud přidáváme zařízení do IS-IS sítě, nastavujeme tento bit, dokud neověříme funkčnost a plnou konvergenci
		- Pokud po restartování zařízení čeká na BGP peer, může narušit IS-IS LSDB nefunkční cestou, proto se tento bit nastavuje po dobu času, nebo do naskočení BGP peeru
			- [RFC 3277](https://datatracker.ietf.org/doc/html/rfc3277)
