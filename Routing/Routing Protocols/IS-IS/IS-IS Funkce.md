# IS-IS Autentizace

|     Paket     |  Level  |                                                  Příkaz                                                  |                 Starý příkaz                 |
|:-------------:|:-------:|:--------------------------------------------------------------------------------------------------------:|:--------------------------------------------:|
|    LAN IIH    | Level 1 |          `if#isis auth mode {text \| md5} level-1`/`if#isis auth key-chain [KEY-CHAIN] level-1`          |     `if#isis password [passwd] level-1`      |
|    LAN IIH    | Level 2 |          `if#isis auth mode {text \| md5} level-2`/`if#isis auth key-chain [KEY-CHAIN] level-2`          |     `if#isis password [passwd] level-1`      |
|    P2P IIH    |   N/A   |                  `if#isis auth mode {text \| md5}`/`if#isis auth key-chain [KEY-CHAIN]`                  |         `if#isis password [passwd]`          |
| LSP/CSNP/PSNP | Level 1 | `router-isis#isis auth mode {text \| md5} level-1`/`router-isis#isis auth key-chain [KEY-CHAIN] level-1` | `router-isis#isis password [passwd] level-1` |
| LSP/CSNP/PSNP | Level 2 | `router-isis#isis auth mode {text \| md5} level-2`/`router-isis#isis auth key-chain [KEY-CHAIN] level-2` | `router-isis#isis password [passwd] level-2` |

IS-IS umožňuje MD5 a plaintext autentizaci, i použití key chainů, ale číslo klíče se v paketech neposílá, může se tedy lišit.

[[IS-IS Pakety#Hello paket|IIH]] jsou autentizovány odděleně od ostatních, protože, zejména [[IS-IS Pakety#Link State PDU|LSP]], nemuže být pozměněno nikým jiným, než routerem, který ho vytváří, heslo pro LSP tedy musí být stejné pro celou areu, to ale, zejména v síťích ISP, může být bezpečnostní problém, a tak se odděluje speciální heslo pro IIH.

V případě, že IIH pakety nejsou autentizovány, nedojde k navázání spojení, ani kdyby heslo pro ostatní pakety bylo správné.
Pokud je heslo pro IIH správné, ale pro ostatní pakety nikoli, Adjacency bude Up, ale nebude docházet k výměně LSPs.

IIH lze nastavit *interface-based*.

# IS-IS IPv6

Protože IS-IS používá TLV pole, nedochází k žádné změně u [[IPv6]], stačí pouze zapnout protokol, stejně, jako u IPv4.