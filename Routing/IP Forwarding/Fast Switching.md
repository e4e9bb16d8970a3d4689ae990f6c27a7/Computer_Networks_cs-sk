# Fast Switching
---

Jedná se o zásadní rychlostní skok oproti [[IP Forwarding|Process Switchingu]].
Jeho hlavní výhoda je přidání cache nazývané *route cache* nebo *switching cache*, která obsahuje cílovou IP adresu, next-hop a L2 header informace, čili všechny informace, které router potřebuje pro routing.

Jak je popsané v [[IP Forwarding]], proces hledání optimální cesty je poměrně zdlouhavý, a proto přidání rychlé cache paměti, ve které může router rychle najít všechny důležité informace a nemusí například čekat na ARP Request, je značné zrychlení.

Při dostání paketu si router nejprve prohledá cache a pokud zjistí, že nemá záznam pro danou IP adresu, předá paket normánímu Process Switchingu, po zjištění všech informací odešle paket na správnou next-hop adresu a zapíše zjištěné informace do cache, což značně zrychlí proces pro další pakety na stejnou IP adresu, vzhledem k tomu, že síťový provoz se málokdy skládá pouze z jednoho paketu.

Cache záznamy jsou vytvořeny na základě cílové IP adresy, ne sítě, což umožňuje reagovat na změny v routingu například pomocí [[ACL]] nebo [[PBR]], ale znamená to, že cache musí být, vzhledem k její omezené velikosti, rychle promazávána.

Již desítky let se upřednostňuje [[CEF]], a tak samo Cisco podporu Fast Switchingu odebralo z IOSu od verzí `12.2(25)S` a `12.4(20)T`.