# Network Transfer Protocols
---

Často potřebujeme přesouvat data po síti, nejčastěji například zálohovat konfiguraci nebo stáhnout nový IOS.
Obecně se k těmto účelům používá příkaz `copy <PROTOCOL><PATH>`, v některých případech ale musíme nastavit i věci jako autentizaci.

## [[FTP]]

```
R(config)#ip ftp username <USERNAMe>     \\ Nastavení přihlašovacího jména
R(config)#ip ftp password <PASSWORD>     \\ Nastavení přihlašovacícho hesla 
```

```
R(config)#ip ftp source-interface <IF>     \\ Specifikace zdrojového interfacu a IP adresy
```

```
R(config)#ip ftp passive \\ Nastavení pasivního režimu
```

### [[FTP#TFTP|TFTP]] Server

TFTP server se vytvoří jednodušše, stačí specifikovat soubor, který chcete sdílet.

```
R(config)#tftp-server <PATH> alias <NAME> {ACL}     
```

## [[SCP]]

```
R#copy scp://<USER>@<IP>/<PATH> <LOCAL_PATH>
```

Pro vytvoření SCP serveru je nutné povolit [[AAA]] a nastavit přihlašovací údaje, následně ho lze povolit jedním příkazem.

```
R(config)#ip scp server enable
```
