# Embedded Event Manager (EEM)
---

Jedná se o Cisco Engine zabudovaný v operačním systému (IOS, IOS-XE, NX-OS...), který nabízí širokou možnost automatizace a reagování na stavy a události v zařízení.

Skládá se z 2 částí:

## Event Detector

Jedná se o rozhraní mezi EEM a různými systémovými stavy.
EEM má detektory pro spoustu funkcí a sleduje velkou plejádu jejich stavů.

Například: 

```R
R#show event manager detector all
No.  Name                Version   Node        Type    
1    application         01.00     node0/0     RP      
2    identity            01.00     node0/0     RP      
3    neighbor-discovery  01.00     node0/0     RP      
4    rf                  01.00     node0/0     RP      
5    msp                 03.00     node0/0     RP      
6    routing             02.00     node0/0     RP      
7    syslog              01.00     node0/0     RP      
8    generic             01.00     node0/0     RP      
9    nhrp                01.00     node0/0     RP      
10   track               01.00     node0/0     RP      
11   resource            01.00     node0/0     RP      
12   cli                 01.00     node0/0     RP      
13   counter             01.00     node0/0     RP      
14   interface           01.00     node0/0     RP      
15   ioswdsysmon         01.00     node0/0     RP      
16   none                01.00     node0/0     RP      
17   oir                 01.00     node0/0     RP      
18   snmp                01.00     node0/0     RP      
19   snmp-object         01.00     node0/0     RP      
20   ipsla               01.00     node0/0     RP      
21   snmp-notification   01.00     node0/0     RP      
22   timer               01.00     node0/0     RP      
23   test                01.00     node0/0     RP      
24   config              01.00     node0/0     RP      
25   env                 01.00     node0/0     RP      
26   ds                  01.00     node0/0     RP      
27   crash               01.00     node0/0     RP      
28   nf                  01.00     node0/0     RP      
29   rpc                 01.00     node0/0     RP
```

## Policy Director

V případě vytvoření Appletu, Policy Director hlídá stavy sledovaných objektů a v případě, že se splní podmínka, kterou applet zavedl, provede se naprogramovaná akce.

### Applets

Jsou postaveny jako programovací jazyky na *if-then* stavech:

```
R(config)#event manager applet <NAME>     \\ Vytvoření Appletu
R(config-applet)#event <DETECTOR> <...>     \\ Vytvoření podmínky if, vybírá se detektor a z něj následně stav
R(config-applet)#action <NUMBER> <CMD>     \\ Určení, co se má stát při splnění podmínky
```

### [[TCL]]

Často v případě, že chceme reagovat na změnu v zařízení, nebo i v síti, potřebujeme změnit nastavení zařízení, toto nastavení může být pomocí `action` klauzule zdlouhavé a nemusí nabízet všechny možnosti, které potřebujeme, v takovém případě lze využít TCL scriptu.
Například pokud pomocí [[IP SLA]] zjistime, že nám vypadlo [[DHCP]] nebo je nespolehlivé, můžeme vytvořit script, který vytvoří provizorní DHCP na routeru, notifikuje administrátora e-mailem a [[SNMP]] a nastaví nové EEM a IP SLA pro případ opětovného naběhnutí DHCP.

```
R(config)#event manager applet <NAME>     \\ Vytvoření Appletu
R(config-applet)#event <DETECTOR> <...>     \\ Vytvoření podmínky if, vybírá se detektor a z něj následně stav
R(config-applet)#action <NUMBER> cli command <"tclsh <PATH>">     \\ Spuštění scriptu
```