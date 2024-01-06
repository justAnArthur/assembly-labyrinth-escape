# Úloha 23 -- Artur Kozubov

  -----------------------------------------------------------------------
                    1                 2                 3
  ----------------- ----------------- ----------------- -----------------
  1                                                     

  2                                                     

  3                                                     

  4                                                     

  5                                                     

  6                                                     

  7                                                     

  8                                                     
  -----------------------------------------------------------------------

## Zadanie

Napíšte program, ktorý bude simulovať pohyb hráča v bludisku podľa
obrázka. Ľavé horné políčko bludiska má súradnice (riadok, stĺpec) =
(1,1) a pravé spodné políčko má súradnice (8,3). V bludisku sa
nachádzajú nepriechodné steny, vyznačené hrubou čiarou. Hráč môže
začínať hru na ktoromkoľvek políčku a môže vykonávať kroky o 1 políčko
smerom na niektorú svetovú stranu. Hráč sa môže pokúsiť aj o krok smerom
do steny, ale jeho súradnice sa v takomto prípade nezmenia. Z bludiska
vedie jeden východ na jeho okraji.

V pamäti údajov (PÚ) uchovávajte riadkovú súradnicu hráča na adrese
**a0h** a stĺpcovú na adrese **b0h**. Od adresy **0h** so 4-bajtovými
rozostupmi (4h, 8h, ch, 10h, 14h, 18h, 1ch, 20h, atď.) bude pred
spustením programu v pamäti údajov uložená postupnosť hodnôt
reprezentujúcich pohyby hráča o 1 políčko nasledovne:

-   **1h** -- pohyb hore,

-   **2h** -- pohyb vpravo,

-   **3h** -- pohyb dole,

-   **4h** -- pohyb vľavo,

-   **0h** -- koniec.

Po načítaní hodnoty **0h** sa program ukončí. Môžete predpokladať, že
v postupnosti sa iné čísla ako **0h-4h** nebudú nachádzať.

## Riešenie

### Nápad

  ------------------------------------------------------------------------------
  **[X]{.smallcaps}**\   1               2                   3
  **[Y]{.smallcaps}**                                        
  ---------------------- --------------- ------------------- -------------------
  1                      9               8                   12

  2                      1               0                   4

  3                      1               0                   4

  4                      1               0                   4

  5                      1               0                   4

  6                      3               6                   5

  7                      8               12                  5

  8                      3               2                   6
  ------------------------------------------------------------------------------

Ide o to, aby sa do každej bunky zapísalo desatinné číslo, ktoré v
binárnom kóde bude reprezentovať túto logiku:
**[(hore.vpravo.dole.vľavo)]{.smallcaps}**. každá cifra bude uchovávať
pamäť o tom, či v danom smere existuje stena **[(1)]{.smallcaps}** alebo
nie **[(0)]{.smallcaps}**. Potom to všetko prevedieme do desiatkovej
sústavy a zapíšeme do registra **[(64h + X.Y)]{.smallcaps}**. Šifrovaná
mapa je zobrazená nižšie

#### Príklad kódovania poľa

  -------------------------------------------------------------------------
             1                    2                    3
  ---------- -------------------- -------------------- --------------------
  1          1001                 1000                 1100

  2          0001                 0000                 0100

  3          0001                 0000                 0100

  4          0001                 0000                 0100

  5          0001                 0000                 0100

  6          0011                 0110                 0101

  7          1000                 1100                 0101

  8          0011                 0010                 0110
  -------------------------------------------------------------------------

-   

*Prečo?*

Jednoduché zistenie, či sa v určitej koordináte nachádza stena.
Napríklad sa nachádzame v bode\
(1h, 1h) **[(1001)]{.smallcaps}**, ak chceme zistiť, či je hore stena,
použijeme *AND* 8h **[(1000)]{.smallcaps}**: 1001 AND 1000 = 1h - stena
existuje.

### Pamäť programu

+-----+--------+---------------+--------------------------------------+
| Adr | Label  | Inštrukcia    | Komentár                             |
| esa |        |               |                                      |
+=====+========+===============+======================================+
| 0h  | LOADX  | LW            | načítame začiatočnú riadkovú         |
|     |        | \             | **[X]{.smallcaps}** súradnicu z PÚ   |
|     |        | $11,0070(\$0) | z adresy *70h* do registra *R11*     |
+-----+--------+---------------+--------------------------------------+
| 4h  | LOADY  | LW            | načítame začiatočnú stĺpcovú         |
|     |        | \             | **[Y]{.smallcaps}** súradnicu z PÚ   |
|     |        | $12,0080(\$0) | z adresy *80h* do registra *R12*     |
+-----+--------+---------------+--------------------------------------+
| 8h  | [L     | LW            | do registra *R14* načítame prvok     |
|     | OADV]{ | \$            | postupnosti z PÚ z adresy, na ktorú  |
|     | .mark} | 14,0000(\$15) | ukazuje ukazovateľ v registri *R15*  |
|     |        |               |                                      |
|     |        |               | (0р je počiatočná)                   |
+-----+--------+---------------+--------------------------------------+
| ch  | [      | ADDI          | zväčšíme ukazovateľ v registri *R15* |
|     | INCS]{ | \             | o *4h*, aby ukazoval na ďalší prvok  |
|     | .mark} | $15,\$15,0004 | postupnosti v poradí                 |
+-----+--------+---------------+--------------------------------------+
| 10h | CH     | BEQ           | kontrola súradníc (na výstup z       |
|     | ECKING | \$11,\$0      | labyrintu), či súradnica nie je mimo |
|     |        | ,[WIN]{.mark} | hraníc mapy *3 x 9*                  |
|     |        |               |                                      |
|     |        |               | ak áno - skok na *WIN*, aby sa       |
|     |        |               | zapísalo (exit found) a potom        |
|     |        |               | nasleduje *HALT*, ktorý zapíše       |
|     |        |               | konečné súradnice späť do pamäte     |
+-----+--------+---------------+--------------------------------------+
| 14h |        | BEQ           |                                      |
|     |        | \$12,\$0      |                                      |
|     |        | ,[WIN]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| 18h |        | BEQ           |                                      |
|     |        | \$11,\$4      |                                      |
|     |        | ,[WIN]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| 1ch |        | BEQ           |                                      |
|     |        | \$12,\$9      |                                      |
|     |        | ,[WIN]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| 20h | P      | MUL           | vytvorenie ukazovateľa pomocou       |
|     | OINTER | \             | súradníc                             |
|     |        | $13,\$11,\$10 |                                      |
|     |        |               | **[(64h + X.Y)]{.smallcaps}**,       |
|     |        |               | uloženie a prečítanie kódovanej      |
|     |        |               | bunky na registra *\$13*             |
+-----+--------+---------------+--------------------------------------+
| 24h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 28h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 2ch |        | ADD           |                                      |
|     |        | \             |                                      |
|     |        | $13,\$13,\$12 |                                      |
+-----+--------+---------------+--------------------------------------+
| 30h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 34h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 38h |        | MUL           |                                      |
|     |        | \$13,\$13,\$4 |                                      |
+-----+--------+---------------+--------------------------------------+
| 3ch |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 40h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 44h |        | LW            |                                      |
|     |        | \$            |                                      |
|     |        | 13,0064(\$13) |                                      |
+-----+--------+---------------+--------------------------------------+
| 48h | MOVING | BEQ           | ak je načítaný prvok postupnosti     |
|     |        | \$14,\$       | v reg. *R14* rovný 1\                |
|     |        | 1,[UP]{.mark} | (konštantu 1 máme uloženú v reg.     |
|     |        |               | *R1*)\                               |
|     |        |               | skoč na podprogram pre vykonanie     |
|     |        |               | pohybu hore\                         |
|     |        |               | ktorý sa nachádza na libely *„UP"*   |
+-----+--------+---------------+--------------------------------------+
| 4ch |        | BEQ           | ak je načítaný prvok postupnosti     |
|     |        | \$14,\$2,[    | v reg. *R14* rovný 2\                |
|     |        | RIGHT]{.mark} | (konštantu 2 máme uloženú v reg.     |
|     |        |               | *R2*)\                               |
|     |        |               | skoč na podprogram pre vykonanie     |
|     |        |               | pohybu vpravo\                       |
|     |        |               | ktorý sa nachádza na libely „RIGHT"  |
+-----+--------+---------------+--------------------------------------+
| 50h |        | BEQ           | ak je načítaný prvok postupnosti     |
|     |        | \$14,\$3,     | v reg. *R14* rovný 3\                |
|     |        | [DOWN]{.mark} | (konštantu 3 máme uloženú v reg.     |
|     |        |               | *R3*)\                               |
|     |        |               | skoč na podprogram pre vykonanie     |
|     |        |               | pohybu dole\                         |
|     |        |               | ktorý sa nachádza na libely „DOWN"   |
+-----+--------+---------------+--------------------------------------+
| 54h |        | BEQ           | ak je načítaný prvok postupnosti     |
|     |        | \$14,\$4,     | v reg. R22 rovný 4\                  |
|     |        | [LEFT]{.mark} | (konštantu 4 máme uloženú v reg.     |
|     |        |               | R4)\                                 |
|     |        |               | skoč na podprogram pre vykonanie     |
|     |        |               | pohybu vľavo\                        |
|     |        |               | ktorý sa nachádza na libely „LEFT"   |
+-----+--------+---------------+--------------------------------------+
| 58h |        | BEQ           | a skočíme niekam na koniec programu  |
|     |        | \$0,\$0,      |                                      |
|     |        | [HALT]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| \   |        |               |                                      |
| ... |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 70h | [UP]{  | ANDI          | uložiť výsledok, či je hore stena ?  |
|     | .mark} | \             |                                      |
|     |        | $13,\$13,0008 |                                      |
+-----+--------+---------------+--------------------------------------+
| 74h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 78h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 7ch |        | BNEQ          | ak áno, nemôžeme sa už pohnúť hore,  |
|     |        | \$13,\$0,[    | takže ideme na [ďalší prvok]{.mark}  |
|     |        | LOADV]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| 80h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 84h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| 88h |        | SUBI          | ak nie, posuň sa hore (zmenši        |
|     |        | \             | riadkovú súradnicu o 1)              |
|     |        | $12,\$12,0001 |                                      |
+-----+--------+---------------+--------------------------------------+
| 8ch |        | BEQ           | a ideme na [ďalší prvok]{.mark}      |
|     |        | \$0,\$0,[     |                                      |
|     |        | LOADV]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| \   |        |               |                                      |
| ... |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| a0h | [R     | ANDI          | uložiť výsledok, či je vpravo stena? |
|     | IGHT]{ | \             |                                      |
|     | .mark} | $13,\$13,0004 |                                      |
+-----+--------+---------------+--------------------------------------+
| a4h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| a8h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| ach |        | BNEQ          | ak áno, nemôžeme sa už pohnúť        |
|     |        | \$13,\$0,[    | vpravo, takže ideme na [ďalší        |
|     |        | LOADV]{.mark} | prvok]{.mark}                        |
+-----+--------+---------------+--------------------------------------+
| b0h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| b4h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| b8h |        | ADDI          | ak nie, posuň sa vpravo (zväčši      |
|     |        | \             | stĺpcovú súradnicu o 1)              |
|     |        | $11,\$11,0001 |                                      |
+-----+--------+---------------+--------------------------------------+
| bch |        | BEQ           | a ideme na [ďalší prvok]{.mark}      |
|     |        | \$0,\$0,[     |                                      |
|     |        | LOADV]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| \   |        |               |                                      |
| ... |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| d0h | [      | ANDI          | uložiť výsledok, či je dole stena ?  |
|     | DOWN]{ | \             |                                      |
|     | .mark} | $13,\$13,0002 |                                      |
+-----+--------+---------------+--------------------------------------+
| d4h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| d8h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| dch |        | BNEQ          | ak áno, nemôžeme sa už pohnúť dole,  |
|     |        | \$13,\$0,[    | takže ideme na [ďalší prvok]{.mark}  |
|     |        | LOADV]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| e0h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| e4h |        | NOP           |                                      |
+-----+--------+---------------+--------------------------------------+
| e8h |        | ADDI          | ak nie, posuň sa dole (zväčši        |
|     |        | \             | riadkovú súradnicu o 1)              |
|     |        | $12,\$12,0001 |                                      |
+-----+--------+---------------+--------------------------------------+
| ech |        | BEQ           | a ideme na [ďalší prvok]{.mark}      |
|     |        | \$0,\$0,[     |                                      |
|     |        | LOADV]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| \   |        |               |                                      |
| ... |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   | [      | ANDI          | uložiť výsledok, či je vľavo stena ? |
| 00h | LEFT]{ | \             |                                      |
|     | .mark} | $13,\$13,0001 |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | NOP           |                                      |
| 04h |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | NOP           |                                      |
| 08h |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | BNEQ          | ak áno, nemôžeme sa už pohnúť vľavo, |
| 0ch |        | \$13,\$0,[    | takže ideme na [ďalší prvok]{.mark}  |
|     |        | LOADV]{.mark} |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | NOP           |                                      |
| 10h |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | NOP           |                                      |
| 14h |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | SUBI          | ak nie, posuň sa vľavo (zmenši       |
| 18h |        | \             | stĺpcovú súradnicu o 1)              |
|     |        | $11,\$11,0001 |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | BEQ           | a ideme na [ďalší prvok]{.mark}      |
| 1ch |        | \$0,\$0,LOADV |                                      |
+-----+--------+---------------+--------------------------------------+
| \   |        |               |                                      |
| ... |        |               |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   | [WIN]{ | SW            |                                      |
| 40h | .mark} | \$1,0060(\$0) |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   | [      | SW            |                                      |
| 44h | HALT]{ | \             |                                      |
|     | .mark} | $11,0070(\$0) |                                      |
+-----+--------+---------------+--------------------------------------+
| 1   |        | SW            |                                      |
| 48h |        | \             |                                      |
|     |        | $12,0080(\$0) |                                      |
+-----+--------+---------------+--------------------------------------+

### 

### Register 

  ----------------------------------------------------------------------------------
  Register        Údaj                     Komentár
  --------------- ------------------------ -----------------------------------------
  R1 - R4, R9     1h-4h, 9h                pre porovnanie

  R10             ah                       na vytvorenie ukazovateľa

  R11, R12        **[\...]{.smallcaps}**   **[X, Y]{.smallcaps}**

  R14                                      sem sa bude načítavať prvok postupnosti
                                           z pamäte údajov

  R15                                      ukazovateľ do postupnosti prvkov, na
                                           začiatku ukazuje na 1. prvok
  ----------------------------------------------------------------------------------

![Table Description automatically
generated](media/image1.png){width="1.7418175853018372in"
height="3.641982720909886in"}

### Data memory

+-------------------+--------------------------------------------------+
| Register          | Komentár                                         |
+===================+==================================================+
| [0h --            | postupnosť krokov                                |
| 5ch]{.mark}       |                                                  |
+-------------------+--------------------------------------------------+
| [60h]{.mark}      | výsledok, exit alebo nie na konci programu       |
+-------------------+--------------------------------------------------+
| [70h, 80h]{.mark} | **[X, Y]{.smallcaps}**                           |
+-------------------+--------------------------------------------------+
| [90h --           | ## **[zašifrovaná karta]{.smallcaps}**           |
| fch]{.mark}       |                                                  |
+-------------------+--------------------------------------------------+

![Table Description automatically
generated](media/image2.png){width="3.2916666666666665in"
height="3.533333333333333in"}

### Nasimulujme postupnosť

#### 1. 

Počiatočný bod: ( 1, 1 ) **[( X, Y )\
]{.smallcaps}**Postupnosť: 1 2 2 2 2 3 3 3 3 3 3 3 3 3 1 3 4 4 1 4 2
0**[\
]{.smallcaps}**

#### Pamäť údajov pred spustením programu

**[\*presne rovnaké ako pôvodné (vyššie)]{.smallcaps}**

### Pamäť údajov po skončení programu

![Table Description automatically
generated](media/image3.png){width="1.7418175853018372in"
height="3.641982720909886in"} ![Table Description automatically
generated](media/image4.png){width="3.2919520997375327in"
height="3.533639545056868in"}

Tak sa našlo východisko a výsledkom je jeden.

## Záver

Tento typ šifrovania sa dá použiť aj vtedy, keď máme k dispozícii viac
možností pohybu, napríklad súradnicu Z, a je tiež vhodné a jednoduché
šifrovať a dešifrovať informácie, ktoré potrebujeme.

Tento spôsob zápisu čísel do registra (\[offset\].X.Y) - nás obmedzuje
vo veľkosti mapy 9 na 9, pretože nie je možné zapisovať do registra
(\[offset\].X.12), pretože súradnica musí byť jednociferná.
