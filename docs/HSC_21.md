# HLS – výhody a nevýhody oproti popisu HDL/Model Composer, popis flow, zpracování základních prvků jazyka C do HW struktur, Interface AXI4

## HLS (Vitis) – vysokoúrovňová syntéza

Určujeme, co se má s daty stát (C/C++), ne to, jak se mají přesouvat (HDL)

## Co HLS provádí

Převod C/C++ funkcí do zadrátovaných funkcí (v rámci toho dělá i věci níže) 

- Plánování operací 
    - Aut. Plánování operací na více taktů (některé jsou i vícetaktové) 
    - Rychlé FPGA (malé clk, ale hodně HW) – více operací za jeden takt
    - Je třeba dodržet datové závoslosti – posloupnost operací (+-*/)
- Přiřazování zdrojů – určuje, které zdroje budou použity (na základě plánování)
    - Se sdílením/bez sdílení – jestli musím použít více násobiček (a*b+c*d) 
    - Přiřazování lze ovlivnit – cílovou frekvencí, direktivy (pragmas)
- Extrakce řídicí logiky – smyčka v C/C++ vede na stav. Automat 
    - Dořeší se další souvislosti
        - Latency – počet cyklů nezbytných pro celkový výpočet
        - Initiation interval – počet cyklů před přijetím nových dat
        - Troughput – počet cyklů mezi přijímáním dat
        - Loop latency – počet cyklů jedné smyčky
        - Loop initiation interval – počet cyklů mezi iteracemi smyčky
        - Loop latency – veškerý počet cyklů smyčky

**Zpracování kodu**

- Hlavní funkce
    - Argumenty této funkce vytvoří I/O porty 
    - Direktivy umožnují namísto portu HW protokol (AXIMM,AXIS,BRAM)
- Ostatní funkce – překlad do RTL, voláním funkce vytvořením instanci
- Smyčky 
    - Zabalené a zřetězené (výchozí) – každá iterace používá stejné zdroje
    - Rozbalené – pokud lze jejich meze staticky vyhodnotit – nutné více HW 
- Pole – práce s ním vede k použití BRAM(bloková ram)
    - Pokud to lze a je to výhodné, tak syntéza slučuje a rozděluje pole do větších či menších RAM 

- Problémy
    - Printf(),fscanf, rekurzivní funkce – nelze
    - Dynamicky vytvářené a mazané objekty – nelze, musí být statické
    - Generic pointery – ne

## Základní flow

- Základní C/C++ funkce
- Transformace funkcí pro RTL popis
- Vytvoření testbench
- Verifikace funkce
- Vytvoření RTL popisu
- Verifikace na RTL úrovni
- Vytvoření ovladačů¨
- Export IP jádra

## Interface AXI4

Pomocí direktiv kompilátoru lze argumenty funkce upravit na interface
- Axis(stream), s_axilite(konfigurační registry),m_axi(obsluha DMA – direct memory access controler)

Možnost použít AXI 4 Burst – akce, která pošle více dat na seřazené adresy

- Zavolá se memcpy(), dále to probíhá ve zřetezeném for cyklu
- Požadavky, aby se to dalo použít
    - Smyčky musí být pipeline
    - Adresy seřazené vzestupně
    - Přístup do paměti – bez podmínky
    - Pouze 1 read nebo 1 write v cyklu pro stejný port

## Výhody oproti HDL/Model Composer

- Algoritmický popis řešení je rychlejší než návrh HW- neřešíme strukturu 
    - 100 řádků kodu C může být 5000 řádků VHDL
- Můžeme využít hardwarovou akceleraci – úkoly dělá kromě cpu i spec. HW
- Oddělená funkcionalita (C/C++) od reálné implementace (direktivy to specifikuju)
- Rychlejší testování funkčnosti (C kód)
- Čitelnější kód – práce s poli, strukturami

## Nevýhody oproti HDL
- Nutné převádět  C/C++ kod do HDL jazyka v RTL popisu
- Verifikace ve všech fázích návrhu - zdržování
- Zkušený HDL programátor zabere méňe zdrojů než HLS  (nižší možná frekvence)
- Viz – zpracování kódu – problémy (nesmí se používat konstrukce známe z Cčka)
- Možnost syntaktických chyb (oproti model composer)
