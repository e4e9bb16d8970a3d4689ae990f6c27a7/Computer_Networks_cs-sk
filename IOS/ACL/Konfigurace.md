[[ACL]]
# IP ACL

## Standard
---
### Pravidlo pro jedno zařízení

```
R1(config)#ip access-list standard <Číslo|Jméno>    \\ Přepnutí se do Access Listu
R1(config-std-nacl)#<Permit|Deny> <Host> <IP>    \\ Povolení/Zablokování pro specifického hosta
```

```
R1(config-if)#ip access-group <Číslo|Jméno> <In|Out>    \\ Přiřazení ACL na Interface, na kterém funguje
```

### Pravidlo pro více zařízení

```
R1(config)#ip access-list standard <Číslo|Jméno>    \\ Přepnutí se do Access Listu
R1(config-std-nacl)#<Permit|Deny> <IP> <Wildcard_Mask>    \\ Povolení/Zablokování pro rozsah
```

```
R1(config-if)#ip access-group <Číslo|Jméno> <In|Out>    \\ Přiřazení ACL na Interface, na kterém funguje
```

### Přepsání

```
R1(config)#ip access-list standard <Číslo|Jméno>    \\ Přepnutí se do Access Listu
R1(config-std-nacl)#<Číslo_řádku> <Náhradní_pravidlo>    \\ Přepsání pravidla novým
```

#### Smazání

```
R1(config)#ip access-list standard <Číslo|Jméno>    \\ Přepnutí se do Access Listu
R1(config-std-nacl)#no <Číslo_řádku|Pravidlo>    \\ Smazání řadku, nebo pravidla
```

### Komentáře

```
R1(config)#ip access-list standard <Číslo|Jméno>     \\ Přepnutí se do Access Listu
R1(config-std-nacl)#remark <Číslo_řádku>     \\ Přidání komentáře
```

## Extended
---

```
R1(config)#ip access-list extended <Číslo|Jméno>
R1(config-ext-nacl)#<Permit|Deny> <Protokol> <SRC_Scope> <SRC_Port(s)> <DST_IP> <DST_Protocol(s)>
```

```
R1(config-ext-nacl)#permit ip any any    \\ Povolení veškerého provozu
```

Ostatní úkony jsou shodné se Standard ACL.

## Named
---
```
R1(config)#ip access-list <standard|extended> <Jméno>
```