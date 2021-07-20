# EIGRP Named mode
---

Počínaje `IOS 15.0(1)M` Cisco routery mají zabudovaný *Named mode* konfigurace. Dnes je tento mód doporučený pro naprostou většinu aplikací.
Původní *classic | autonomous system mode* je označen za zastaralý a je stále dostupný z důvodu zpětné kompatibility, ale nové příkazy a funkce, které jsou pro EIGRP poskytovány nebo budou poskytovány již nebudou zabudovávány do classic módu.
Na zařízení je možné mít více instancí EIGRP, a to včetně Classic a Named.

Důvod pro vytvoření Named módu je jednoduchý, postupen času a s přidáváním dalších funkcí, IPv6 apod. byl vytvořen Named mód, který všechny EIGRP-related příkazy implementuje do jednoho konfiguračního režimu, všechny ostatní příkazy a konfigurace, jako například per-interface timery, autentizace atd..., jsou ignorovány a nastaveny přes named režim.

Named mód je rozdělen do 3 stavebních bloků:

- Address Family (AF) sekce
	- Zde se definuje základní nastavení typu `network` a `neighbor`
- Per-AF-interface sekce
	- Jedná se o příkazy specifické pro interfacy
- Per-AF-topology sekce
	- Zde se jedná o podporu *Multi Topology Routing* MTR


