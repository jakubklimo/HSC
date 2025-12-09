# AMBA AXI: parametry, základní topologie, IP jádra, prostředky připojení periferií

## Parametry
- Šířka až 1024 bitů 
- Burst přenosy do 256 slov
- FIXED , INCR, WRAP 
- Adresní šířka do 64 slov
- rychlá

## Základní topologie
- Tady trochu stručně už je to probraný ve 12 a 13
- Typ spojení PTP: MASTER-SLAVE
- 5 kanálů: Čtení (Adresní a datový kanál), Zápis (Adresní, datový a potvrzovací kanál)
- AXI Interconnect neboli Crossbar (propojení více Masterů s více Slaves)
- Handshake mechanismus (Ready a Valid)
- Oddělené cesty pro čtení a zápis

## IP jádra

Druhy AXI4 IP jáder:

- AXI4-DMA: vyžaduje, aby CPU pozastavilo, zapsalo do konfiguračních registrů (zdrojová adresa, délka, řízení) pomocí AXI-Lite a poté spustilo
    - Konfigurovatelný MM2S a S2MM převodník
        - 2xDatamover – kanály čtení a zápisu
            - 32-1024 MM, 32-1024S, Burst 2-256, DRE, 8-26 bitů délka přenosu
        - Volitelný SG
        - Volitelný řídící a status stream
        - Řízení z CPU pomocí uživatelských registrů
- AXI4-Lite IPIC: Provides a simple control interface (Point-to-Point) for user IP to connect to an AXI interconnect for configuration.
    - IPIC = IP Interface Controller
    - Dekodér adres, Registrový soubor, Zápis s byte enable, Timeout čítač, Podpora burst
- AXI-DataMover: I když plní podobnou funkci jako řadič s přímým přístupem do paměti (DMA), funguje odlišně: je řízen hardwarem, nikoli softwarově konfigurován
    - funguje jako most mezi AXI4-Stream a AXI4 Memory-Mapped
    - Zadáte mu příkaz (přes rozhraní AXI-Stream) obsahující adresu a počet bajtů a on okamžitě provede přenos.
    - Konfigurovatelný MM2S a S2MM převodník
    - 2xDatamover – kanály čtení a zápisu
        - 32-1024 MM, 32-1024S, Burst 2-256
        - DRE
        - 23 bitů délka přenosu
    - Volitelný SG
    - Volitelný řídící a status stream
    - Řízení z IP jádra pomocí řídícího Stream rozhraní
        - Command, Status
    - Nemá ovladače

## Prostředky připojení periferií

- PTP
    - Přímí připojení periferií
- AXI Interconnection
    - CPU má limitovaný počet Master portů
    - Použití prostředníka: Arbiteru
- DMA (Direct Memory Access)
    - AXI4-Stream periferie nemůžeš připojit přímo k CPU (jelikož rozumí jen adresám což stream nemá) je potřeba použít DMA Engine pro jejich přemostění
- Memory Mapping
    - Pro AXI4-Lite a AXI4-Full
    - Každá periferie dostane unikátní adresu
