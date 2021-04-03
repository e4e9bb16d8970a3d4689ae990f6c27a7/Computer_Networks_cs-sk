# Rušení

## UTP

Provádí se měření několika hodnot, aby se zjistil stav kabelu.
A jeho použitelnost, síťové rozvody by měli být certifikované.

-   **NEXT**
	-   Přeslech na stejném konci sousedního páru
	-   Power Sum NEXT (PS NEXT)
	-   Přeslech mezi všemy páry
-   **FEXT**
	-   Přeslech na druhém konci sousedního páru
-   **Alien Crosstalk**
	-   Přeslech mezi jednotlivými kabely
-   **Delay Skew**
	-   Vzhledem k tomu, že jsou páry krouceny, jsou i jinak dlouhé
	-   Tato rozdílná délka může zapříčinit zpoždění signálu a tak ho narušit
	-   Vše po 25 ns je výborné, pod 50 ns je akceptovatelné
-   **Propagation Delay**
	-   Maximální doba přenosu přes médium
![[FextNext.png]]

## Optika
-   **Polarizační vidová disperze (PMD)**
	-   Rozdíl mezi rychlostí signálu jednoho vlákna a druhého kvůli různým odrazům
-   **Chromatická disperze**
	-   Rozdílná rychlost na základě rozdílné rychlosti šíření světla v látce pro různé vlnové délky