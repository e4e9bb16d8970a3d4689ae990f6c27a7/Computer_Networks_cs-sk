# Vyzařování
---

## H-Plane

Označuje radiaci magnetického pole, většinou se jedná o vertikální vyzařování.

## E-Plane

Označuje radiaci elektrického pole, většinou se jedná o horizontální vyzařování.

![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fi.stack.imgur.com%2FFuyoB.png&f=1&nofb=1)

Některé antény mohou mít obrácenou polarizaci a E-Plane může být vertikální, ale většina namá, pro [[Typy sítí#Bridging|P2P/P2MP]] spoje musí být polarizace antén shodná.

# Typy
---
### Omnidirectional (Všerměrová)

Tento typ antém teoreticky, málo kdy prakticky, vyzařuje ekvivalentně ve všech směrech.

![|900](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fmpantenna.wpengine.netdna-cdn.com%2Fwp-content%2Fuploads%2F2015%2F02%2FFIGURE-3.png&f=1&nofb=1)

Nejčastěji dipólová anténa:

![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2F4.bp.blogspot.com%2F-N90A1dZonhw%2FWA5HLU7WIBI%2FAAAAAAAAH3Y%2F9dkD9dV_Ukoc42aFLNtzubZ1ROm8Pq-UACK4B%2Fs1600%2Flpda1151000.jpg&f=1&nofb=1)

Nebo i:

![|900](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fep.yimg.com%2Fay%2Fyhst-46542159655918%2F2-4-5-8ghz-2-dbi-2-4ghz-3-dbi-at-5ghz-rp-sma-rubber-duck-wifi-antenna-13.gif&f=1&nofb=1)



### Directional (Směrová)

Tento typ antény soustředí tok do určitého směru.

#### Sektorová (Patch)

Tato anténa nabízí poměrně dobré pokrytí v jednom směru, typicky používaná v halách nebo na stadionech.

![|900](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fcz.jirous.com%2Fimg%2Fobrazky%2F674219e4d68d744725a2cf8e71ade902.jpg&f=1&nofb=1)
![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fimg.datacomp.sk%2Fmikrotik-mt-rbsxt5hpnd-sektorova-vonkajsia_i264678.jpg&f=1&nofb=1)

#### Yagi

Velmi podobná, jako Patch, s tím rozdílem, že má menší úhly a je tedy shopna šířit signál mnohem dále, stále sektorově.

![|900](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2F2.bp.blogspot.com%2F_VDlpMIP3Jd8%2FTPC1bs32vpI%2FAAAAAAAAADo%2FuTaTAqsYAKg%2Fs1600%2Fb.JPG&f=1&nofb=1)

#### Parabola

Tato anténa využívá fyzikálních vlastností paraboly, díky čemuž je velice směrová, ale také má největší zisky ze všech antén.

![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fes.jirous.com%2Fimg%2Fobrazky%2F7bb6dde65567d24994ffc7725cccff35.jpg&f=1&nofb=1)
![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Faruhaz.alphasonic.hu%2FMIKROTIK_LGH_2-i180171.jpg&f=1&nofb=1)

### Omni (Dual-Directional)

Tento typ antény je speciální, září pouze na 2 strany.
Typicky pro použití v průchodu nebo tunelu.

![|900](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.researchgate.net%2Fprofile%2FNikolaos_Karadimas%2Fpublication%2F270276150%2Ffigure%2Ffig5%2FAS%3A295128519462916%401447375530634%2FOmni-Directional-Antenna.png&f=1&nofb=1)

### Úhly

Typicky více ziskové antény mají i nižší úhly, díky čemuš jsou více ziskové, to ovluvňuje i diagram šíření signálu:

![[RF_Degree.png|900]]

### Frekvence

Většina antén je také specificky testována na frekvenční rozhraní, nelze tedy použit dipólovou anténu pro pozemní vysílání a použít ji na `2.4GHz` spoj.



