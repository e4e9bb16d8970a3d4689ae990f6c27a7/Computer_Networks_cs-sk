[[VTP]]
# VTPv3 vs VTPv2

- Změra sole serveru
  - Primary
    - Může upravovat VLAN databázi
    - Může být maximálně 1 ve VTP instanci
  - Secondary
    - Nemůže upravovat VLAN databázi
    - Může jich být více
    - Může být povýšen do role Primary, jako jediný
  - Specifikum, který switch je Primary server není uloženo v konfiguraci, ale jedná se o runtime konfiguraci, po restartování dojde k přepnutí do sekundárního módu
      
- Bezpečnost
	- Heslo může být uloženo v encrypted formě, ze které ho již není možné zobrazit zpět v plain textu
	- Tato encrypted forma textu je použita pro Advertisements, ale pokud chceme povýšit Secondary server na Primary, musíme opět zadat plain text heslo
	- Zároveň, switch příjme update pouze v případě, že se odesílatelem shoduje na MAC adrese primárního serveru
		
- Full Range VLAN
	- VTPv3 je schopno přenášet Full Range VLAN i Private VLAN
	- VTP Prunning se ale vztahuje pouze na normal-range VLANy
		
- Módy
	- Kromě Transparent módu, který přeposílá VTP je možné zadat Off mode, který je zahazuje
	- Toto je možné dělat na bázi jednotlivých Trunků, takže je možné mít v síti více VTP
	- Primary server může jako jediný upravovat VLAN databázi, stejně jako u VTPv2, ale u VTPv3 i pokud se switch v jiném módu odpojí z instance VTP nemůže svou databázi upravit 
		
- Více možností
	- VTPv3 může synchronizovat nastavení více funkcí, než jenom VLAN, napříkald i [[MSTP]].

## Další

Nadále není možné resetovat revision number pouze přepnutím do transparent módu a zpět, nyní je nutné změnit Domain name nebo VTP Password.

Pokud je v síti zařízení, které pracuje pouze jako VTPv2, zařízení s VTPv3 se na jeho portu přepne do fungování jako VTPv2 a vše funguje normálně, toto není podporováno u VTPv1.

Také je zjednodušeno ukládání VLAN databáze, u VTPv3 je pouze v souboru vlan.dat, pouze v transparentním a off režimu je uložena i v running-config.
		
