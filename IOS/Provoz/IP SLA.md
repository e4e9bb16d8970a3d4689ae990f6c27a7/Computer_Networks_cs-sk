# IP SLA
---
*Cisco IOS Ip Service Level Agreement*/*Service Assurance Agent (SAA)*/*Response Time Reporter (RTR)*

Tato služba má za úkol poskytovat informace o síti, zejména její stabilitě a schopnostech.
Je založená na posílání dat od zdroje (source) jinému zařízení, responder. Tak umožňuje měřit kvalitu sítě ve speciálních situacích, například se specifickým QoS provozem.

Umožňuje měřit:

- Delay
- Jitter
- Packet loss
- Packet sequencing
- Hop count
- Konektivitu protokolů (ICMP, UDP, TCP)
- Download time
- Voice-quality metrics (MOS)

Základní na stavení je v několika krocích:

- Nastavení samotného SLA
- Nastavení hraničních hodnot
- Nastavení responderu
- Schedule SLA
- Práce s výsledky

## Konfigurace

```
R(config)#ip sla <NUMBER> 
R(config-ip-sla)#<FUNKCE> <RESPONDER_IP>     \\ Nastavení funkce a destinace, například DHCP, DNS, ICMP...
R(config-ip-sla-<FUNKCE>)#frequency <SECONDS>     \\ Nastavení opakování
```
```
R(config-ip-sla-<FUNKCE>)#threshold <MSEC>     \\ Nastavení hraniční hodnoty
```

```
R(config)#ip sla schedule <NUMBER> start-time <now | time> life <seconds | forever> {ageout} {recurring}     \\ Nastavení spuštění
```

```
R#show ip sla summary     \\ Zobrazení všech SLAs
```

V případě, že konfigurujeme nějakou specifickou komunikaci, která není na zařízení defaultně podporována (ICMP Echo), musíme na responderu nastavit funkci responderu.

```
R2(config)#ip sla responder     \\ Zapne odpovídání na všechnu SLA komunikaci
```

```
R2(config)#ip sla responder <tcp-connect | udp-echo> ip address <IP> port <PORT>     \\ Nastavení pro konkrétní socket
```

### Autentizace

```
R(config)#key chain <NAME>
R(config-keychain)#key <NUMBER>
R(config-keychain-key)#keystring <PASSWD>
R(config)#ip sla key-chain <NAME>
```

### Práce s výsledky

Práce s výdledky může být čistě informativní, ale častěji je lepší využít Object-Tracking:

```
R(config)#track <NUMBER> ip sla <NUMBER> state     \\ Provázání Tracku s SLA
```

Na starších IOS může být SLA v Object-Trackingu označeno pod *RTR*.