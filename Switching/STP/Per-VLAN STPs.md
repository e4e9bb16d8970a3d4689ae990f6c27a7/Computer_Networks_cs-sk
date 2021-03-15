[[VLAN]]
# PVSTPs
---

Jedná se o Cisco uspůsobenou verzi STP, která umožňuje spouštět STP instanci pro každou VLANu.
Díky tomu může nastavit například různé Root Bridge pro různé rozsahy VLAN.

To umožňuje jednak skrze nastavení, ale hlavně do BPDU přidává číslo VLANy:

|**BPDU Pole**|**Délka**|
|:----:|:----:|
| Originated VLAN (PVID) | 3B |

## Verze

|**Verze**|**Typ**|**Funkce**|**Jádro**|**Další**|
|:-:|:-:|:-:|:-:|:-:|
|STP|*common STP (CST)*|Pouze jedna instance|[[STP]]|/|
|PVSTP|*per-vlan STP*| Instance pro každou VLANu|[[STP]]|Pouze [[Terminologie#ISL\|ISL]], používá speciální BPDUs|
|PVSTP+|*per-vlan STP*|Instance pro každou VLANu|[[STP]]|Podporuje [[Terminologie#802 1q\|802.1Q]], [IEEE 802.1D](https://en.wikipedia.org/wiki/IEEE_802.1D)|
|RPVSTP|*per-vlan STP*| Instance pro každou VLANu|[[Rapid STP]]|Pouze [[Terminologie#ISL\|ISL]]|
|RPVSTP+|*per-vlan STP*|Instance pro každou VLANu|[[Rapid STP]]|Podporuje [[Terminologie#802 1q\|802.1Q]]|

Od verze `Cisco IOS 15.2(4)E` je defaultní RPVSTP+.
Původní PVSTP už nelze na dnešních zařízeních ani zapnout, pod nastavením `rapid-pvstp` se ve skutečnosti nachází RPVSTP+ a doporučená verze je [[MSTP]].