[[VLAN]]
# PVSTPs
---

Jedná se o Cisco uspůsobenou verzi STP, která umožňuje spouštět STP instanci pro každou VLANu.
Funguje na MAC adrese `01:00:0c:cc:cc:cd`.
Díky tomu může nastavit například různé Root Bridge pro různé rozsahy VLAN.

To umožňuje jednak skrze nastavení, ale hlavně do BPDU přidává číslo VLANy:

|**BPDU Pole**|**Délka**|
|:----:|:----:|
| Originated VLAN (PVID/TLV) | 3B |

To slouží například k nacházení [[Switching/VLAN/Teorie/Terminologie#Native VLAN|native VLAN]] nastavení a jejího missmatche, v případě, že jsou [[Switching/VLAN/Teorie/Terminologie#Native VLAN|native VLANy]] špatně nastavené v CST regionu.


## Verze

|**Verze**|**Typ**|**Funkce**|**Jádro**|**Další**|
|:-:|:-:|:-:|:-:|:-:|
|STP|*common STP (CST)*|Pouze jedna instance|[[STP]]|/|
|PVSTP|*per-vlan STP*| Instance pro každou VLANu|[[STP]]|Pouze [[Switching/VLAN/Teorie/Terminologie#ISL\|ISL]], používá speciální BPDUs|
|PVSTP+|*per-vlan STP*|Instance pro každou VLANu|[[STP]]|Podporuje [[Switching/VLAN/Teorie/Terminologie#802 1q\|802.1Q]], [IEEE 802.1D](https://en.wikipedia.org/wiki/IEEE_802.1D)|
|RPVSTP|*per-vlan STP*| Instance pro každou VLANu|[[Rapid STP]]|Pouze [[Switching/VLAN/Teorie/Terminologie#ISL\|ISL]]|
|RPVSTP+|*per-vlan STP*|Instance pro každou VLANu|[[Rapid STP]]|Podporuje [[Switching/VLAN/Teorie/Terminologie#802 1q\|802.1Q]]|

Od verze `Cisco IOS 15.2(4)E` je defaultní RPVSTP+.
Původní PVSTP už nelze na dnešních zařízeních ani zapnout, pod nastavením `rapid-pvstp` se ve skutečnosti nachází RPVSTP+ a doporučená verze je [[MSTP]].

## Kompatibilita

PVSTP je Cisco proptietární protokol, který ovšem počítá s tím, že v síti jsou i jinné switche, než ty Cisco.

PVSTP+ posílá na [[Switching/VLAN/Teorie/Terminologie#Trunk|truncích]] i klasické [[STP Terminologie#BPDUs|STP BPDUs]], aby se předešlo problémům s nekompatibilitou.

(V případě Access portů so posílá pouze IEEE STP s [[STP Terminologie#Bridge ID|System ID Extension]] přiřazené VLANy portu.)

Uvnitř CST regionu je pouze jedna instance CST STP, která se promítá do VLAN 1 instance v PVST regionu, ostatní VLANy mají ovšem vlastní STP procesy, dle PVSTP+.

PVSTP+ se chová k CST regionu jako k *loop-free* sdílenému segmentu.
> PVST+ instances for VLANs other than VLAN 1 in PVST+ regions treat CST regions simply as loop-free shared segments.

Přes který jsou přeposílány, ale nezpracovávány, díky rozdílné Multicast MAC adrese, PVSTP BPDUs, a tak se pro, přímo nespojené, PVSTP segmenty nic neděje a mohou utvořit *loop-free* topologii.

VLAN 1 BPDUs jsou trochu speciální, jsou posílány jak [[STP]], tak i pomocí [[Per-VLAN STPs|PVSTP]], ale protože v CST regionu se zpracovává pouze [[STP]] [[STP Terminologie#Configuration BPDU 0x00|BPDU]], tak PVSTP slouží pouze k detekci [[Switching/VLAN/Teorie/Terminologie#Native VLAN|native VLAN]] missmatche.