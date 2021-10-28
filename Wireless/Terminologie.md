# Terminologie bezdrátových sítí
---

## Architektura
---

### BSSID

*Basic Service Set ID* [[Typy sítí#Basic Service Set|BSS]]
Jedná se o [[MAC]] adresu konkrétního AP.

### SSID

*Service Set ID* 
Jedná se o ID sítě, kterou poskytuje/jí AP, často jedno sdílí SSID více AP.

### Roaming

Protože jedno SSID může být sdíleno více AP (BSSID), roaming umožňuje koncovému uživateli měnit BSSID bez změny SSID.
Jinými slovy umožňuje měnit AP, dle signálu, a stále být připojen do sítě.
Při přepojení se nemění pouze BSSID, ale často i Channel.

## dB
---

Jedná se o relativní poměr výkonu k základnímu výkonu, který je `1mW`.

$$
dB = log_{10}(mW)\times 10
$$
$$
mW = 10^{\frac{dB}{10}}
$$

$0dB = K$
$3dB = 2 \times K$
$10dB = 10 \times K$

Tento neintuitivní systém vychází z toho, že `dB` je založen na logaritmu.

### dBi

Jedná se o `dB` pro teoretický isotropní zářič, tedy anténu, které vysílá ve všech směrech 3D prostoru stejným výkonem, tato anténa je pouze teoretická a není možné ji vytvořit.
Je zaměnitelné s `dBm`.

### dBd

Jedná se o `dB` pro dipólovou anténu, která je běžná.

$$
dBd = dBi - 2.14
$$
$$
dBi = dBd + 2.14
$$

![|900](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fteletopix.org%2Fwp-content%2Fuploads%2F2013%2F04%2FAntanna-gain-for-LTE.jpg&f=1&nofb=1)

## EIRP

*Effective Isotropic Radiated Power*

Jedná se o konečnou hodnotu sílu signálu, která do sebe započítává všechny vlastnosti při přenosu signálu, typicky *Transmitter Power (+dBm)*, *Cable Loss (-dBm)*, *Antena Gain (+dBm)*.

# Přenos signálu
---

## MIMO

*Multiple Input Multiple Output*

Jedná se o funkci počínající *802.11n* umožňující pomocí více *T* a *R* antén vytvořít více separátních spojení a znásobit tak rychlost samotného spoje.

## Beamforming

Tato funkce umožňuje zaměřit signál přesněji přijemci a vysílat přimo jeho směrem, oproti normálnímu všesměrovému vysílání.
