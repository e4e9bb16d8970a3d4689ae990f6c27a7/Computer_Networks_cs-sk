[[VTP]]
# Konfigurace
---

## VTPv1 a v2

Musíme vytvořit doménu.

Na stanicích v *Client* módu není nutné nastavovat doménu, switch si ji automaticky se synchronizuje z prvního *VTP* updatu.

```
SW(config)#vtp domain <Doména>
```                                                            

Volitelně heslo, není uloženo v *running-config*.

```
SW(config)#vtp password <Heslo>
```                                                

Mód v jakém operuje.

I přes to, že defaultní mód je *Server*, VTP updaty switch začne posílat až po nastavení domény.
Je doporučeno mít v doméně více, než jeden server.

```
SW(config)#vtp mode {server, client, transparent, off}
```                     

Můžeme nastavit i verzi, 2. verze podporuje Token Ring, VLAN consistency check, unrecognized TLV a v transparentním módu přeposílá advertisements.
Defaultní je verze 1.

```
SW(config)#vtp version <Verze>
```               

## VTPv3

```
SW(config)# vtp version 3   \\ Přepnutí režimu
```
```
SW(config)# vtp password <Heslo> hidden   \\ Nastavení hesla, uložení v encrypted módu
```
```
SW(config)# do vtp primary   \\ Pro přepnutí na Primary server
```

## Informace

```
SW#show vtp status          // základní info o běhu VTP na switchi
```         
```
SW#show vtp counters        // statistika VTP přenosů
```                                      
```
SW#show vtp password        // zobrazí VTP heslo
```


