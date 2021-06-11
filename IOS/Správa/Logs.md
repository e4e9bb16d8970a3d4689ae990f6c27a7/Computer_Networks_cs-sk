# Logy
---

Syslog stejně jako na kterém koli systému je naprosto zásadní funkcí, která velmi ulehčuje throubleshooting.
Defaultně Cisco zařízení neukládají log do nevolatilní paměti, pouze je, v případě nastavení, zobrazují na výstup, terminál.
[Syslog](https://datatracker.ietf.org/doc/html/rfc5424) je jednoduchý protokol umožňující přeposílání informací ze zařízení na centrální prvek k dalšímu zpracování, tato funkce sice může otevřít vektory útoku, ale lokální prohlížení logů je úkon, který se na větších sítích nedá vykonávat.
Jeho předností je, že stojí někde mezi manuálním prohlížení logů na zařízení a plným nastavení [[SNMP]].
Syslog defaultně posílá logy ve formě *traps* na server na UDP:514.

Zprávy mají určitý formát:

```
<Pořadí> <Čas>: %<Druh (Facility)>-<Závažnost>-<MNEMONIC>:<Popis>
```

```
*Jun  5 12:17:45.562: %LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
```

Některé tyto informace lze pozměnit nebo s nimi dále pracovat, například lze změnit číslování zpráv:

```
R(config)#service timestamps <debug | log> <datetime | uptime> {localtime | msec | year | show-timezone}
```
```
R(config)#service sequence-numbers 
```

## Defaultní nastavení

|Funkce|Nastavení|
|:-:|:-:|
|Logging|Zapnuto|
|Úroveň závažnosti|7 (debuggning)|
|Velikost bufferu|4096|
|Velikost historie|1|
|Časové razítka|Vypnuto|
|Synchroní logování|Vypnuto|
|Druh (Facility)|Local7|

## Konfigurace
---

Při nastavování nejrůznějších výstupů se nastavuje *severity level*, úroveň závažnosti. Tato škála je od 0 do 7 s tím, že 0 je nejzávažnější, při nastavování úrovně se nastaví i všechny nižší, takže například při nastavení úrovně 5 se povolují i úrovně 0 - 4.

### Lokální

Toto nastavení vyhradí určité místo v nevolatilní paměti pro dlouhodobější ukládání logů.

```
R(config)#logging on     \\ Zapnutí/Vypnutí logování, defaultně zapnuto
R(config)#logging buffered <SIZE> <LEVEL> {xml}     \\ Nastavení velikosti bufferu a typů zpráv, které se do něj uloží
```

```
R(config)#logging rate-limit <SECONDS> <all | console> {except <LEVEL>}
```

Toto nastavení určuje maximální rychlost zobrazování zpráv, v případě vážnějšího problému se mohou logy stát téměř nečitelnými...

```
R(config)#logging userinfo     \\ Zapsání do logu při přihlášení do priv režimu
```

#### Monitor (SSH, Telnet)

```
R(config)#logging monitor <LEVEL>     \\ Určení levelu práv, které se zobrazují na VTY
```

#### Console

```
R(config)#logging console <LEVEL>     \\ Určení levelu práv, které se zobrazují na CON
```

#### SNMP

```
R(config)#logging snmp-trap <LEVEL>     \\ Určení levelu práv, které se zobrazují na SNMP
```

#### Trap

```
R(config)#logging trap <LEVEL>     \\ Určení levelu práv, které se zobrazují na syslogu
```

#### History

Jedná se o buffer pro SNMP a syslog traps, protože není zaručeno, že budou doručeny, lokální zařízení si uchovává jejich historii.

```
R(config)#logging history <LEVEL>     \\ Určení levelu práv, které se ukládají
R(config)#logging history size <MSG_N>     \\ Určení počtu zpráv
R#show logging history
```

### Příkazy

Přes toto nastavení lze zapnout logování zadaných příkazů a jejich archivování.

```
R(config)#archive
R(config-archive)#log config
R(config-archive-log)#logging enable     \\ Zapne logování na lokální uložiště
R(config-archive-log)#logging size <MSG_N>     \\ Změní velikost uložistě
```

```
R(config-archive-log)#hidekeys     \\ Nahradí hesla v příkazech *
```

```
R(config-archive-log)#notify syslog     \\ Přepošle příkazy na syslog server
```

### Syslog

Používání Syslog protokolu, který umožňuje přeposílání logů na centrální prvek v síti.

```
R(config)#logging origin-id <ip | string | hostname>     \\ Nastavení identifikace zařízení
R(config)#logging host <IP> {transport <udp | tcp>} {xml}     \\ nastavení serveru, kam se posílají logy
R(config)#logging trap <LEVEL>     \\ nastavení od kdy se logy posílají
```
