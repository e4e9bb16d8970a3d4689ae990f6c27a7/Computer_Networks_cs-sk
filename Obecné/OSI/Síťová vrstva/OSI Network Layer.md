# OSI Network Layer
---

Krom referenčního modelu ISO/OSI, vytvořilo ISO i řadu protokolů ,které měli konkurovat například TCP/IP, řada z nich je dnes používána.
Oproti TCP/IP mají ISO/OSI protokoly rozdílnou terminologii:
- **System** = síťový bod, node
- **IS** (Intermediate System) = router
- **ES** (End System) = koncové zařízení
- **Circuit** = interface
- **Domain** = AS

ISO/OSI také specifikuje 2 typy L3 komunikace:
- **Connection Oriented mode**
	- Pro tento typ není TCP/IP protějšek
	- Využíván u *X.25* protokolu
- **Connectionless Oriented mode**
	- Podobný IP protokolu
	- Nenavazuje L3 spojení
	- *Connectionless mode network protocol* (CLNP)
		- *CLNP* je pro ISO/OSI protokoly jako [[IPv4]] a [[IPv6]] pro TCP/IP
		- Set služeb se nazývá *CLNS*
		- Zde se nachází [[IS-IS]]


# NSAP
---

ISO/OSI používá *Network Service Access Point* (NSAP) adresaci, která má za úkol identifikovat **službu** v síti.
NSAP adresa je pro službu, nebo node, nikoli interface, něco podobného není v TCP/IP síti obvyklé, nejblíže by tomu byla konfigurace, kdy máme pouze jednu [[IPv6#Global Unicast|IPv6 Global Unicast]] adresu na loopback interfacu a na fyzických interfacech pouze [[IPv6#Link-Local|link-local]] adresaci, pokud bychom následně nad loopback interfacy spustili nějaký routing protokol, dosáhneme podobného stavu sítě.

## Datagram

- ***Initial Domain Part (IDP)***
	- **Authority & Format Identifier** (AFI)
		- `1B` `00-FF`
		- Identifikuje zbývajících polí
	- **Initial Domain Identifier** (IDI)
		- Může být vynechán
		- Délku určuje AFI
	- *AFI* a *IDI* určují doménu
- ***Domain Specific Part (DSP)***
	- Závysí na AFI
	- **High Order Domain Specific Part** (HO-DSP)
		- Identifikuje část domény/areu
	- **System ID**
		- Identifikuje node/službu
		- V případě [[IS-IS]] často [[MAC]]
	- **NSAP Selector** (SEL/NSEL)
		- `1B` 
		- Identifikuje službu na/nad L3
			- `22`/`1D` - OSI TP4 Transport Protocol
			- `2F` - GRE IP-over-CLNP
		- `00`
			- Pokud je nastaven na `00` identifikuje celý node, nikoli službu
			- Tento typ se používá u [[IS-IS]]
			- *Network Entity Title* (NET) 

Samotná NSAP adresa je volitelně dělitelná po bytech, lze ji zadat jakkoli, Cisco zařízení si ji ale upraví do defaultní podoby.

```
49.0001.1234.5678.3333.00     \\ defaultní
```

=

```
4900.0112.3456.7833.3300
```

=

```
4900011234567833300
```

```
<AFI>.<IDI>.{HO-DSP}.<System ID>.<SEL>
```

U `AFI=49` se jedná o lokální adresu a *HO-DSP* je na administrátoru a nemusí byt použito.
NSAP adresa jako celek může mít délku mezi `8B` a `20B`.

### AFI

| AFI | Význam                              | IDI  |  HO-DSP  |
|:---:|:----------------------------------- |:----:|:--------:|
| 39  | Data Country Code (ISO 3166)        | `2B` |  `10B`   |
| 45  | Mezinározní telefonní čísla (E.164) | `8B` |   `4B`   |
| 49  | Lokální použití                     | Není | `0B-12B` |


