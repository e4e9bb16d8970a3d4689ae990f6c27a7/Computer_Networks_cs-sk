# Proxy ARP
---

V případě špatně nakonfigurované masky, z jakéhokoli důvodu, například dle defaultního *classfull* dělení, se může stát, že například na koncovém zařízení máme masku `/8`, ale reálná síť, tedy maska na routeru je `/24`. V takovém případě nebude fungovat ARP z například `10.1.0.10` na `10.1.0.5`, protože koncové zařízení si myslí, že obě zařízení jsou na shodné síti, a tedy může použít ARP, ve skutečnosti nejsou na jedné *broadcast* doméně, a tak ARP nebude fungovat.

ARP Proxy tento problém řeší tak, že router tento ARP Request přijme a přepošle ho do příslušné sítě, v případě, že se mu vrátí odpověď, pošle koncovému zařízení ARP Reply s vlastní MAC adresou.

Tato funkce je defaultně povolena, ale lze ji vypnout: `R1(config-if)#no ip proxy-arp`

# Reverse ARP
---

Jedná se o jeden z prvních protokolů na automatické zjištování IP adres, definován byl již v roce 1984 v [RFC 903](https://tools.ietf.org/html/rfc903).

Funguje na principu ARP s tím rozdílem, že ARP Request obsahuje vlastní MAC adresu s `0.0.0.0` IP adresou, jedná se tak o obrácený, reversní, princip ARPu.
Nakonfigurovaný server pak na tento požadavek odpoví ARP Reply s příslušnou IP adresou.

Tento protokol má řadu nevýhod, kvůli kterým nebyl ostatně nikdy moc využíván.
- Přiděluje pouze IP adresu, ani DG nebo masku
- Je pouze na broadcastové doméně
- Je nutné manuálně přidělit každé MAC adrese IP adresu

# BOOTP
---

Funguje podobně, jako [[#Reverse ARP]], s tím rozdílem, že neposílá ARP Framy, ale využívá vlastní nad [[UDP]].
Framy obsahují vlastní MAC adresu s `0.0.0.0` IP adresou a nakonfigurovaný server poté pošle Reply s nakonfigurovanou IP adresou.

Oproti [[Reverse ARP]] má řadu výhod:
- Lze přidělit celou řadu dalších informací
- Není nutné mít server v broadcastové doméně, lze tak mít pro celou síť centrální prvek

Ale nevýhoda v podobě manuálního přidělování zůstává, to vyřešil až [[DHCP]].
