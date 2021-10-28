# Základní vlastnoti RF
---

## Elektromagnetické spektrum
---

![[EM.png|"2000"]]

Foton reprezentuje částici/vlnu přenášející elektromagnetické záření, toto záření je generováno v elektromagnetickém poli oscilací elektrického a magnetického pole, vyšší oscilace znamená vyšší frekvenci a tím i energii fotonu.
Jakýkoliv průchozí proud vodičem generuje magnetické pole a naopak magnetické pole generuje proud na vodiči, když vodičem, v našem případě vysílající anténou, povedeme střídavý elektrický proud, vodič bude vyzařovat elektromagnetické záření o frekvenci tohoto střídavého proudu, které může být indukováno jako proud na jiném vodiči, v našem případě příjmovou anténou.
Díky tomu, že elektromagnetické pole je všeobklopující, oproti například vzduchu, který potřebuje částice pro propagaci, toto záření může být přeposíláno i bez přímo připojeného vodiče, drátu, jedná se tedy o bezdrátové spojení.

![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fkuow-prod.imgix.net%2Fstore%2F37a0df06095c48e64dc02bc213a9c37a.gif%3Fixlib%3Drails-2.1.4%26auto%3Dformat%26crop%3Dfaces%26fit%3Dcrop%26h%3D634%26w%3D924&f=1&nofb=1)
![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.emraware.com%2FImages%2Fantenna_animation.gif&f=1&nofb=1)
![[FR_AM.png|900]]
Vlnová delká, frekvence a energie je lehce zaměnitelná na základě těchto zákonů:

$$
\lambda = \frac{c}{f}
$$
$$
\mathrm{E} = h \times f
$$
Co se energie týče, ionizující záření začíná na frekvenci 750 THz.

## Noice
---

V prostoru je obecně vždy nějaké zarušení, které musí vysílající strana překonat pro dodání kvalitního signálu, přijímaný signál se označuje za *Received Signal Strenght Indicator* (RSSI), poměr tohoto chtěného signálu a nechtěného rušení označuje *Signal Noice Ration* (SNR).
![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fdocumentation.meraki.com%2F%40api%2Fdeki%2Ffiles%2F4251%2Fclipboard_e8c4bdb9e8daf8c81cb963f4185979f69.png%3Frevision%3D1&f=1&nofb=1)
## Free Path Loss
---

Samotná vlna ztrácí svou sílu na základě kvadrátu vzdálenosti od vysílacího bodu znamenaje, že pokud jsme 2x daleko od vysílajícího bodu, síla signálu bude pouze čtvrtinová.
Již při prvotním pohledu na rozdíl mezi vysokou a nízkou frekvencí je jasné, že nižší frekvence musí překonat k cíli nižší vzdálenost, protože má mnohem přímější cestu k cíli, kdežto vysokofrekvenční vlna musí mimo jiné oscilovat okolo vlastní osy, čímž si samotnou cestu zdelšuje a snižuje vlastní schopnost penetrace.
Jinými slovy, čím vyšší frekvence, tím horší dosah signálu a penetrace elektromagnetických vln, například přes zdi.

Vzdáleností tedy obecně vlna ztrácí na čitelnosti, protože se splošťuje její amplitůda, toto můžeme upravit vysílacím výkonem, při zvýšení vysílacího výkonu se zvedne původní amplitůda, čímž bude elektromagnetické záření jednodušeji rozpoznáno i na větší vzdálenosti.

## Absorpce
---

Elektromagnetické záření neindukuje proud pouze na naší požadované anténě, ale i na dalších objektech, které mu stojí v průběhu přenosu v cestě, čím delší ona cesta je, tím spíše se srazí s těmito objekty a předá jim svou energii a naopak, čím kratší cesta je, tím menší je pravděpodobnost oslabení vlny.

Různé materiály reagují různě s frekvencemi, například spektrum okolo `2.45 GHz` je velmi náchylné na $H_2O$, protože molekula vody má oscilační frekvenci právě na `2.45 GHz`, toho mimo jiné využívají mikrovlné trouby, ale zároveň to znemaná, že klasický `2.4 GHz` spoj bude mít problém penetroval cokoli s vysokým obsahem vody, typicky bezdrátový spoj, který by měl penetrovat listnatý strom...

## Reflekce
---

Stejně, jako viditelné světlo v zrcadle, se i ostatní frekvence elektromagnetického záření odrážejí od různých povrchů, nejčastěji metalických.
Odrazy způsobují rušení, zpoždění a řadu dalších problémů v bezdrátové síti, je tedy potřeba se jim co nejvíce vyhnout, nejčastěji je potřeba si dát pozor při montáži AP, aby nebylo montování v blízkosti metalického materiálu a pokud je, aby bylo minimálně jednu vlnovou délku od něj.

### Multipath

Jedná se o mnohonásobné příjmutí stejných rádiových vln kvůli jejich odrážení a tím pádem spoždění.
Koncový příjemce si s takovou situací musí umět poradit, ve starších sítích zařízení příjmulo první, nebo nejsilnější vlnu.

## Scattering
---

Jedná se o jev, typicky ve vnějším prostředí, kdy máme ve vzduchu vysoký obsah odrazivého materiálu, typicky při dešti nebo v nějaké vyrobní hale například s kovovými štěpkami,  při kterém se signál roztříšťuje o každou odrazovou částici a tím pádem se značně zhoršuje signál.

## Refraction
---

Jedná se o zalomení signálu při přechodu z jednoho média do druhého.

![[RF_Wave.png|900]]

## Line of Sight
---

Zejména při vytváření [[Typy sítí#Bridging|P2P/P2MP]] spojů o vysokých frekvencích je důležitý *Lone of Sight* (LOS), jedná se o přímou viditelnost mezi zářičem a příjemcem, tato viditelnost nesmí být narušena jakýmkoli materiálem.
Déle je je důležité udržet volnou *Fresnal Zone*, jedná se o 3D prostor okolo LOS, ve kterém se šíří většina signálu, tento prostor závysí na délce spoje a frekvenci.
Fresnal zone je teoreticky nekonečná, ale pro praktické využití je potřeba mít volných minimálně 60%, jinak dochází ke značným ztrátám bandwithu.

![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.tekonelectronics.com%2Fimagem%2FLoS_Fresnel_Zone_EN.jpg&f=1&nofb=1)

$$
r = 17.32 \times \sqrt{\frac{d(km)}{4f(GHz)}}
$$
$r(m)$ pro nejširší bod Fresnal zone, který je pochopitelně uprostřed spoje.

[Online kalkulačka](https://afar.net/fresnel-zone-calculator/)

## Channels
---

### `2.4GHz`

Kvůli omezení rušení jednotlivých vyzařujících zařízení je vymezen rozsah spektra pro naše použití (`2412MHz` - `2484MHz`) a toto spektrum je dále rozděleno do 14 kanálů, každý o vlastní šířce `5MHz`.

| Chanell | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  | 12  | 13  | 14  |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Frekvence   (`MHz`)|2412|2417|    2422 |2427     |2432     |2437     | 2442    |  2447   |  2452   |2457     | 2462    |   2467  | 2472    |  2484   |

Většina bezdrátových technologií přitom využívá [[#Spread Spectrum]], větší šířku pásma, než původních `5MHz`. Typicky *802.11* `20MHz/40MHz` (*802.11b* `22MHz`).
![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.duckware.com%2Ftech%2Fimages%2Fchannels24.jpg&f=1&nofb=1)
Pro nepřekrývající se kanály je tedy nejlepší vybrat **1**, **6** a **11**, nebo i **1**, **5**, **9**, **13**.

Nutno podotknout, že EU používá **13** kanálů, US **11** kanálů a JAP **14** kanálů.

## Spread Spectrum
---

Samotný termín *Spread Spectrum* označuje způsob, ajkym je provoz v rádiovém spektru rozprostřen.

### DSSS

*Direct Sqeuence Spread Spectrum*

Tato metoda využívaná u *802.11b* nebo *GPS* převádí data na takzvanné **Chipy**, `0` a `1` kóduje delší sekvencí, diky čemuž může dojít k zarušení nebo ztrátě jednotilvých bytů v sekvenci, ale konečný bit, který sekvence reprezentuje, bude stále čitelný.

`1` = `01001000111`
`0` = `10110111000`

Tato metoda je velmi dobrá v proti rušení, proto je ostatně využívána u *GPS* a *Galileo*, ale zato nabízí značné omezení bandwithu, oproti [[#OFDM]] využívá najednou celý `22MHz` channel.

### OFDM

*Orthogonal Frequency-Division Multiplexing*

Tento přístup pro obranu proti zarušení rozděluje přidělené pásmo, defaultně `20MHz`, na *sub-carrier* o šířce `312.5KHz`, které poté přiděluje jednotlivým uživatelům.
Rozděluje na 52 *sub-carrier*, přičemž 4 jsou pro řízení a 48 pro tok dat.

Tento způsob je využíván *802.11a/g/n/ac/ax*.

## Modulace
---

Jedná se o způsoby ovlivnění rádiových vln, které umožňují do tohoto média zabudovat informaci.

![|900](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Amfm3-en-de.gif/220px-Amfm3-en-de.gif)

### AM (Amplituda)

Tento typ modulace mění amplitůdu, tedy sílu, signálu.

- #### QAM
	*Quadrature Amplitude Modulation*
	Používá QPSK s AM.
	Využívá rozdělení na 4 částí, které nebízí QPSK a dále je rozděluje na další sub-části pomocí AM.
	![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse3.mm.bing.net%2Fth%3Fid%3DOIP.oisggk-HutSrUU9FgBWbMgHaF1%26pid%3DApi&f=1)
	Tato modulace se využívá dnes v *802.11* standardech:
		- *16-QAM* ($4^2$) - *802.11a/g/n*
		- *64-QAM* ($4^3$) - *802.11a/g/n*
		- *256-QAM* ($4^4$) - *802.11ac*
		- *1024-QAM* ($4^5$) - *802.11ax*

### FM (Frekvence)

Tento typ modulace mění frekvenci signálu.

### PM (Fáze)

Tento typ posouvá fázi signálu.

![|900](https://upload.wikimedia.org/wikipedia/commons/7/72/Phase_Modulation.png)

#### PSK / PSK

*Phase-shift keying*

Jedná se o způsob ovlivňování fáze signálu při přenosu, typicky zastavením vysílání a okamžitém započetí vysílání v rozdílné fázi, což pozmění signál, z čehož lze následně číst data.

- ##### BPSK
*Binary phase-shift keying*
Tato metoda umožňujr změnu fáze o $180\degree$, čímž umožňuje samotné kódování informace.

![|900](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fwww.evalidate.in%2Flab2%2Fimages%2FBPSK_I%2FBPSK_I1.jpg&f=1&nofb=1)

- ##### QPSK
*Quadruple phase-shift keying*
Tato metoda umožňuje změnu fáze o $90\degree$, čímž zvyšuje bitrate oproti *BPSK*.
![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.elprocus.com%2Fwp-content%2Fuploads%2FQuadrature-Phase-Shift-Keying-Waveform.jpg&f=1&nofb=1)


