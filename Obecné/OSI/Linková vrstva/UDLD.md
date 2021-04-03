# Unidirectional Link Detection
---

Tato funkce umožňuje monitorování funkce optických kabelů a zjišťování zda nejsou jednosměrné.

Funguje tak, že posílá frame se System ID a Port ID, který musí příchozí switch přesně zopakovat, poslat zpět, pokud vše funguje, port je v *bidirectional* stavu.

- Normal
  - Pokud frame není zopakován port je považován za *undetermined*, ale zůstává *up up*
- Aggressive
  - Pokud switch nedostane odpověď, pošle ještě 8 framů v 1s intervalech
  - Pokud ani poté nedostane odpověď, přepne port do ErrDisabled stavu

Funkce je defautlně povolena v Normal módu na všech SFP-based portech.

```
SW(config)#udld enable {agressive}
SW(config-if)#udld enable {agressive}
```

Zablokované interfacy je nutné ručně restartovat.

```
SW#udld reset
```

Lze zapnout a nakonfigurovat recovery interval, po kterém se pokusí znovu zpřístupnit port, defaultně 5 minut.

```
SW(config)#udld recovery interval     \\ Zapnutí
SW(config)#udld recovery interval <čas>     \\ Nakonfigurování času
```

UDLD musí být nakonfigurováno na obou zařízeních, nastavení lze zkontrolovat pomocí:

```
SW#show udld neighbors
SW#show udld <if>
```
 


