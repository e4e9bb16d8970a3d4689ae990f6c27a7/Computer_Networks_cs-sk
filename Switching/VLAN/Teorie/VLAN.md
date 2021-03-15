# VLAN

| VLAN | Popis |
| :------: | :---: |
| 0 a 4095 | Rezervované pro systémové použití |
| 1 | Defaultní *VLAN*, nelze smazat, defaultně na všech portech |
|2 - 1001 | *Standard* rozsah |
| 1002 - 1005 | Speciální *VLAN*y pro *Token Ring* a *FDDI* |
| 1006 - 4094 | *Extended* rozsah |

## Použití

### Oddělení sítě

Klasické oddělení sítě, pomocí subnetu, na *Hubu* (*L1*) a *Switchi* (*L2*) selhává, protože tyto prvky se nedívají na IP adresaci a rámce prostě posílají na jiné porty, je tak možno tuto komunikaci odchytávat.
Použitím *VLAN* se komunikace bude posílat pouze na porty zařazené do vlastní *VLANy*, ne na všechny porty.

Pro správné fungování *VLANy* je nezbytné i oddělení pomocí subnetů, například *routing*.

### Praktické výhody

#### Snížení broadcastů
- Hlavní výhodou je vytvoření více menších broadcastových domén, což vede ke snížení trafficu

#### Zjednodušená správa
- K přesunu zařízení do jiné sítě stačí překonfigurovat *VLAN*, nikoliv fyzické připojení

#### Bezpečnost
- Dnes se používá řada provozů, které jsou kritické a nemusí mít přístup do celé sítě a naopak, například IP telefonie a komunikace mezi *AP*. Navíc *VLAN* podporují *QoS*. 

#### Snížení HW
- Díky tomu, že různé sítě mohou být na jednou switchi, nepotřebujeme například kupovat switch pro každé patro.

## Přiřazování do VLAN

### Dle portu

- Port switche je napevno nakonfigurován do určité VLANy
- Veškerá komunikace přicházející na tento port automaticky spadá do této VLAN, celý switch připojen na tento port bude spadat do této VLANy
- Nejrychlejší a nejpoužívanější řešení

### Dle MAC adresy

- Port se zařadí sám do VLANy podle zdrojové MAC adresy, musíme tedy spravovat tabulku MAC adres pro každé zařízení spolu s VLANou
- Výhodou je, že se jedná o dynamické řešení, nezáleží na portu, ale MAC adrese, ale switch musí vyhledávat v tabulce MAC adres
#### Provedení

1. Port si nastaví VLAN podle prvního framu, který na něho přijde a zůstane mu, dokud se nevypne
2. Switch zařazuje rámce samostatně dle MAC adresy, toto je velmi náročné na výkon
3. VLAN Membership Policy Server (VMPS)
  - Cisco řešení, speciální server, který spravuje tabulky MAC adres
  - Port se přiřazuje do VLANy, takže všechny zařízení (max. 20) musí být ve stejné VLAN

### Dle protokolu

- Může například oddělit IP provoz od AppleTalk, normální provoz od Voice

### Dle autentizace

- Ověří se uživatel nebo zařízení pomocí protokolu IEEE 802.1x [^1] a podle informací se automaticky umístí do VLANy
- Je to primárně bezpečnostní metoda, které řídí přístup do sítě (NAC), ale po rozšíření slouží i pro VLANy
- RADIUS server, který ověřuje identitu uživatele, obsahuje také mapovaní uživatelů na VLANy a tuto informaci zašle po úspěšné autentizaci
- U této metody je možné nastavení, že v případě, kdy není uživatel autentizován, tak je zařazen do speciální hostovské VLANy
- U Cisco switchů může být port single-host, kdy je možno připojit pouze jedno zařízení nebo multiple-host, kdy sice může být do portu připojeno více zařízení, ale ve chvíli, kdy se první autentizuje, tak je port autentizovaný (a zařazený do VLANy) a komunikovat mohou všechna zařízení

[^1]: https://en.wikipedia.org/wiki/IEEE_802.1X
















































































