# Základní IP jádra s návazností na HW FPGA, obvody hodin, čítače/časovače, GPIO

## IP jádro (Intellectual Property Core)

Je to analogie softwarové knihovny pro hardware. Představuje předpřipravený blok logiky (např. řadič paměti, procesor, komunikační rozhraní), který vkládáme do návrhu v FPGA jako „součástku“.

AMD dodává společně s Vivado řadu jader zdarma
- Infrastruktura SoC – hodiny, sběrnice, CPU, mailbox, mutex
- Základní komunikace – GPIO, IIC, SPI, CAN
- Rychlé komunikace - ETH, USB, PCIE,…
- Speciální jádra – Inference NN, rekonfigurace, …
- Některá jádra nelze rozumně provozovat bez SW stack (ETH, USB) nebo OS
- Některá jsou zatížena licencí (10G ETH, H265Encoder,…)

## Obvody hodin (Clocking Wizard)

V FPGA ke správě a generování hodinových signálů slouží IP jádro Clocking Wizard, které konfiguruje hardwarové bloky PLL (Phase-Locked Loop) nebo MMCM (Mixed-Mode Clock Manager).

### Princip PLL (Digitálně řízený fázový závěs)

- Základem je zpětnovazební smyčka, která srovnává fázi a frekvenci vstupu s výstupem.
- **Hlavní části**:
    - **PFD (Phase Frequency Detector)**: Porovnává vstupní hodiny se zpětnou vazbou.
    - **CP (Charge Pump) + LF (Loop Filter)**: Převádí rozdíl fází na napětí pro řízení oscilátoru.
    - **VCO (Voltage Controlled Oscillator)**: Generuje frekvenci na základě vstupního napětí.
    - **Děličky (D, M, O)**: Umožňují násobení a dělení frekvence.
- Vzorec pro výstupní frekvenci:  
    $$ F_{VCO} = F_{CLKIN} \times \frac{M}{D} $$
    $$ F_{OUT} = F_{CLKIN} \times \frac{M}{D \times O} $$

    - Kde je násobitel (ve zpětné vazbě), je vstupní dělička a je výstupní dělička.

- Vlastnosti:
    - Syntéza frekvence, fázový posuv
    - Až 7 hodinových výstupů s rozdílnou frekvencí a fází
    - Možnost rekonfigurace za běhu
    - Obsahuje signál **LOCKED**, který CPU informuje, že frekvence je stabilní.
- Limitace
    - Vstupní hodiny <10, 800>MHz
    - Frekvence VCO <5,1200> MHz
    - M <2,128> po inkrementu 0.125
    - D <1, 106>

## Systémový Reset (Processor System Reset)

**Processor System Reset** je IP jádro, které v FPGA **řídí a synchronizuje všechny resetové signály** v procesorovém systému.  
Zajišťuje, že se systém spustí **až po stabilizaci hodin** (kontroluje např. locked z PLL/MMCM). Kombinuje různé zdroje resetů (externí, softwarové, debug) a vytváří z nich **bezpečné synchronizované resety** pro procesor, periférie i sběrnice.  
Drží reset aktivní, dokud nejsou všechny podmínky splněny, aby nedošlo k chybám při startu. Celkově slouží jako **centrální, spolehlivý generátor resetů** pro celý procesorový subsystém ve FPGA.

**Sekvence startu**: Aby systém naběhl bezpečně, resetuje se postupně: Sběrnice Interconnect Periferie (po 16 cyklech) Procesor (po dalších 16 cyklech) .

## Čítače/časovače

Jedná se o univerzální programovatelný blok pro měření času nebo generování událostí.
- Konfigurace HW
    - Až dva 32bit čítače nebo kaskádní 64bit režim
    - Polarita generovaných signálů; Polarita spouští
- **Základní režimy**:
    - **Generate (Generující)**: Počítá do nastavené hodnoty a pošle na výstup 0/1 dle nastavení (nebo cyklicky v režimu Auto-reload).
    - **Capture (Snímací)**: Funguje jako stopky. Uloží aktuální hodnotu čítače v momentě, kdy přijde externí signál (Capture Trig).
    - **PWM (Pulzní šířková modulace)**: Využívá dva čítače – první určuje frekvenci (periodu) a druhý střídu (plnění/duty cycle) signálu.
    - **64bit**: HW musí obsahovat oba čítače
- **SW Obsluha**: Ovladač XTmrCtr. Funkce jako XTmrCtr_Initialize, _Start, _Stop, _GetCaptureValue

## GPIO (General Purpose Input/Output)

Základní rozhraní pro ovládání pinů. Každý pin může fungovat jako vstup nebo výstup, což vyžaduje speciální hardwarové řešení pro obousměrnou komunikaci.

Jednosměrná komunikace
- Primitivní schéma
    - výstupní buffer živí drát
    - vstupní buffer obdobně živí vnitřek obvodu
- nelze ale zároveň tedy poslouchat a vysílat, aby byla možná obousměrná komunikace musíme realizovat jeden komunikační prvek pro poslouchání a druhý pro vysílání
- použitelné pro některé sériové komunikace: SPI, UART

Třístavový budič
- je řešení jednosměrné komunikace, realizuje mechanizmus odpojení budiče pomocí tranzistoru tranzistoru na vstupu EN
- Je základem GPIO v FPGA
- Má logicky tři stavy:
    - l.0, l.1 a vysoká impedance (High-Z) kdy je budič elektricky odpojen a pin může sloužit jako vstup
- To umožňuje **obousměrnou komunikaci po jednom vodiči** (v časovém multiplexu) – obě strany se střídají v tom, kdo mluví a kdo poslouchá. K řízení směru je nutný **handshake**

AXI GPIO
- Přímé ovládání vstupů a výstupů, základem je třístavový budič
- HW volby:
    - Počet kanálů, Povolení přerušení, Šířka kanálů, Směr (IO, I, O)
- Ovládací registry jsou přímo připojené k třístavovému budiči
    - **DATA Register**: Hodnota dat (to, co čteme nebo zapisujeme).
    - **TRI Register**: Řídí signál EN (směr toku).  Log. 0 = Výstup aktivní, Log. 1 = Výstup odpojen (Input / High-Z).
- **SW obsluha**
- **xparameters.h**: Obsahuje definice a ID všech jader (např. XPAR_AXI_TIMER_0_DEVICE_ID). 
- **Ovladače (Drivers)**: C funkcepro ovládání HW (např. XGpio_SetDataDirection pro zápis do TRI registru)