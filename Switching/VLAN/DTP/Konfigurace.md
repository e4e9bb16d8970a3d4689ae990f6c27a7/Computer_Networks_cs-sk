[[DTP]]
# Konfigurace

```
SW1(config-if)#switchport nonegotiate     \\ Negeneruje DTP rámce
SW1(config-if)#switchport mode dynamic <auto|desirable>     \\ Přepnutí mezi módy
```

```
SW1#show dtp {IF}     \\ Zobrazení informací
```