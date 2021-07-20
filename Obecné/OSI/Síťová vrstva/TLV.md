# Type-Lenght-Value
---

Jedná se o formát pro ukládání a posílání různých dat v jednom datagramu, každé TLV obsahuje informaci, kterou chce odelásatel sdílet.

Dělí se na:

## Type

Kód indikující typ informace.

## Lenght

Délka TLV polí.

## Value

Pole obsahující konkrétní informaci pro určitý protokol.

Tento typ zápisu dat používá řada protokolů, například:

[[EIGRP]]
[[CDP]], [[LLDP]]
[[IS-IS]]