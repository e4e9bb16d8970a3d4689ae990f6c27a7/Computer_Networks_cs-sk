# Open Shortest Path First
---

Jedná se o otevřený [[Routing#IGP - Interior Gateway Protocol|IGP]] [[Routing#Link-State|Link-State]] routovací protokol popsaný v [RFC 2328](https://datatracker.ietf.org/doc/html/rfc2328).
Na základě přichozích informací protokolu o všech linkách v síti sestavuje *Link-state database* (*LSDB*), nad kterou využívá [Dijkstrův algoritmus](https://cs.wikipedia.org/wiki/Dijkstr%C5%AFv_algoritmus) pro výběr nejlepší cesty do každé sítě a sestavení *loop-free* topologie.
V praxi se vypočítává nejlepší cesta j nodům (routerům), které mají jako atributy připojené sítě.

