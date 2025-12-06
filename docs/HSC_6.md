# Zynq PS

**Zynq-7000 APSoC** je hybridní čip kombinující procesorový systém a programovatelnou logiku FPGA (SoC Zynq). PS (Processing System) je v čipu Zynq Hard Macro. To znamená, že je to fyzicky vyrobený procesor, který je neměnný a funguje nezávisle na tom, zda je naprogramovaná FPGA část.

## Charakteristika a architektura

Srdcem systému je **Application Processing Unit (APU)**, která obsahuje:

- **Jádra**: Dual-core ARM Cortex-A9 MPCore.
- **Architektura**: ARMv7 (32-bit instrukce, podpora Thumb-2).
- **Výkon**: Až 1 GHz (typ. 667 MHz), cca 2.5 DMIPS/MHz na jádro.
- **Vlastnosti vykonávání**:
    - **Superskalární**: Procesor dokáže načíst a spustit více instrukcí (až 2) v jednom hodinovém taktu, což zvyšuje rychlost zpracování.
    - **Out-of-Order execution**: CPU může měnit pořadí instrukcí. Pokud jedna instrukce čeká na pomalou paměť, CPU „předběhne“ a vykoná jinou instrukci, která je připravená. Má proměnnou délku pipeline.
    - **Predikce skoků**: Procesor se snaží odhadnout, kam program skočí (např. v podmínce if), a přednačte si instrukce dopředu, aby se nezastavil. Dynamická (pomocí 2bit větví) i statická.

## Charakteristika vektorové jednotky

Každé jádro má vlastní koprocesory pro specializované výpočty, které by hlavní CPU zdržovaly.

- **NEON (Media Processing Engine)**: 128-bitová **SIMD** (Single Instruction Multiple Data) jednotka.
    - Umožňuje zpracovat více dat jednou instrukcí (např. 2x64bit, 4x32bit, 16x8bit).
    - Podpora signed i unsigned celočíselných formátů
    - Podpora polynomů nad dvojkových tělesem (kryptografie) 
    - Využití: DSP, multimédia, maticové operace.
    - Obsahuje 32 samostatných registrů (32x64, 32x32, 16x128).
- **VFPv3 (FPU)**: Je hardwarová jednotka pro počítání s desetinnými čísly (plovoucí řádová čárka). 

## Paměťový subsystém

Paměťový systém Zynqu je hierarchický a zahrnuje vše od rychlých cache až po řadiče externích pamětí.

- **MMU (Memory Management Unit)**: Hardwarová podpora pro **virtuální paměť**. Překládá virtuální adresy na fyzické, což je nezbytné pro běh operačních systémů jako Linux
- **Low Memory (< 1GB)**: Zde se nachází **OCM (On-Chip Memory, rychlá SRAM)** a hlavní operační paměť **DDR**.
- **PL Adresy (M_AXI_GP)(<1GB,3GB>)**: Dva obrovské bloky adres vyhrazené pro komunikaci s **FPGA**. 
    - Když procesor zapíše data na tyto adresy, signál projde přes můstky (AXI GP porty) přímo do vašeho vlastního hardwaru v FPGA. Takto ovládáte své IP bloky.
- **<3GB,4GB>**
- **I/O Periferie**
    - **IOP Registers**: Nastavení periferií jako USB, Ethernet, UART.
    - **SMC Memories**: Přímý přístup k externím Flash pamětem (NAND/NOR).
- **Systémové a Privátní registry**
    - **SLCR (System Level Control Registers)**: nastavení hodin, resetů a multiplexingu pinů (MIO).
    - **CPU Private**: Registry soukromé pro každé jádro (časovače, přerušení).	
- **< 4G – 256KB, 4G>** - zde je znovu mapována **OCM** - Slouží jako rychlá odkládací paměť na konci adresního prostoru, kterou procesor vidí vždy, bez ohledu na nastavení DDR.

## Cache

**je malá, velmi rychlá paměť**, která leží přímo u procesoru a slouží k urychlení přístupu k často používaným datům nebo instrukcím.

**Hierarchie Cache a Koherence**:
- **L1 Cache (Privátní)**: Každé jádro má vlastní **32 KB pro instrukce** a **32 KB pro data**. Funguje na principu Harvard (oddělená data a kód).
- **L2 Cache (Sdílená)**: Větší **512 KB** paměť společná pro obě jádra.
- **SCU (Snoop Control Unit)**: Řadič, který zajišťuje **cache koherenci** (aby obě jádra viděla stejná data) a propojuje jádra s L2 cache a pamětí.


## Periferie

PS obsahuje sadu "tvrdých" (Hard IP) periferií, které se nekonfigurují v FPGA.

- **Sada periferií** (vždy po dvojici): USB 2.0, Gigabit Ethernet (GigE), SDIO, UART, SPI, I2C, CAN, GPIO.
- **Multiplexing (MIO vs. EMIO)**: Protože čip nemá dostatek fyzických pinů pro všechny periferie najednou, používá se multiplexing:
    - **MIO (Multiplexed I/O)**: 54 vyhrazených pinů přímo z PS. Nejkratší cesta, nezabírá logiku FPGA.
    - **EMIO (Extended MIO)**: Pokud nestačí MIO piny, lze signály periferií přesměrovat do FPGA a vyvést je na jeho libovolné piny.

## Propojení (Interconnect)

- Všechny tyto části (CPU, Paměti, Periferie, FPGA) jsou propojeny systémem sběrnic **AMBA AXI**. 
- Využívá se **Central Interconnect** s architekturou Crossbar (přepínač), který propojuje CPU, paměti, periferie a FPGA část.
