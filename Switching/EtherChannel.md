# EtherChannel
---

*EtherChannel* nebo také *Link Aggregation* (*LAG*), *Port-channel* je rozšířenou metodou pro spojování více fyzických spojů do jednoho logického a zvýšit tag propustnost linky.
Po vytvoření se jevý pro protokoly typu [[STP]], ale i normální provoz jako normální linka. 

Lze ho vytvořit až z 8 linek, tato hodnota je vybrána z důvodu toho, že Ethernetové ryhlosti jsou většinou v násobcách deseti, pokud bychom tedy potřebovaly rychlost větší, než `8 Gb/s`, je výhodnější použít `10 Gb/s` spoj.
V případě vypadnutí některé z linek EtherChannel jako takový nadále funguje, ale ztratil jednu linky a tím i její rychlost, což se projevý ve výsledné propustnosti, ale i ceně linky v [[STP]].

## Load Sharing Algorithm

Vzhledem k tomu, že oproti [[PPP#Multilink|Multilinku]] nerozkládá framy na jednotlivé části pro každou linku nebo oproti blokovaným linkám per-vlan ve [[STP]] využívá algoritmus pro rozesílání komunikace mezi fyzické spoje, jedná se o *load-sharing* a ne *load-balancing*. 

Switch vytvoří hash na základě informací L2, L3, L4 vrstev, nejčastěji na základě [[MAC]] adres, který poté přiřadí jednomu z portů v EtherChanelu, v případě, že se hash sestavuje nad více informacemi, například zdrojové a cílové [[MAC]] adrese, pak se nad těmito poly provede XOR operace nad jejímž výsledkem se vypočítá hash.
Pro jedno koncové zařízení se tedy nejedná o zvýšení rychlosti, pokud máme EtherChanel sestaven z 6x `1Gb/s` linek, výsledná maximální propustnost bude stále pouze `1 Gb/s`, ale už se o něj nebude muset dělit s 6 dalšímy zařízeními.
Zároveň se po stejných linkách rozesílá pouze *flow*, což je nutné, aby nedošlo z narušení pořadí framů, ale má to i nevýhody, například pokud sestavujeme hash na základě cílové [[MAC]] adresy a 90% provozu je s MAC adresu Default Gateway, poté bude EtherChannel témeř bez utilizace.

Na mnoha platformách hashovací funkce vrací 3-bitový výsledek od 0 do 7, čímž čísluje výstupní port, z toho lze usoudit, že procento provozu pro jeden port při použití maximálního počtu, tedy 8 linek v EtherChannelu je 12.5%, na tomto základě se rozděluje poměr provozu:

|Počet portů|Poměr (1 = 12.5%)|
|:-:|:-:|
|8|1:1:1:1:1:1:1:1|
|7|2:1:1:1:1:1:1|
|6|2:2:1:1:1:1|
|5|2:2:2:1:1|
|4|2:2:2:2|
|3|3:3:2|
|2|4:4|
|1|8|

Na jiných Cisco platformách se hodnota rozšiřuje ze 3-bitové na 8-bitovou, což umožňuje 256 hodnot a zajišťuje tak mnohem spravedlivější load-sharing vzhledem k tomu, že v takovém případě se 1 = 0.390625%.

## Nastavení
---

Pro správnou konfiguraci je nutné dodržet několik předpokladů:

- Stejná rychlost a duplex linek
- Stejný typ a nastavení linek (Access, Trunk, povolené VLANy)
- Stejný STP port cost
- Žádné SPAN

Při přidání interfacu do Port-channelu je automaticky vytvořen logický interface Port-channel, který zdědí nastavení prvního fyzického interface, tedy toho, nad kterým byl vytvořen, nebo který do něho byl přidán jako první, zároveň Port-channel posílá framy pod MAC adresou tohoto interafacu.
Všechny ostatní interface, které se přidají do Port-chanelu, budou porovnávány s tímto nastavením, pokud se nějaké liší, stále budou přidány, ale budou fungovat v *suspended* módu, což znamená, že mezi ně nebude prováděn load-sharing a nebude se na ně aplikovat nastavení, které použijete na Port-channelu, to se uplatní pouze na non-suspended fyzické porty.

Port-channel může být, jak L2, tak L3, toto nastavení záleží na tom, zda je první port `switchport`, nebo `no switchport` (routed port).

Dále je potřeba si dát pozor na funkčnost mezi switchi, pokud jeden ze switchů nerozeznává Port-chanel a chová se k linkám, jako ke 2 spojům, může se vytvořil smyčka.
Smyčkám by měla předejít funkce [[STP Funkce#Dispute|Dispute]] a Cisco implementovalo novou funkci jménem *EtherChannel Misconfig Guard*, ten pracuje na presmise, že pokud jsou [[STP Terminologie#BPDUs|BPDUs]] posílána z Port-channelu, mají jednu MAC adresu, pokud nejsou posílána z Port-channelu, objeví se [[STP Terminologie#BPDUs|BPDUs]] s různými MAC adresami a celý Port-channel se přepne do *err-disabled* stavu. Funkce je defaultně zapnuta, ale lze ji vypnout pomocí `SW(config)#no spanning-tree etherchannel guard misconfig`.
Tato funkce ale není všespásná a jsou případy, kdy se vytvoří smyčka.

Je proto doporučeno, aby se použil některý z dynamických protokolů.

### Protokoly

Základní rozdělení je na statické nastavení a dynamický protokol, je doporučeno používat dynamický protokol, protože poskytuje informace o stavu linek v Port-channelu, zejména u optických spojů, u kterých může selhat mezilehlé optické zařízení nebo spoj. V případě Ethernetu se vadná linka vyřadí z aktivního používání.

#### Dynamické
---

##### LACP

IEEE 802.1AX Link Aggregation Control Protocol je otevřený standart protokolu, který každých 30s posílá keep-alive framy pro udržování konektivity na MAC `0180:C200:0002`.
Podporuje až 16 portů, ale všechny porty nad 8 jsou náhradní v takzvaném *Hot-Standby* (H) módu. Výběr portů, které jsou náhradní a které ostatní je na superiorním switchi, to určuje priorita a MAC adresa, stejný princip jako u [[STP Terminologie#Bridge ID|STP BID]], ten následně rozhodne na základě priority a ID portu, stejně jako u [[STP]].

- Passive
  - Neinicuje vytvoření Port-channelu 
  - Neposílá framy
- Active
  - Iniciuje vytvoření Port-channelu
  - Posílá framy
  - Může vytvořit Port-channel pouze, pokud je na druhé straně také LACP

```
SW(config-if-range)#channel-group <1-255> mode {active|passive}     \\ Vytvoření Port-channelu
```

###### LACP Fast

Defaultně se LACP frame posílá každých 30s a po 3 nedostaných framech se linka odpojí, což může znamenat až 90s okno nedostupnoti sítě.
Tento problém řeší nastavení `SW(config-if)#lacp rate fast`, které stáhne hello timer na 1s.

###### Minimální počet linek

Defaultně se Port-channel vytvoří potom, co se úspěšně vytvoří LACP sousedství alespoň na jedné z linek, toto číslo lze změnit.

```
SW(config)#interface port-channel <INT>
SW(config-if)#port-channel min-links <MIN>
```

###### Maximální počet linek

Maximální počet linek je 16 s tím, že všechny nad 8 jsou v Hot-standby módu, číslo, nad které jsou linky v Hot-standby módu a jsou tedy záložní lze změnit z defaultních 8.

```
SW(config)#interface port-channel <INT>
SW(config-if)#lacp max-bundle <MAX>
```

###### System priority

Koncept priority je shodný s [[STP]], rozhoduje, kdo v případě selhání linky a tím pádem výběru nové linky z Hot-standby módu, rozhodne.

```
SW(config)#lacp system-priority <PR>
```

###### Port priority

Priorita portu rozhoduje o přednosti převedení portu z Hot-standby módu na funkční.

```
SW(config-if)#lacp port-priority <PR>
```

##### PAgP

Cisco proptietární protokol, který posílá frmay na MAC adresu `0100:0CCC:CCCC`.
Podporuje maximálne 8 portů.

- Auto
  - Neinicuje vytvoření Port-channelu 
  - Neposílá framy
- Desirable
    - Iniciuje vytvoření Port-channelu
  - Posílá framy každých 30s
  - Může vytvořit Port-channel pouze, pokud je na druhé straně také PAgP

PAgP defaultně operuje v *silent* módu, což mu umožňuje vytvořit Port-channel i se zařízením, které nepodporuje PAgP.

```
SW(config-if-range)#channel-group <1-255> mode {desirable|auto} [non-silent]     \\ Vytvoření Port-channelu
```

#### Manuální
---

Manuální vytvoří Port-channel pouze na základě informací na lokálním switchi, nemá žádnou kontrolu nad správností konfigurace mezi switchi, ale funguje vždy.

```
SW(config-if-range)#channel-group <1-255> mode on     \\ Vytvoření Port-channelu
```