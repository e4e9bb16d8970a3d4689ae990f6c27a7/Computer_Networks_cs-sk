# IS-IS 
---

Jedná se o otevřený ISO [[Routing#IGP - Interior Gateway Protocol|IGP]], [[Routing#Link-State|link-state]] routovací protokol využívající SPF pro OSI protocol suite.
Původně byl určen jako ISO/OSI L3 protokol pro *Connectionless Network Protocol* ([[OSI Network Layer]]).
Samotné IS-IS je dnes používáno jako [Integrated IS-IS](https://www.ietf.org/rfc/rfc1195.txt) pro fungování s TCP/IP suitem.
Stále je používán v backbone areách ISP sítí a jedná se o underlying technologii pro [TRILL](https://en.wikipedia.org/wiki/TRILL_(computing))/[Cisco FabricPath](https://ourtechplanet.com/what-is-cisco-fabricpath/).
Protože se nepoužívá L3, ale datagramy jsou encapsulované na L2 a používá [[TLV]], IS-IS je protocol independent.

## ![[OSI Network Layer]]

