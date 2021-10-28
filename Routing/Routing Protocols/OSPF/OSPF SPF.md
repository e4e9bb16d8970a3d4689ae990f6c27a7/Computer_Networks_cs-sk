# SPF Calculation
---

OSPF využívá [Dijkstra's Algorithm](https://www.youtube.com/watch?v=EFg3u_E6eHU) pro určení nejlepší cesty do cílové sítě. Algoritmus pracuje s routery, linkami, cenou linek a stavem (up/down).

## Steady-State Operation

I po konvergenci sítě všechny routery v arei provádí několik úkonů:

- Rozesílání Hello paketů na základě interface-based hello intervalu
- Očekávání Hello paketu v mezi dead timeru
- Každý router rozesílá svá LSA každý *Link-State Refresh* (LSRefresh) timer (30min)
- Každý router očekává LSRefresh na každý LSA záznam v mezi *LSA MaxAge* timeru (60min)
