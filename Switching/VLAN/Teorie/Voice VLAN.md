[[VLAN]]
# Voice VLAN

Pro *VoIP* má Cisco řadu zjednodušení.

Jedním z nich je konfigurace, kdy je do portu připojen Cisco telefon (který obsahuje malý 3-portový switch) a za ním je připojeno PC. 
Na portu nastavíme access *VLAN*, do ní spadá komunikace *PC*, a také voice *VLAN* (někde označována jako auxiliary *VLAN* - má i více použití), do které se zařadí komunikace telefonu.
Aby vše fungovalo jak má, tak musíme použít *Cisco IP telefon* a na portu musí být povoleno *CDP*.
Ve skutečnosti vše funguje tak, že se na portu nastaví trunk, access *VLAN* se stane native *VLAN* (tedy netagovaná) a komunikace telefonu použije *802.1q*.
V případě nastavení Voice VLANy, switch automaticky nastaví port jako [[STP Funkce#STP Portfast|PortFast]].
## Konfigurace

```
SW1(config)#switchport mode access <VLAN ID>     \\ Nastavení access VLAN pro PC
SW1(config)#switchport voice vlan <VLAN ID>     \\ Nastavení VLAN pro telefon
```

nebo

```
SW1(config)#switchport trunk encapsulation <Encapsulace>     \\ Nastavení encapsulace pro Trunk
SW1(config)#switchport mode trunk
SW1(config)#switchport trunk native vlan <VLAN ID>     \\ Nastavené VLAN pro PC
SW1(config)#switchport trunk allowed vlan add <PC_VLAN>
SW1(config)#switchport trunk allowed vlan add <Phone_VLAN>
```
