# SNMP
---

*Simple Network Management Protocol* / *Internet Standart Management Framework*

Jedná se o protokol umožňující zprávu síťových zařízení.
Funguje na základě periodického posílání zpráv od agenta k manageru, respektive Manager si je vyžádá, obsahujících řadu informací o zařízení, na základě kterých se v případě mnoha programů vytváří vizualizace stavu sítě, velmi jednoduše se tak zjišťují změny, chyby nebo zatížení sítě.

Topologie se skládá z ***SNMP Agent***, které posílají informace, typicky router, switch, a ***SNMP Manager***, který sbírá data a nějak je zpracovává, typickým způsobem zpracování mohou být produkty [PRTG Network Monitor](https://www.paessler.com/prtg) nebo [Solarwinds](https://www.solarwinds.com/) souhrně nazývané jako NMSs(Network Management Systems).

Jednotlivé verze SNMP jsou definovány několika oblastmi:

## SMI

*Structure of Management Information*

Jedná se o konvence struktury přeposílaných dat, aby s nimi mohli pracovat nejrůznější NMSs. 

## MIB

*Management Information Base*

Jedná se o databázi informací o objektech ***Object Identifiers*** (OIDs), které mohou být ze zařízení získány, typicky třeba stavy interfaců, zatížení CPU ...
MIB je strukturována hierarchicky s kořenem **iso**, takže IOD připomíná IPv4 adresu (1.3.6.1.2.1).
Jsou standardizovány mezi verzemi se spojením s SMI, ale i vendor-specific.

- [RFC 1156]() - MIB-1
- [RFC 1213]() - MIB-2 
- Poté IETF přestala vyvíjet standartizované MIBs, místo toho to přenechala výrobcům, aby si je upravili specificky pro jejich zařízení

Za zmínku stojí Remote Monitoring MIB (RMON), což je standardizovaná MIB, která pomocí SET zpráv umožňuje nastavit monitorování provozu a odpovídání na určité zatížení sítě.

Cisco OID lze zjistit na [této stránce](https://snmp.cloudapps.cisco.com/Support/SNMP/do/BrowseOID.do).

## Zprávy

SNMP používá UDP port 161 (162 pro Trap) a 10161 (10162 pro Trap) pro Secure SNMP, ověřování zpráv probíhá na úrovni L7, na zprávy se většinou odpovídá pomocí Request zprávy, krom Trap.

|Zpráva|Verze|Odpověď|Posíláno od|Účel|
|:-:|:-:|:-:|:-:|:-:|
|Get|1|Response|Manager|Požadavek o jednu hodnotu|
|GetNext|1|Response|Manager|Požadavek o další proměnou v MIB databázi|
|GetBulk|2|Response|Manager|Požadavek o více hodnot, vylepšená verze GetNext, hodí se pro zjišťování komplexních struktur, jako Routing Table|
|Response|1|None|Agent|Jedná se o odpověď obsahující samotné informace pro SET a GET zprávy|
|Set|1|Response|Manager|Zpráva, která říká Agentovy, aby nastavil proměnou na specifickou hodnotu|
|Trap|1|None|Agent|Jedná se o nevyžádanou zprávu s informacemi, typicky se nastaví pro odeslání při vypnutí zařízení nebo interfacu|
|Inform|2|Response|Manager|Posílá se mezi Managery pro výměnu MIB dat|

## Bezpečnost

### v1 + v2c

V těchto verzích je bezpečnost řešena pouze jako autentizace, v1 má zabudovanou a v2c přidává podporu do v2 pro takzvané komunity, jedná se v podstatě o heslo k ověření Managera.

### v3

v3 podporuje krom autentizace jako takové, která je již řešena pomocí MD5 hesla, taky zprávy, které jsou posílány v šifrované podobě pomocí, nejčastěji, DES/AES a jejich integrita je zabezpečena pomocí SHA hashe.

## Verze
---

|Verze|Popis|
|:-:|:-:|
|1|Používá SMIv1 a MIB-1 a autentizaci pomocí komunit|
|2|Používá SMIv2 a MIB-2, odebralo nutnost používání komunit, přidalo GetBulk a Inform zprávy|
|2c|Jedná se o rozšíření v2, povoluje funkci komunit z v1|
|3|Oproti v2c přidává robustnější zabezpečení|

## Konfigurace
---

```
R(config)#no snmp-server     \\ Vypnutí SNMP
```

### Informace o zařízení

Pro jednoduchou identifikaci zařízení na Manageru je nutné poskytnout nějaké informace o zařízení.

```
R(config-if)#snmp-server contact <JMÉNO>     \\ Nastavení jména, například administrátora
R(config-if)#snmp-server location <MÍSTO>     \\ Nastavení lokace, například 2 budova 3 patro...
R(config-if)#snmp-server chassis-id <POPIS>     \\ Nastavení jména konkrétního zařízení
```

```
R(config)#snmp-server system-shutdown     \\ Zapnutí možnosti vypnout zařízení pomocí SNMP
```

### v2c konfigurace
---

```
R(config-if)#snmp-server community <PASSWD> <ro | rw> {acl}    \\ Nastavení hesla a zapnutí SNMP v2c
```

#### Traps

```
R(config-if)#snmp-server host <IP> <traps | inform> version <2c> <PASSWD> {TRAP}    \\ Nastavení traps a příjemce
R(config-if)#snmp-server enable traps <TRAP>     \\ Povolení dalších traps
```

```
R(config-if)#snmp-server enable traps snmp linkdown linkup coldstart warmstart authentication
```

#### Filtrování

Defaultně si může manager zjistit všechny informace, ale my můžeme nastavit, aby měl přístup pouze ke specifickým OID.

```
R(config-if)#snmp-server view <JMÉNO> <MIB_OID/NAME> <include | exclude>     \\ Nastavení pravidla
R(config-if)#snmp-server community <PASSWD> <ro | rw> view <VIEW>     \\ Přiřazení pravidla komunitě
```

### v3 konfigurace
---

#### Základní konfigurace

```
R(config-if)#snmp-server group <GROUP> v3 <auth | noauth | priv>     \\ Vytvoření skupiny
```

- `auth`
	- Používá se pouze autentizace, ale nešifrují se zprávy
- `noauth`
	- Nepoužívá se ani autentizace
- `priv`
	- Používá se autentizace a zprávy se šifrují

```
R(config-if)#snmp-server user <JMÉNO> <GROUP> v3 auth <ALG> <PASSWD> priv <ALG> <SHARED_KEY>
```

Vzhledem k tomu, že autentizace a bezpečnost obecně je halvní vylepšení v3 oproti v2c, další nastavení je shodné.

#### Filtrování

Defaultně si může manager zjistit všechny informace, ale my můžeme nastavit, aby měl přístup pouze ke specifickým OID.

```
R(config-if)#snmp-server view <JMÉNO> <MIB_OID> <include | exclude>
```

```
R(config-if)#snmp-server group <GROUP> v3 priv read <VIEW> write <VIEW>     \\ Vytvoření skupiny
```

```
R(config-if)#snmp-server user <JMÉNO> <GROUP> v3 auth <ALG> <PASSWD> priv <ALG> <SHARED_KEY>
```
