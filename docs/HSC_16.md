# Jazyk VHDL - knihovny, entita – porty a generické parametry, architektura – deklarační část, tělo, přehled paralelních příkazů, proces a citlivostní seznam

**VHDL** – strukturovaný, hierarchický, paralelní i impretativní (sekvenční), silně typově orientovaný(předcházení chybám, kontrola během kompilace), není case sensitive

## Knihovny

- Package
    - hierarchicky nad entitou a architekturou 
    - nelze uvést deklaraci entity nebo architektury
    - pro opakované použití kódu napříč designem
    - deklarační část (package) – hlavičky funkcí, procedur, konstant, signálů
    - tělo (package body) – definice funkcí,  
    - př. std_logic_1164(std_logic_vector, vícehodnotová(9) logika, logické operace, konverzní funkce), numeric_std(aritmetické a logické operace)
    - USE library_name.package_name.element(all) – všechny části package
- Library
    - Obsahuje části kompilovaného VHDL kódu
    - Může obsahovat entitu, architekturu, package
    - Pracovní knihovna – ukazatel do pracovního adresáře, kde jsou používané soubory
    - LIBRARY library_name;

**Package a knihovna se vztahují k nejbližší entitě, pro další použití je nutné příkazy LIBRARY a USE opakovat**

## Entita

- Definice hierarchického obvodu – černá skříňka, nepopisuje funkci obvodu
- Tělo architektury obsahuje
    - Deklarační část – popis vnějších signálů entity (IN, OUT, INOUT, BUFFER)
    - Seznam generických konstant  
        - př. GENERIC (Loop : integer := 3);
        - obdoba portů, ale nepředstavuje žádný signál
        - obdoba konstanty (viditelná v entitě a přiřazených architekturách)
        - používá se jako parametr (neměnící se v čase) – lepší čitelnost
- Funkce
    - Podprogram, co vrací hodnotu
    - Mají vstupní operandy a jeden výstupní
    - Nemohou zde být deklarovány signály
    - Lze deklarovat konstanty a proměnné, inicializují se při každém volání 
- Procedury
    - Podprogram, co modifikuje vstupní parametry (IN,OUT,INOUT)
    - Lze předávat konstanty, proměnné i signály, lze z ní volat další procedur

![Obrázek](pics/hsc16_1.png)

## Architektura

- Popis chování entity, každá entita má alespoň jednu architekturu
- Obsahuje 
    - Deklarační část – Definice signálů …  
    PORT ( vstup : IN BIT ;  
    a1, b1 : IN BIT_VECTOR (3 DOWNTO 0) ;  
    vystup : OUT BIT ) ;
    - Příkazovou část  - uzavřeno do BEGIN - END
- Možný popis na různých úrovních abstrakce
    - **Strukturní** – prvky a jejich propojení
    - **RTL** - Register Transfer level-neznáme přesně zapojení obvodu, ale známe jeho funkci, v rámci syntézy se převede RTL na strukturální netlist (zapojení)
    - **Behaviorální** – popis chování obvodu - registry, stavové aut., test. budicí soubory

![Obrázek](pics/hsc16_2.png)

## Paralerní příkazy

- Souběžně vykonávané(dataflow) – vykonání trvá nulovou dobu
- Přiřazení
    - Nepodmíněné 
    - Podmíněné 
    - Výběrové (obdoba case, ale nesekvenční)
- Strukturní
    - Generující příkaz – nemá sekvenční charakter
        - Generuje kod podle poskytnutého schéma
        - Schéma může být podmínka IF nebo FOR
    - Instance entity, komponenty – vloží do kodu blok definovaný v jiném souboru
        - Je-li kod ve VHDL – instance entity
        - Jiná forma – deklarace a instance komponenty
- Simulační – Assert, Report 
- Pro sekvenční zpracování (behavioral)
    - Proces

![Obrázek](pics/hsc16_3.png)

## Process

- Pro popis sekvenčních dějů
- Obsah se vykonává sekvenčně ale spouštění více procesů je paralelní
- Spuštěn při změně objektu v citlivostním seznamu, dá se tam dát i all
- Přiřazení <= proběhnou po dokončení procesu nebo příkazem WAIT 
- Přiřazení := proběhne okamžitě, proměnné jsou vidět jen v procesu

![Obrázek](pics/hsc16_4.png)