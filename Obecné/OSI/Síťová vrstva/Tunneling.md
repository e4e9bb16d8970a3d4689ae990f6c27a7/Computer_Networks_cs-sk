# GRE
---
## Popis

Pokud požadujeme IPv6 Stack a ISP nebo Core podporuje pouze IPv4 Stack, máme několik možností jak zajistit funkčnost IPv6 přes tuto zónu.

IPv6 pakety jsou zapouzdřeny do GRE tunelu a poslány pomocí IPv4, na druhé straně tunelu je přijímač, který tyto pakety de-encapsuluje a nadále funguje s IPv6 Stackem.

## Problémy

Jedním z problémů je nutnost vytvořit tunel s každým zařízením, pokud jich je v síti více.
Proto je doporučeno používat tento tunel v případě přemostění ISP do nějaké výchozí brány, která se následně bude starat o směrování.

## Konfigurace

### R1

```
R1(config)#interface tunnel <Číslo>
R1(config-if)#tunnel source <IF>     \\ Může jít přímo o Gigabit, ale lepší je použít Loopback
R1(config-if)#tunnel destination <IP_R2>     \\ R1 musí mít IPv4 spojení s R2
R1(config-if)#ipv6 address <IPv6_adresa>     \\ Může jít o libovolnou IPv6 adresu
R1(config)#ipv6 route <IPv6_Net/Prefix> tunnel <Číslo>     \\ Nutno nastavit routing
```

### R2

```
R2(config)#interface tunnel <Číslo>
R2(config-if)#tunnel source <IF>     \\ Může jít přímo o Gigabit, ale lepší je použít Loopback
R2(config-if)#tunnel destination <IP_R1>     \\ R1 musí mít IPv4 spojení s R1
R2(config-if)#ipv6 address <IPv6_adresa>     \\ Může jít o libovolnou IPv6 adresu
R2(config)#ipv6 route <IPv6_Net/Prefix> tunnel <Číslo>     \\ Nutno nastavit routing
```

# 6to4
---
## Popis

Pokud požadujeme IPv6 Stack a ISP nebo Core podporuje pouze IPv4 Stack, máme několik možností jak zajistit funkčnost IPv6 přes tuto zónu.

IPv6 pakety jsou poslány jako nad-vrstva IPv4 paketu pomocí protokolu 41.

Hlavní výhodou oproti GRE tunelu je automatické routování, nemusí se vytvářet full-mesh tunelů mezi zařízeními.

Příchozí zapouzdřený paket se na cílovém zařízení de-encapsuluje a dále funguje jako IPv6.

## IPv4-in-IPv6

Důvod, proč tento tunel funguje automaticky, je ten, že IPv4 cílová adresa je obsažena přímo v IPv6 adrese, pro tyto možnosti se vyhradil rozsah `2002∷/16`.

<b><span style="color:#31B404">`2002:`</span></b> <b><span style="color:#FF8000">`BEA8:A0A`</span></b> `::/128`
- <span style="color:#31B404">`2002`</span> - Vyhrazený prostor pro 6to4 překlad
- <span style="color:#FF8000">`BEA8:A0A`</span> - Hexa přepis IPv4 adresy (`192.168.10.10`)

Nad takto postavenými adresami se provádí IPv4 routing.
Touto technikou lze routovat i Global Unicast nebo Unique Local adresy, jako next-hop se nastaví adresa z rozsahu  2002∷/16, na který je pomocí 6to4 konektivita.

## Konfigurace

### R1

```
R1(config)#interface tunnel <Číslo>
R1(config-if)#tunnel source <IF>
R1(config-if)#ipv6 address <6to4_Adresa>
R1(config-if)#tunnel mode ipv6ip 6to4     \\ Nastavení tunelu na 6to4 (překlad)
R1(config)#ipv6 route <IPv6_Net/Prefix> tunnel <Číslo>     \\ Nutno nastavit routing
```

### R2

```
R2(config)#interface tunnel <Číslo>
R2(config-if)#tunnel source <IF>
R2(config-if)#ipv6 address <6to4_Adresa>
R2(config-if)#tunnel mode ipv6ip 6to4     \\ Nastavení tunelu na 6to4 (překlad)
R2(config)#ipv6 route <IPv6_Net/Prefix> tunnel <Číslo>     \\ Nutno nastavit routing
```
