# Microblaze – charakteristika a architektura CPU, možnosti konfigurace, možnosti cache

**Microblaze** je vysoce konfigurovatelný **Soft-core** procesor od firmy AMD (Xilinx). Není to fyzický čip, ale IP jádro implementované v programovatelné logice (FPGA).

## Charakteristika a architektura CPU

Vychází z architektury **RISC** (Reduced Instruction Set Computer) a je velmi podobná architektuře MIPS.

- **Instrukční sada**: Používá 32-bitové instrukce, pracuje se 3 operandy a má k dispozici 32 obecných registrů.
- **Pipeline (Zřetězené zpracování)**: Délka pipeline je **konfigurovatelná**, což ovlivňuje výkon a spotřebu místa v FPGA:
    - **3 stupně**: Šetří místo (Logiku), ale má nižší maximální frekvenci.
    - **5 stupňů**: Zlatý střed (výkon/plocha).
    - **8 stupňů**: Pro maximální frekvenci (vhodné pro velmi rychlé aplikace), ale zabere více místa.
- **Endianita**: Bi-endian (lze nastavit Big nebo Little endian).

## Možnosti konfigurace

Velkou výhodou Soft-core je, že spotřebuje jen tolik FPGA zdrojů, kolik funkcí zapnete.

- **Aritmetické jednotky (ALU)**:
    - **Barrel Shifter: Standardně umí CPU posun jen o 1 bit. Zapnutí této jednotky umožní posun o libovolný počet bitů v jednom taktu (nutné pro šifrování/kryptografii)**
    - **HW Násobička a Dělička**: Pokud se nezapnou, tyto operace se počítají softwarově a velmi pomalu (iteracemi).
        - SW výpočet dělení ≈ 32×(shift, odečtení, porovnání, skok, uložení bitu, shift, skok)
        - HW výpočet ≈ 32×iterace v HW
    - **Pattern Comparator**: HW vyhledávání shody bajtů ve slově (akcelerace textových operací).
- **FPU (Floating Point Unit)**: Hardwarová podpora pro desetinná čísla (IEEE 754 float).
    - Basic: Základní operace (+, -, *, /).
    - Extended: Pokročilé funkce (odmocnina, konverze int/float).
    - Softwarová emulace (součástí knihoven)
        - Násobení= Xor znamének, součet exponentů, násobení mantis, zaokrouhlení, normalizace
        - Sčítání = normalizace menšího čísla, součet, zaokrouhlení, normalizace výpočtu
- **MMU (Memory Management Unit)**: Volitelný blok pro správu virtuální paměti a ochranu paměťových stránek. Nutnost, pokud chcete na Microblaze spustit Linux.
    - Volitelné a nastavitelné cache
- **Stream Links (FSL/AXI Stream)**: Rozhraní pro přímé a rychlé napojení vlastních HW akcelerátorů přímo do procesoru (obchází se tím pomalá sběrnice).
- **Stream Links (FSL/AXI Stream)**: Rozhraní pro přímé a rychlé napojení vlastních HW akcelerátorů přímo do procesoru (obchází se tím pomalá sběrnice).

## Možnosti cache

Microblaze používá harvardskou architekturu (oddělené cesty pro data a instrukce) a dva typy sběrnic lokální paměť a externí paměť.

- Typy sběrnic:
    - **LMB (Local Memory Bus)**: Proprietární sběrnice s nízkou latencí. Slouží pro připojení lokální paměti (BRAM) uvnitř FPGA (pro bootloader, kritický kód).
    - **AXI**: Standardní sběrnice pro připojení externích pamětí (DDR) a periferií.
- **Cache (L1)**: Lze zapnout Instrukční (I-Cache) a Datovou (D-Cache).
    - **Write-Through**: Data se zapisují do cache i do paměti hned (bezpečnější).
    - **Write-Back**: Zapisuje se jen do cache (rychlejší), do paměti až při vyhození z cache.
- **Speciální funkce cache**:
    - **Victim Cache**: Malá vyrovnávací paměť pro "vyhozená" data z hlavní cache, zvyšuje výkon.
    - **Speculative loading**: Načítá data z paměti dříve, než si o ně program řekne (odhaduje potřebu).

## Microblaze V

- RISC-V RV32I ISA
    - 32 registrů, rozšiřitelný 32bit PC
    - In-Order, single-issue
    - Barel Shifter, CSR instrukce, fence
- Volitelně:
    - Rozšíření M(ultiplier/divider), A(tomic instructions), C(ompressed instructions), F(loating point)
    - Memory protection / Memory Management, Cache
    - Pipeline 3,4,5,8 (Mem)
    - Volitelné datové a instrukční sběrnice
    - Podpora 4 uživatelských instrukcí (stream interface)
    - Lockstep mód

### Microblaze V vs 11

- O 20% nižší max. frekvence
- O ~100% vyšší spotřeba zdrojů
- Vyladěnější návrh - 11
- Linux support remíza
- Do budoucna RISC-V 