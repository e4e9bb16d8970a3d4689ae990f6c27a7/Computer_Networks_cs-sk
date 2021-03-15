# IP ACL

## Standard
---
Jednoduchý Access Control List, filtruje pouze na základě Source IP adresy/Sítě
Access list defaultně blokuje všechen provoz (`Deny Any`), pokud chceme opačnou logiku, blokovat vybrané a zbytek povolit, musíme na konec Access listu zapsa `Permit Any`.

### Range

|Range|Type|
|:-----:|:----:|
|1 - 99 | Standard|
|1300 - 1999 | Extended|

## Extended
---
Oproti Standard ACL, Extended poskytuje více možností, podle kterých filtrovat.
- Protocol
- Source IP
- Source Port
- Destination IP
- Destination Port

### Range 
|Range|Type|
|:-----:|:----:|
|100 - 199 | Standard|
|2000 - 2699 | Extended|

## Named
---
Jedná se o Standard nebo Extended ACL, který není označen číselně, ale jmenně.

