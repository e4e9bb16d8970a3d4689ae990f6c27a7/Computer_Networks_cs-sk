# IPsec
---

*IPsec* je soubor otevřených standardů pro vytváření zabezpečených VPN.
Nabízí řadu služeb:

|Bezpečnostní služba|Popis|Metoda|
|:-:|:-:|:-:|
|**Peer Authentication**|Ověření koncových zařízení VPN spojení skrze autentizaci|Pre-Shared Key (PSK), Certifikáty|
|**Data Confidentiality**|Chrání data před odposloucháváním, šifruje|Data Encryption Standard (DES), Triple DES (3DES), Advanced Encryption Standart (AES)|
|**Data Integrity**|Předchází *MitM* útokům díky zabezpečení integrity dat pomocí hashů|Hash Message Authentication Code: MD5, SHA-1|
|**Replay Detection**|Předchází *MitM* útokům, kdy se utočník může pokusit odchytit VPN provoz, znovu poslat ho peeru pro vytvoření vlastního VPN spojení|Každý paket má sekvenční číslo, VPN zařízení nepříjme paket, který již jednou příjmulo|

>Dnes je doporučeno používat AES, nikoli DES/3DES.

## Authentication Header (AH)
---
Poskytuje *Peer authentication*, *Data integrity* a *Replay detection*, díky tomu, že přidává digitální podpis enkapsulovaného paketu, podobný kontrolnímu součtu.
Problém ale je, že nepodporuje *Data confidentiality* (šifrování) ani *Nat Transversal* (*NAT-T*), z těchto důvodů se od ní odstupuje.
Používá IP:51.

```packetdiag
packetdiag {
  colwidth = 32;
  node_height = 72;

  0-7: Next Header;
8-15: Payload Lenght;
16-31:Reserved;
32-63:Security Parameter Index;
64-95: Authentication Data (digest) (Variable lenght);
}
```

## Encapsulating Security Payload (ESP)
---
Poskytuje *Peer authentication*, *Data confidentiality*, *Data integrity* a *Replay detection*.
ESP zabezpečuje *Data confidentiality* pomocí šifrování, běží na IP:50 a k enkapsulovanému (původnímu) paketu přidává 2 hlavičky a podporuje *NAT-T*.
Běžně funguje ve 2 módech:
- **Tunnel mode**
	- Šifruje celý původní paket a přidává nové IPsec hlavičky, ty jsou použity k routingu a overlay funkcím.
- **Transport mode**
	- Šifruje pouze payload původního paketu a routing je založen na původní IP hlavičce.

![[Pasted image 20220202120447.png]]

### Podporované protokoly
---
#### Šifrování

- **Data Encryption Standart (DES)**
	- `56b` symetrické šifrování dat
	- Jedná se o slabé šifrování a nemělo by být používáno
- **Triple DES (3DES)**
	- Šifrovací algoritmus DES je proveden třínásobně nad daty, pokaždé s jinými `56b` klíči
	- Dnes nepoužíván díky AES
- **Advanced Encry (AES)**
	- Symetrické šifrování navrženo jako náhrada DES/3DES.
	- Podporuje `128b`, `192b` a `256b` klíče
	- Založen na *Rijndael* algoritmu

#### Hashování
- **Message Digest 5 (MD5)**
	- Jednosměrné `128b` hashování používané pro ověřování dat.
	- Cisco zařízení používají bezpečnější *MD5 HMAC*.
	- Dnes je doporučeno používat SHA
- **Secure Hash Algorithm (SHA)**
	- Jednosměrné `160b` hashování používané pro ověření dat.
	- Cisco používá bezpečnější *SHA-1 HMAC*

#### Ostatní

- **Diffie-Hellman (DH)**
	- Jedná se o způsob bezpečné výměny sdílených klíčů používaných šifrovacími algoritmy, jako AES, přes nezabezpečený kanál.
	- **DH Group**
		- Označuje délku a způsob generování klíče používaného pro funkci DH.
		- *group 1* má délku `768b`
		- *group 5* má délku `1536b`
		- *group 14* má délku `2048b`
		- *group 19* má délku `256b` s eliptickou křivkou
		- Pro `128b` klíče by se měla využívat nejméně G14, nejlépe G19, G12, G24
		- Pro `256b` klíče by se měla využívat nejméně G21, G24
- **RSA signatures**
	- *public-key* používán pro autentizaci peerů
- **Pre-Shared Key**
	- Systém, při kterém je lokálně zadaný klíč použit jako přístupový údaj pro autentizaci

## Transform Sets
---
Jedná se o konkrétní set protokolů používaných pro daný přenos ve VNP.
Na tomto nastavení musí být shoda mezi peery, k dohodě dochází při SA vyjednávání.

- **Authentication Header transform**
	- `ah-md5-hmac`
		- AH s MD5 algoritmem
		- Nedoporučeno
	- `ah-sha-hmac`
		- AH s SHA algoritmem
	- `ah-sha256-hmac`
		- AH s SHA-256 algoritmem
	- `ah-sha384-hmac`
		- AH s SHA-384 algoritmem
	- `ah-sha512-hmac`
		- AH s SHA-512 algoritmem
- **ESP encryption transform**
	- `esp-aes`
		- ESP s AES-128 šifrováním
	- `esp-gcm` / `esp-gmac`
		- ESP s `128b` (defaultně) / `256b` šifrováním
	- `esp-aes 192`
		- ESP s AES-192 šifrováním
	- `esp-aes 256`
		- ESP s AES-256 šifrováním
	- `esp-des` / `esp-3des`
		- ESP s `56b` / `168b` DES šifrováním
		- Nedoporučováno
	- `esp-null`
		- Bez šifrování
	- `esp-seal`
		- ESP s SEAL-160 šifrováním
- **ESP authentication transform**
	- `esp-md5-hmac`
		- ESP s MD5 autentizací
		- Nedoporučeno
	- `esp-sha-hmac`
		- ESP s SHA autentizací
- **IP compression transform**
	- `comp-lzs`
		- IP komprese s *LZS* algoritmem

## Internet Key Exchange
---
Jedná se o protokol, který má za úkol provést autentizaci mezi dvěma koncovými body, a navázat tak *Security Associations* (*SA*), také známou, jako *IKE Tunnel*.
Tyto spojení se používají pro přenos Control-Plane.

Existují 2 verze IKE protokolu, IKEv2 bylo navržena jako nástupce IKEv1 a měl za úkol vyřešit řadu IKEv1 problémů, například podporu *EAP* (certificate-based autentizace), anti-DoS funkcí, zefektivnění navazování SA tunelu a jednotnost v rámci jednoho RFC, protože IKEv1 je rozprostřen mezi hned několik RFCs.

### IKEv1
---
*Internet Security Association Key Management Protocol* (*ISAKMP*) je framework pro autentizaci a bezpečnou výměnu klíčů mezi dvěma peery pro navázání, úpravu a ukončení *SA*.
Je navržen pro podporu více způsobů výměny klíčů a používá UDP:500.

IKE je implementací tohoto ISAKMP pomocí *Oakley and Skeme key exchange techniques*.
*Oakley* poskytuje *Perfect Forward Secrecy (PFS)* proklíče, ochranu identity a autentizaci.
*Skeme* poskytuje anonymitu, zaručuje, že se jedná o správný peer a možnost rychlého obnovení klíčů.
Pro Cisco je *ISAKMP* a *IKE* používáno zaměnitelně.

IKEv1 používá 2 fáze navazování IPsec SA tunelu:

#### Phase 1

Navazuje obousměrný SA tunel mezi dvěma IKE peery, také známý, jako ISAKMP SA/IKE SA.
Díky tomu, že tento tunel je obousměrný, jakýkoli z peerů mlže zahájit *Phase 2* vyjednávání.

Pro vyjednání 1. fáze se může použít *Main mode* (*MM*), nebo *Aggressive mode* (*AM*).
Peer zahajující vyjednávání  se nazývá *initiator*, peer odpovídající se označuje jako *responder*.

##### MM

*Main mode* se skládá z 6 zpráv, při výměně každé z nich se snaží informace zabezpečit.

- **MM1**
	- Jedná se o první zprávu poslanou pro vyjednání IKE SA
	- Posílá ji *initiator* a obsahuje informace o tom, jak by měl tunel vypadat, *responder* si vybere jednu z možných konfigurací, pakliže má odpovídající konfiguraci.
	- MM1 obsahuje informace o:
		- *Hash algorithm* (MD5 / SHA)
		- *Encryption algorithm* (DES / 3DES / AES)
		- *Authentication method* (Pre-Shared Key / Certificate)
		- *Diffie-Hellman group* (G5, G14, G20 ...)
		- *Lifetime*
			- Určuje po jaké době má být IKE SA tunel automaticky ukončen.
			- Defaultně je 24 hodin.
			- Jako jediná hodnota se nemusí shodovat, pokud se na obou peerech liší, pak se vybere nejnižší čas.
- **MM2**
	- Tuto zprávu posílá *responder* na *initiator* s konkrétní konfigurací IKE SA, kterou přijímá.
- **MM3**
	- *Initiator* začíná s váměnou DH klíče.
- **MH4**
	- *Responder* posílá své klíče.
	- V tuto chvíli došlo k výměně klíčů a IKE SA tunel je tedy šifrovaný
- **MH5**
	- *Initiator* začíná autentizaci posláním vlastní IP adresy na *responder*.
- **MH6**
	- *Responder* pošne na *initiator* stejnou autentizační zprávu a IKE SA tunel je plně vytvořen.

Při použití tohoto módu jsou identity IKE peeru skryté, fungování je velmi bezpečné, ale vyjednání trvá delší dobu, oproti *AM*.

##### AM

*Aggressive mode* používá 3 zprávy a jeho vyjednání je rychlejší, za to ale i méně bezpečné a identity peerů lze odposlechnout.

- **AM1**
	- *Initiator* pošle najednou všechny informace z *MM1* - *MM3*, *MM5*
- **AM2**
	- *Responder* pošle všechny informace najednou obsažené v *MM2*, *MM4*, *MM6*
- **AM3**
	- *Responder* pošle ekvivalent *MM5* zprávy

#### Phase 2

Navazuje jednosměrné IPsec SA za pomocí již navázaného IKE SA v 1. fázi.
Oproti IKE SA, které jsou obousměrné, IPsec SA, o jejiž navazování se *Phase 2* stará, jsou jednosměrné a pro plnou komunikaci se tedy běžně navazují 2.
Metoda pro navazování IPsec SA se nazývá *Quick mode* a používá 3 zprávy:
- **QM1**
	- Posílá ji *initiator*, v tomto případě se může jednat o kterýkoli z peerů
	- Obsahuje seznam navazovaných IPsec SA, v jedné zprávě jich může navázat více
	- Obsahuje seznam dohodnutých, z 1. fáze, šifrovacích a hashovacích algoritmů
	- Obsahuje provoz, který se má šifrovat
- **QM2**
	- Zpráva od *redponder* obsahující odpovídající IPsec parametry
- **QM3**
	- Kontrolní zpráva, po jejíž přijmutí by měli existovat 2 jednosměrné IPsec SA kanály

##### Perfect Forward Secrecy (PFS)

Jedná se o volitelnou funkci 2. fáze, která je doporučená, ale ne vyžadována, protože vyžaduje další DH výměny.
Hlavním cílem této funkce je vyšší rezistence proti crypto útokům tím, že klíče jsou vždy generovány samostatně, bez pomoci předchozích.

### IKEv2
---
Jedná se o evoluci IKEv1 obsahující řadu zjednodušení a vylepšení, které ho dělají obecně efektivnějším.
Jednou z hlavních změn je způsob navazování SAs:
V IKEv2 se komunikace skládá z párů požadavků a odpovědí nazývaných *exchanges* nebo *request/response pairs*.
- **IKE_SA_INIT**
	- Jedná se o první pár, který vyjednává šifrovací a hashovací algoritmy
	- Provádí DH výměnu
	- Ekvivalent párům *MM1* - *MM4*
- **IKE_AUTH**
	- Autentizuje předešlý pár a potvrzuje identitu peerů a certifikátů
	- Vytváří IKE SA a *child SA* (IPsec SA)
	- Ekvivalent k IKEv1 *MM5* - *MM6* a *QM1* - *QM2*
- **CREATE_CHILD_SA**
	- V případě, že jsou potřeba další IPsec SAs, posílají se tyto zprávy, oproti IKEv1, kde je potřeba projít celý QM
### IKEv1 vs IKEv2

IKEv2 nabízí řadu výhod oproti IKEv1:
- **Efektivnost**
	- K vytvoření SA se používá méně zpráv
- **Elliptic Curve Digital Signature Algorithm (ECDSA-SIG)**
	- Jedná se o novější alternativu pro veřejné klíče, která je efektivnější
	- Určitá implementace, dle RFC 4754, je i v IKEv1, ale není široce používána
- **Extensible Authentication Protocol (EAP)**
	- Používání EAP, například oproti PSK, je dobrou a bezpečnou variantou pro remote-access VPNs
- **Next Generation Encryption (NGE)**
	- Toto rozšíření nabízí silnější klíče, a tedy i silnější šifrování
- **Asymmetric authentication**
	- IKEv2 umožňuje 2 peerům používat rozdílné typy autentizace, díky specifikaci v *IKE_AUTH*
- **Anti-DoS**
	- IKEv2 umožňuje detekovat, kdy je pod DoS útokem, ukončit spojení a šetřit tak prostředky
- *Exchange modes*
	- |**IKEv1**|**IKEv2**|
	|:-:|:-:|
	|Main mode, Aggressive mode, Quick mode|IKE_SA_INIT, IKE_AUTH, CREATE_CHILD_SA|
- *Minimální počet vyměněných zpráv*
	- |**IKEv1**|**IKEv2**|
	|:-:|:-:|
	|9 (MM) / 6 (AM)|4|
- *Podporované autentizační metody*
	- |**IKEv1**|**IKEv2**|
	|:-:|:-:|
	|PSK, RSA-SIG, Public key|PSK, RSA-SIG, ECDSA-SIG, EAP|
- *Next Generation Encryption (NGE)*
	- |**IKEv1**|**IKEv2**|
	|:-:|:-:|
	|Nepodporuje|AES-GCM, SHA-256, SHA-384, SHA-512, HMAC-SHA-256, ECDH-384, EXDSA-384|
- *Ochrana proti útoku*
	- |**IKEv1**|**IKEv2**|
	|:-:|:-:|
	|*MitM*, *Eavesdropping*|*MitM*, *Eavesdropping*, *Anti-DoS*|