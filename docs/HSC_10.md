# Způsoby obsluhy periferií, polling, interrupt, základní struktura ovladače s jednoduchým příkladem

Procesor může komunikovat s periferiemi a reagovat na jejich stav dvěma základními způsoby: aktivním dotazováním (Polling) nebo pomocí přerušení (Interrupt).

## Polling (Dotazování)

- Jde o nejjednodušší, ale často neefektivní metodu.
- **Princip**: Procesor v nekonečné smyčce aktivně a opakovaně kontroluje stav periferie (např. čte stavový registr, zda přišla data nebo zda je čítač hotov)
- **Modely obsluhy**:
    - Sekvenční: Zkontroluje periferii A -> provede akci, zkontroluje B -> provede akci
    - Hromadný: Zkontroluje všechny periferie a následně provede akce dle zjištěných stavů
- Nevýhody:
    - Vytěžuje CPU neustálým dotazováním, i když se nic neděje
    - Obtížné řešení priorit (pokud obsluha jedné periferie trvá dlouho, druhá musí čekat)
    - blokuje ostatní výpočty

## Interupt (Přerušení)

- Kontrola periferie vyvolána událostí
- **Princip**: Procesor vykonává svůj hlavní program. Jakmile periferie potřebuje pozornost (Timer dopočítal, přišla signál z komunikace, tlačítko atd), pošle signál IRQ (Interrupt Request). CPU pozastaví práci a skočí do speciální funkce obsluhy
- Klíčové pojmy:
    - Exception – výjimka – událost uvnitř CPU která způsobí skok do režimu obsluhy
    - Interrupt – přerušení – žádost o obsluhu vyvolaná změnou stavu ext. signálu
    - IRQ – Interrupt Request – Identifikátor přerušení podle kterého CPU najde správnou obsluhu
    - Interrupt vector table - Tabulka adres, kam má procesor skočit pro konkrétní přerušení
    - Handler(ISR) – Funkce, která vykoná samotnou reakci na událost
    - Callback – Uživatelská funkce volaná z handleru ("rozsviť LED, když Timer dojede")
        - V stm je to třeba když přijde přerušení od časovače a máš tam tu funkci do které píšeš co se má stát, kam to píšeš je handler a co tam píšeš je callback
- **Průběh obsluhy**:
    - Periferie vyvolá Interrupt -> Řadič přerušení obdrží IRQ -> CPU vyvolá Exception -> CPU použije Interrupt Vector Table -> Spustí se Handler (ISR) -> Handler může zavolat Callback ->Návrat z Exception

## Generic Interrupt Controller (GIC)

- Řadič umístěný přímo v procesorovém systému (PS) Zynq
- Podpora přerušení z PL
- Zdroje přerušení:
    - 16 obecný přerušení
    - 2 privátní přerušení
- Nastavení: 
    - **Spoušť**: Náběžná hrana (Rising) nebo Vysoká úroveň (High) .
    - **Priorita**: Lze nastavit důležitost.
- **Poznámka**: Základní API nepodporuje vnořená přerušení.

## AXI Interrupt Controller (AXI INTC)

- Řadič implementovaný v programovatelné logice (FPGA).
- **Použití**: Když v systému není hardwarový procesor nebo nestačí počet IRQ linek .
- **HW Konfigurace (masky)**:
    - **Počet**: Nastavitelný počet vstupních přerušení.
    - **Typ vstupu**: Detailní nastavení maskou – Level/Edge, Rising/Falling, High/Low .
    - **Typ výstupu**: Lze nastavit typ signálu generovaného směrem k procesoru.

## Základní struktura ovladače IP jádra

- **Definice hardwaru (xparameters.h)**
    - Tento soubor je automaticky generovaný podle HW návrhu.
    - Obsahuje unikátní identifikátory pro každou periferii ve formátu:  
    **XPAR_<NÁZEV_INSTANCE>_DEVICE_ID**
- **Řídící struktura (Instance)**
    - Každý ovladač má definovanou C strukturu (např. **struct XGpio, struct XTmrCtr**), která uchovává stav periferie a její základní adresu
    - Je nutné vytvořit proměnnou (instanci) tohoto typu.
- **Inicializace (Initialize)**
    - Funkce, která propojí softwarovou instanci s konkrétním hardwarem pomocí ID.
    - Obecný tvar: **X<TypDriveru>_Initialize(&Instance, DeviceID)**
- **Vlastní práce (API funkce)**
    - funkce pro konfiguraci (např. směr pinu) a provoz (čtení/zápis)
    - Vždy přebírají ukazatel na instanci jako první parametr.

```
// 1. Hlavičkové soubory
#include "xparameters.h" // Obsahuje ID hardwaru
#include "xgpio.h"       // Obsahuje definice driveru (XGpio)
// 2. Vytvoření instance (objektu) ovladače
XGpio GpioInstance;
int main() {
    // 3. Inicializace - propojení instance s HW ID
    XGpio_Initialize(&GpioInstance, XPAR_AXI_GPIO_0_DEVICE_ID);
    // 4. Nastavení a Ovládání
    // Nastavení směru kanálu 1 na výstup (0 = Output) 
    XGpio_SetDataDirection(&GpioInstance, 1, 0x0);
    // Zápis hodnoty (rozsvícení všech LED) 
    XGpio_DiscreteWrite(&GpioInstance, 1, 0xFF);
    return 0;
}
```