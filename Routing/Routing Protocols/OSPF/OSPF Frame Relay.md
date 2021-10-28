# OSPF Frame Relay
---

Pri konfiguraci OSPF přes Frame Relay může být koncept Mpt a P-P poměrně složitý, je několik nastavení, na které je potřeba si dát pozor.

- Ujistěte se, že Hello/Dead časovače jsou správně nakonfigurovány
- Pokud jeden router očekává DR/BDR volbu a jiný ne, tyto dvě zařízení sice budou ve [[OSPF Adjacency#Full|Full]] stavu se sesynchronizovanými LSDB, ale finální funkcionalita bude velmi zvláštní, next-hop router totiž nebude dostupný
- DR/BDR musí mít PVC ke každému routeru v síti, aby s ním mohl výměňovat informace, router se bez PVC ke každému zařízení nemůže stát DR/BDR
- `neighbor` příkaz musí být nakonfigurován na oboustranách spoje

Pro nejrychlejší nastavení OSPF na Frame Relay, bez potřeby volby DR/BDR a specifikování sousedů, je, v případě, že to topologie dovoluje, možné nastavit network type všech routerů na *Point-to-point*, poté není potřeba další konfigurace a OSPF se o fungování postará.
Pokud je nutné použít multipoint subinterfacy nebo není možné použít subinterfacy, je nejjednodušší nastavit network type na *Point-to-multipoint*, poté se opět o zbytek postará OSPF.