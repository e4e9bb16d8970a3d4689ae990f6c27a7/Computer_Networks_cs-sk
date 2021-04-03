# CSMA/CD

## Popis

Původní specifikace Ethernetu počítala s kolizemi.
Přenosové média byla sdílena a huby byly hojně využívány, elektrické signály mohly kolidovat.
Proto obsahovala i specifikaci *Carrier Sence Multiple Access with Collision Detection (CSMA/CD)*, aby se o tyto případy postaral a minimalizoval rušení na přenašeči.

1.  Prvek, který má frame na poslání naslouchá, dokud je sběrnice bez provozu
2.  Poté začne posílat své framy
3.  Nadále poslouchá, jestli nenastala kolize
4.  Pokud kolize nastala, všechny stanice, které zrovna posílaly framy, pošlou jamming signal, aby všichni na síti věděli o kolizi
5.  Po dokončení jammingu, každý prvek, u jehož framů došlo ke kolizi si náhodně nastaví časovač a počká, než začne znovu posílat. Stanice, které neměli kolizi nečekají
6.  Po dokončení všech časovačů se znovu začíná u bodu 1

## Switch Buffering

V případě, že například na switch přijdou 3 framy, se stejným odchozím portem, switch je uchová do paměti a začne je odesílat jeden po druhém. Hub pouze přeposílal elektrické signály.