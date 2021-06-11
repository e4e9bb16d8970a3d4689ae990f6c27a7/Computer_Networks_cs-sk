# Network Time Protocol
---

V síti je důležité mít synchronizovaný čas z mnoha důvodů:
- Správa hesel, které se v intervalech obměňují
- Výměna šifrovacých klíčů
- Určování validity certifikátů
- Synchronizované logy

 Čas je důležité pravidelně aktualizovat a přenastavovat, každé zařízení si určuje vlastní čas na základě vnitřního oscilátoru a základní hodnoty, ten ovšem není 100% a časem se mohou časy mezi zařízeními pozměnit. 

[RFC 958](https://datatracker.ietf.org/doc/html/rfc958) představilo NTP ptorokol, který funguje na základě Klient/Server topologie.
Server na UPD portu 123 nabízí své informace o času, které si klient může pomocí NTP Querry zjistit, vzhledem k tomu, že se jedná o aplikaci, může fungovat napříč IP sítí, k informaci o aktuálním času, kterou odešle server, může být značný, až sekundový, rozdíl kvůli latenci. Z tohoto důvodu může být značně dlouhý čas, po který se bude zařízení synchronizovat, synchronizace na přesnost milisekundy může trvat i hodiny.

Architektura se, podobně jako u DNS, dělí na vrstvy (*stratum*), čím nižší číslo, tím přesnější hodiny teoreticky zařízení má, zařízení na *Stratum 1* je typicky připojeno k atomovým hodinám.
*Stratum 2* zařízení bere čas od *Stratum 1*, *Stratum 3* od *Stratum 2* a tak dále.

Při úspěšném nastavení NTP nakonfiguruje několi věcí:
- Synchronizaci Hardwarových a Softwarových hodin (při restartu)
- Frekvenci a preciznost hodin
- Dobu zapnutí NTP
- Základní, referenční, čas
- Časový offset (clock offset) a latency mezi klientem a serverem nižší vrstvy (root delay)
- Rozptyl hodin samotných (root dispersion) a latence mezi klientem a serverem 1 vrstvy (peer dispersion)
- NTP loopfilter
- Aktualizační interval a dobu od poslední aktualizace

Tyto informace lze zjistit pomocí `R1#show ntp status` nebo `R1#show ntp associations`.

Při nakonfigurování více serverů se bude používat pouze ten s nejnižší úrovní, ostatní jsou v záloze.

## Konfigurace

### Základní

```
R1(config)#ntp server <IP> [prefer] [source <IF>]     \\ Nastavení serveru, se kterým sesynchronizujeme čas
```

```
R2(config)#ntp master <STRATUM>     \\ Zapnutí serveru na tomto zařízení
```

## Peers

V případě více výstupních prvků, které například používají i nějaké [[FHRPs|FHRP]], je dobré synchronizovat nejenom čas k serveru, ale i mezi sebou, od toho slouží *peer* spojení, které zajistí, že si zařízení sesynchronizují čas mezi sebou, jednají tak zároveň, jako klient i server.

```
R1(config)#ntp peer <IP_R2>
R2(config)#ntp peer <IP_R1>
```

### Broadcast

V případě spousty prvků v síti si můžeme ulehčit konfiguraci a prostě informace broadcastovat, čas nebude tolik přesný, ale konfigurace je jednodušší.

Protože síť má latenci, je možné ji nastavit, ale vzhledem k tomu, že se jedná o broadcast a nedomlouvá se na ní, jako u normálního NTP, musí se nastavit manuálně.

```
R1(config)#ntp broadcastdelay <DELAY>     \\ Udává se v mikrosekundách
```

#### Server

```
R1(config-if)#ntp broadcast [destination <IP>]
```

V případě [[IPv6]] není broadcast, takže se nastavuje multicast.

```
R1(config-if)#ntp multicast [destination <IPv4/6>]
```

#### Klient

```
R2(config-if)#ntp broadcast client
```

```
R1(config-if)#ntp multicast client
```

### Autentizace

Protože správný čas je zejména z bezpečnostních důvodů klíčový, nemůžeme dovolit, aby, zejména v broadcastovém módu, mohl kdokoliv náš čas měnit.

```
R1(config)ntp authenticate     \\ Napnutí používání autentizace
R1(config)ntp authentication-key <NUMBER> md5 <PASSWD>     \\ Nastavení hesla
R1(config)ntp thrusted-key <KEY_RANGE>     \\ Povolení hesla
```

```
R1(config)#ntp server <IP> key <NUMBER>     \\ Použití hesla pro připojení
```

```
R1(config-if)#ntp broadcast [destination <IP>]
R1(config-if)#ntp broadcast key <NUMBER>
```

```
R2(config-if)#ntp broadcast client
R2(config-if)#ntp broadcast key <NUMBER>
```

### Source

NTP pakety používají IP adresu výchozího interfacu, tuto lze změnit, aby například jedna IP adresa `lo` byla použita pro NTP.

```
R1(config)#ntp source <IP>     \\ Nemusí fungovat
R1(config)#ntp source <IF>     \\ V tomto případě použije IP adresu na tomto interfacu
```

### Časová zóna

V případě, že máme zařízení ve více zónách, je zvykem neměnit čas a nechat defaultní UTC, v případě, že je máme pouze v jedné zóně, je lepší nastavit offset.

```
R1(config)#clock timezone <NAME> <OFFSET>
```

```
R1(config)#clock summer-time <NAME> date <DATUM_S> <MĚSÍC_S>     \\ Nastavení letního času
```