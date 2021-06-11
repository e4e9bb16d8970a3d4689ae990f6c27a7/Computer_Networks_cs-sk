# [[SSH]]
---
Jedná se o šifrovaný a tudíš bezpečnější provoz, který se dnes běžně využívá.

Pro jeho nastavení je potřeba nakonfigurovat několik věci:

- Vygenerovat klíč
  - Změnit defautlní jméno
  - Přidat zařízení do domény
  - Vytvořit uživatele
- Nastavit samotné SSH
- Přiřadit SSH na VTY

```
R(config)#hostname <NAME>
R1(config)#ip domain-name <NAME>     \\ V případě, že nepoužíváme doménu, je doporučeno použít local
R1(config)#username <NAME> {privilige <LEVEL>} secret <PASSWD>     \\ Vytvoření přihlašovacích údajů
R1(config)#crypto key generate rsa modulus <SIZE>     \\ Vygenerování klíče
```

```
R1(config)#ip ssh version 2     \\ Přepnutí na verzi 2, doporučeno
R1(config)#ip ssh authentication-retries <NUMBER>     \\ Nastavení počtu pokusů o připojení před odpojením
R1(config)#ip ssh loggin events     \\ Zapnutí logování
```

```
R1(config)#line vty <MIN> <MAX>     \\ Přepnutí se na VTY interface, MIN a MAX počet určují celkový počet interfaců a tím i připojení
R1(config-line)#transport input ssh     \\ Povolení POUZE SSH
R1(config-line)#login local
```

# [[Telnet]]
---
Jedná se o starý protokol běžící na TCP:23 a je nešifrovaný, z toho důvodu je nedoporučován!

Pro jeho konfiguraci pouze stačí nastavit heslo a jméno.

```
R(config)#username <NAME> {privilige <LEVEL>} secret <PASSWD>     \\ Vytvoření přihlašovacích údajů
```

```
R(config)#line vty <MIN> <MAX>     \\ Přepnutí se na VTY interface, MIN a MAX počet určují celkový počet interfaců a tím i připojení
R(config-line)#login {local}     \\ V případě, že použijete příkaz local, jako přihlašovací údaje se použijí ty z obecného režimu (username secret)
```

V případě použití pouze příkazu login je nutné určit heslo, v tomto případě je poté nutné i nastavit heslo do privilegovaného režimu (`enable secret <PASSWD>`).

```
R(config-line)#password <PASSWD> 
```

## Další

Lze filtrovat na základě [[ACL]].

```
R(config-line)#access-class <ACL> <in | out>
```

Lze nastavit maximální dobu připojení, po uplynutí v případě neaktivity systém uživatele odpojí.

```
R(config-line)#exec-timeout <MIN>
```

Lze nastavit *privilege level*.

```
R(config-line)#privilege level <0-15>
```

Lze spustit script v případě aktivity na VTY.

```
R(config-line)#script <ACTIVATION_TRIGGER> <PATH>
```

Lze vynutit zobrazení banneru.

```
R(config-line)#motd-banner
```