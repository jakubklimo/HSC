# Embedded systémy, základní charakteristika, typické součásti embedded systémů, ES v ASIC vs FPGA

## Embeded systém

Elektronický systém (kombinace HW a SW) navržený provádět jednu nebo více speciálních funkcí (může být i součástí většího systému - **Integrace**)

Postupná integrace:  
Tranzistory -> Monolitické obvody -> mikroprocesory ->VLSI monolitcké obvody -> Systems on a Chip

Všude: Router, telefon, pračka, mikrovlnka

Funkce se typicky po dobu životnosti výrobku se obvykle nemění

Pracují v reálném čase s definovanou odezvou

Důraz na nízkou spotřebu (často napájené z baterií)

Omezené prostředky (výkon, paměť)  - cena

## Součásti ES

- Procesor
- Vnitřní služby a komunikace
    - Porty procesoru, čítače/časovače 
    - Systémové služby (Interrupt, Watchdog)
    - Interní „sběrnice“
- Paměti
    - Interní SRAM,FLASH
    - Řadiče externí pamětí SRAM/(LP)DDR(2,3)
- Externí komunikace 
    – AD /DA převodníky, USB, UART, Ethernet
- Dedikovaný HW
    - Float procesory, grafika, zvuk,speciální
- SW
    - Založeno na FW, OS

![Obrázek](pics/hsc4_1.png)

## FPGA vs ASIC

U ASIC lze mít mnoho hodinových domén, jsou k dispozici některé základní součásti (sčítačky), je dáno rozložení -> vyšší hustota tranzistorů, vyšší rychlost hodin

FPGA má sdílené zdroje a proměnlivou topologii

### ASIC – aplication specific integrated circuit

navržené pro konkrétní aplikaci – limitovaná funkčnost(-)  
vysoký výpočetní výkon(+)  
nízká spotřeba(+)  
nízká cena(+) - při vyšším počtu kusů  
nelze přeprogramovat(-)  
drahý a dlouhý vývoj(-)  
musí se dělat fyzický návrh(-) -> máme plnou “svobodu” návrhu(+)

### FPGA

nejdříve v továrně vyrobeno a poté to navrhneme -> short time to market  
lze přeprogramovat, rekonfigurovatelné(+)  
ihned k dispozici - ihned ladění - ihned odzkoušení(+)  
nemusí se dělat fyzická návrh - stačí bitstream  
nižší výpočetní výkon(+)  
vyšší spotřeba(-)  
vyšší cena(-)
