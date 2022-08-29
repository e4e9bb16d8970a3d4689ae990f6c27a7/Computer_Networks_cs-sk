# SPF Calculation
---

OSPF využívá [Dijkstra's Algorithm](https://www.youtube.com/watch?v=EFg3u_E6eHU)/[Dijkstra's algorithm (test)](https://algorithms.discrete.ma.tum.de/graph-algorithms/spp-dijkstra/index_en.html) pro určení nejlepší cesty do cílové sítě. Algoritmus pracuje s routery, linkami, cenou linek a stavem (up/down).

1. Počáteční bod má metriku 0
2. Spočítáme metriku všech sousedních bodů (metrika bodu, na kterém jsme + metrika cesty)
  1. Pokud bod již má metriku, tak ji změníme pouze, pokud nová je nižší
3. Přepneme se na bod s nejnižší metrikou z celé sítě
4. Pokračujeme na 1.

Touto metrikou si každý router spočítá nejlepší cestu do každého jiného routeru a uloží si jeho metriku (cenu cesty k němu), tuto metriku sdílí každá síť, kterou daný router ohlašuje za svou.

## Steady-State Operation

I po konvergenci sítě všechny routery v arei provádí několik úkonů:

- Rozesílání Hello paketů na základě interface-based hello intervalu
- Očekávání Hello paketu v mezi dead timeru
- Každý router rozesílá svá LSA každý *Link-State Refresh* (LSRefresh) timer (30min)
- Každý router očekává LSRefresh na každý LSA záznam v mezi *LSA MaxAge* timeru (60min)
