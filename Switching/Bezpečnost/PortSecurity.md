# Port security
---

Jedná se o metodu zabezpečení přístupu do ethernetové sítě, nastavuje bezpečnostní politiky na základě [[MAC]] adres.
Port nesmí být v [[DTP#Módy|dynamic]] módu. 
Defaultně je povolena jedna [[MAC]] adresa v dynamickém módu.

## Porušení pravidel

- Protect
	- Zahazuje komunikaci se špatnou [[MAC]] adresou, ale neposílá oznámení
- Restrict
	- Zahazuje komunikaci se špatnou [[MAC]] adresou
- Shutdown
	- Port přepne do Err-disabled módu

## Konfigurace

```
SW(config-if)#switchport port-security     \\ Zapnutí funkci
```
```
SW(config-if)#switchport port-security maximum <číslo>     \\ Nastavení maxima možných MAC adres
```
```
SW(config-if)#switchport port-security mac-address <MAC>     \\ Přiřazení pevné adresy
SW(config-if)#switchport port-security mac-address sticky     \\ Získání MAC adresy z prvního framu
SW(config-if)#switchport port-security mac-address forbiden <MAC>     \\ Nastavení zakázané MAC 
```
```
SW(config-if)#switchport port-security aging time <min>     \\ Nastavení vypršení dynamické adresy
```
```
SW(config-if)#switchport port-security violation <Protect|Restrict|Shutdown>
```
```
SW(config)#errdisabled recovery cause psecure-violation     \\ Nastavení pro PortSecurity
SW(config)#errdisabled recovery interval <sec>     \\ Nastavení dobu, po které se port zapne
```
```
SW#show port-security interface <int>
```