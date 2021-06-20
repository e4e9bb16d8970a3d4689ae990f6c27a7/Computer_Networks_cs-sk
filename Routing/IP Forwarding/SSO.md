# Stateful Switchover
---

Pro udržení provozuschopnosti na 99,99 % času se do routerů dávají 2 RP (Route Processor), aby, pokud jeden selže, mohl druhý zastoupit jeho funkci, takovéto funkci automatického failoveru se říká Stateful Switchover (SSO).

Po přepojení na záložní RP okamžitě převezme základní funkce, jako interface link, ale funkce jako L3 forwarding nefungují a RIB se musí znovu vytvořit, od toho máme funkce NSF nebo NSR.

## NSR & NSF

Nonstop forwarding (NSF) nebo Nonstop routing (NSR) umožnují routeru udržet si na nějakou dobu obsah FIB a routovat dle něho, než si záložní RP vytvoří RIB, což mlže zahrnovat i nové sousedství při dynamic routing protocols.