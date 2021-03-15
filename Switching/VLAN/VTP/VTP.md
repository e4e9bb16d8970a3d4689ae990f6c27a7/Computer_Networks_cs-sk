[[VLAN]]
# VTP

Při vytvoření VLAN na jednom switchi musíme konfigurovat stejnou VLAN i na ostatních switchích, nebo použít VTP.
VTP je L2 protokol, který spravuje přidávání, mazání a přejmenovávání VLAN uvnitř VTP domény.
VTP doména je vytvořena více switchích, kteří mají stejné jméno domény a jsou v trunku.

## Módy

- Server
	- Spravuje seznam všech VLAN, ukládá je do NVRAM, může vytvářet a mazat VLANy.
	- Přímá a odesílá advertisements přes trunky ve VTP doméně
	- Defaultní mód
- Klient
	- Přímá konfiguraci ze serveru, udržuje lokální kopii VLAN
	- Přímá a odesílá advertisements přes trunky ve VTP doméně
- Transparentní
	- Neúčastní se VTP, pracuje samostatně, může vytvářet i mazat VLANy, ale pouze lokálně
	- Přímá a odesílá advertisements, ale nezveřejňuje své VLANy
	- Jediný mód, kde lze vytvořit Extended a Private VLAN.
- Off
	- Pouze VTPv3
	- Zahazuje Advertisements

Konfigurace VTP se nenachází v running-config.

Server rozesílá (pouze přes trunky) VTP advertisements (oznámení) každých 5 minut nebo při změně v konfiguraci. 
Server udržuje konfigurační revizní číslo (configuration revision number), které při každé změně zvýší o jedna. Klient při synchronizaci porovnává svoje a přijaté číslo. VTP advertisements obsahuje managment domain, revision number, VTP version a známe VLANy a jejich parametry. 

## Advertisements
- Summary
	- Posílána ze serveru a klientů každých 5 minut, nebo při updatu
	- Obsahuje
		- Domain name
		- Revision number
		- Identity of last updater
		- Time stamp of the last update
		- MD5 sum spočítaný na základě VLAN Databáze a VTP hesla a počtu Subset messages, které mohou následovat
	- Neobsahuje VLAN databázi
		
- Subset
	- Posílána ze serveru a klientů při modifikaci VLAN databáze
	- Obsahuje celou VLAN databázi, jeden Subset Advertisement může obsahovat více VLAN, ale pokud je databáze velká, může být rozdělena do více zpráv
	
- Advertisement Request
	- Posílána ze serveru a klientů, jde o požadavek, aby sousední zařízení doplnili databázi
	- Může jít o požadavek o celou, nebo část databáze
	- Jsou posílány při restartu zařízení v Client nebo Server módu, nebo pokud Client dostane Summary Advertisement s vyšším revision number, než jeho
		
- Join
	- Posílána ze serveru a klientů každých 6 sekund, pokud je zapnutý VTP Prunning
	- Obsahuje bit pro každou VLAN ve VTP instanci, normal range, který určuje, zda je na onom zařízení aktivní, dle toho probíhá Pruning 

Obdobou je protokol Generic VLAN Registration Protocol - GVRP a jeho nástupce Multiple VLAN Registration Protocol - MVRP                                    

## VTP Pruning

Zabrání odesílání broadcast, multicast paketů na switche, kde není port pro danou VLAN.
Konfigurace na serveru ovlivní doménu.

```
SWITCH(config)#vtp pruning
```                                                                     

VLAN 1 je *pruning ineligible*, neuplatňuje se na ní, na ostatních je defaultne povolen - *pruning eligible*.