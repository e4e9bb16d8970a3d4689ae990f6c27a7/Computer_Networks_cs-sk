[[Switching/VLAN/Teorie/Terminologie#dot1q-tunnel]]
# Q-in-Q Tunneling (802.1ad)

Umožňuje přenášet *VLAN* tagy sítí *ISP*.
Používá se dvojité tagování framů, první frame má tag z *LAN*, druhý mu přiřadí *ISP* switch, pod kterým s ním bude pracovat v rámci své sítě.

Takových řešení je vícero, například *802.1ah* [^1] (*Provider Backbone Bridges*), *Layer2 Tunneling Protocol* [^2] (*L2TPv3*) nebo *VPLS* [^3] (*VLAN Private LAN Services*)
Ty ale *CCIE R&Sv7.0* nezahrnuje, zahrnuje pouze *Q-in-Q*.


## Popis

Z pohledu *ISP*, na interface přijde framy s *VLAN tagem* (*C-tag*, *LAN* zákazníka), pro zabezpečení v rámci sítě *ISP* a možné doručení je nutné, aby byl framu přidán další *VLAN* tag (*S-tag*).

*LAN ISP* vždy koukají pouze na první *VLAN tag* a zacházejí s framem dle něj.
Výstupní zařízení zase tento *S-tag* odstraní a frame se opět nachází ve *VLAN* zákazníka.

Může být zejména užitečné například pro *CDP* a *VTP* provoz, který lze nakonfigurovat pro průchod tunelem.

Zároveň, na Trunku, který  vstupuje nemůže být *Native VLAN*, to by mohlo vést ke špatnému určení *C-tagu*, což by vedlo k nesprávnému zacházení s provozem.

## Konfigurace

![[Q-in-Q.png]]

```
SP-SW1(config)#system mtu jumbo 1504     \\ Zvýšení MTU z důvodu 802.1Q tagování
```

```
SP-SW1(config)#vlan <VLAN ID>     \\ Vytvoření VLAN S-tag
SP-SW1(config-vlan)#name Customer1
```

```
SP-SW1(config)#interface <IF>     \\ Přepnutí na Interface k zákazníkovy
SP-SW1(config-if)#switchport mode dot1q-tunnel   \\ Nastavení módu tunelu
SP-SW1(config-if)#switchport access vlan <VLAN ID>   \\ Nastavení vnitřní VLAN C-tag
```

```
SP-SW1(config-if)# l2protocol-tunnel cdp   \\ Umožnění přeposílání různých síťových služeb
SP-SW1(config-if)# l2protocol-tunnel lldp
SP-SW1(config-if)# l2protocol-tunnel stp
SP-SW1(config-if)# l2protocol-tunnel vtp
```

[^1]: https://www.cisco.com/c/en/us/support/docs/routers/asr-9000-series-aggregation-services-routers/212882-understanding-basic-802-1ah-provider-bac.html
[^2]: https://en.wikipedia.org/wiki/Layer_2_Tunneling_Protocol
[^3]: https://en.wikipedia.org/wiki/Virtual_Private_LAN_Service

