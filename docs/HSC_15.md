# DMA řadič, scatter gather, ovladače, koherence dat, přehled dostupných IP

## DMA (Direct Memory Access) 
DMA řadič je specializovaný hardware, který umožňuje periferiím (např. síťovým kartám, disků, A/D převodníkům) přistupovat přímo do operační paměti bez zatěžování procesoru. Princip je takový, že procesor nastaví parametry přenosu (zdrojová adresa, cílová adresa, délka), DMA řadič převezme řízení sběrnice a provede přenos dat mezi pamětí a periferií a po dokončení přenosu informuje procesor (obvykle přerušením). Výhodou je, že odlehčuje procesor od rutinních přesunů dat, zvyšuje propustnost systému a umožňuje paralelní běh výpočtů a přenosů.

![Obrázek](pics/hsc15_1.png)

## Scatter-Gather
Scatter-Gather je mechanismus DMA řadiče, který umožňuje efektivní přenos dat uložených v nesouvislých blocích fyzické paměti (fragmentace), aniž by procesor musel inicializovat a řídit každou dílčí transakci zvlášť. Velký soubor dat může být totiž fyzicky rozházen na různé adresy a cílem je tyto jednotlivé fragmenty souboru najít a spojit. Technicky je tento proces realizován pomocí hardwarově zpracovávaného spojového seznamu řídicích struktur zvaných deskriptory (Buffer Descriptors), které procesor předem připraví v operační paměti. Každý deskriptor obsahuje ukazatele na fyzickou adresu datového bufferu, informaci o délce přenosu a odkaz na adresu následujícího deskriptoru. DMA řadič po aktivaci automaticky načte první deskriptor, provede definovaný přenos dat, následně si bez asistence procesoru načte konfiguraci pro další blok z odkazu následujícího ukazatele a takto cyklicky pokračuje až do konce řetězce s možností aktivace cyklického opakování, kdy poslední deskriptor ukazuje na ten první. Dokáže tedy autonomně sesbírat data z různých adres složit je do jednoho seskupení nebo je naopak rozptýlit do volných paměťových bloků, což je klíčové pro vysoký výkon u segmentovaných dat a v systémech se stránkováním paměti, například u síťových paketů nebo multimediálních dat, kde jsou bloky paměti často nesouvislé. Scatter-Gather tedy zvyšuje efektivitu a flexibilitu přenosů.

**Podrobná struktura deskriptoru:**

![Obrázek](pics/hsc15_2.png)

## Ovladače
Ovladače jsou softwarové komponenty, které propojují operační systém s DMA hardwarem. Jejich úkolem je inicializovat řadič, nastavit deskriptory, obsluhovat přerušení a nabídnout aplikacím jednoduché rozhraní pro spuštění přenosu. To znamená, že aplikace zavolá funkci typu „pošli tento buffer do periferie“, ovladač připraví deskriptory a DMA řadič provede samotný přenos. Ovladače tedy zajišťují, aby hardware byl využíván správně a bezpečně.

## Koherence dat
Koherence dat řeší problém, že procesor pracuje s cache pamětí (rychlou vyrovnávací pamětí), zatímco DMA zapisuje přímo do hlavní paměti (DDR RAM). Pokud procesor má v cache starší kopii dat, může dojít k nesouladu. Aby byla data konzistentní a načetla se tedy aktuální data z paměti, je nutné před spuštěním DMA přenosu cache vyprázdnit (flush) a po dokončení přenosu ji prohlásit za neplatnou, tedy takovou, které se už nedá věřit, protože může obsahovat zastaralá nebo nekonzistentní data. Procesor je pak při příštím čtení znovu načte z hlavní paměti, kde už jsou aktuální hodnoty. Některé moderní architektury mají hardwarovou podporu koherence, takže se o to stará systém automaticky. Správná koherence je zásadní pro spolehlivý provoz.

## Přehled dostupných IP
AXI Interconnect: Soubor IP jader infrastruktury sloužících k vytvoření sběrnic v SoC (System on Chip):
- **AXI Crossbar** je režim IP jádra AXI Interconnect, který umožňuje neblokující propojení mezi více AXI master a více AXI slave rozhraními. Každá transakce je směrována podle adresy na odpovídající slave a ke stejnému slave může v daný okamžik mluvit jen jeden master. Současně však mohou probíhat transakce k různým slave bez vzájemného blokování. Jedná se tedy o inteligentní přepínač (switch) neboli dopravní uzel pro sběrnici AXI s optimalizací zaměřenou na využitou plochu čipu nebo na jeho rychlost. 
- **AXI Data Width Converter** slouží jako překladač (adaptér) mezi dvěma komponentami, které mají různou šířku datové sběrnice. Zajišťuje, aby se spolu mohly bavit například procesor (32-bit) a paměť (64-bit), aniž byste museli psát složitou logiku pro spojování nebo dělení dat.
- **AXI Clock Converter** slouží k překlenutí rozdílných hodinových domén mezi dvěma AXI rozhraními. Používá se tehdy, když master a slave běží na odlišných frekvencích nebo mají asynchronní hodinové signály. Converter zajišťuje, že se data přenesou bezpečně, bez hazardů a porušení protokolu AXI.
- **AXI Data FIFO** je vyrovnávací paměť (buffer) navržená speciálně pro protokol AXI4. Její hlavní funkcí je dočasně ukládat data a řídicí signály, které proudí mezi komponentou Master a komponentou Slave, když jedna ze stran není připravena data okamžitě přijmout.
- **AXI Register Slice** slouží ke vložení registrové (pipeline) vrstvy do toku signálů AXI, čímž zkrátí kombinační cesty mezi vstupy a výstupy a usnadní časování u vysokých frekvencí nebo dlouhých fyzických tras na čipu.
- **AXI MMU (Memory Management Unit)** provádí mapování a překlad adres v rámci paměťově mapovaných AXI systémů. Umožňuje přesměrovat požadavky z AXI masterů do různých adresních cílů, slučovat nebo rozdělovat adresní prostor a tím flexibilněji organizovat systémovou paměťovou topologii bez zásahu do samotných master/slave IP bloků

**AXI IPIC (IP Interface Controller)** je rozhraní, které propojuje uživatelské IP jádro s AXI sběrnicí. Slouží jako standardizovaný most mezi vlastním logickým blokem a AXI interconnectem, aby se IP jádro mohlo snadno začlenit do systému založeného na AMBA AXI protokolu.

**AXI Peripheral** je uživatelsky definovaný nebo předpřipravený blok logiky, který komunikuje s ostatními částmi systému přes standardizovanou sběrnici AXI. Slouží jako rozhraní mezi procesorem a periferní logikou.

**AXI DMA** umožňuje vysokorychlostní přenos dat mezi pamětí (AXI4 Memory Mapped) a periferiemi pracujícími se streamovanými daty (AXI4-Stream), bez nutnosti zatěžovat procesor.
- MM2S (Memory-Mapped to Stream): čte blok dat z paměti a posílá je do streamového rozhraní.
- S2MM (Stream to Memory-Mapped): přijímá streamovaná data a ukládá je do paměti.

**AXI DataMover** zajišťuje efektivní přenos dat (délka přenosu je 23 bitů) mezi paměťově mapovaným rozhraním AXI4 a proudovým rozhraním AXI4-Stream. Slouží jako univerzální transportní blok, který dokáže číst data z paměti a posílat je do streamu, nebo naopak přijímat streamovaná data a zapisovat je do paměti.

**PS DMA controller (Processing System Direct Memory Access)** je specializovaný controller s 8 virtuálními kanály, který je určený výhradně k tomu, aby přenášel data z jednoho místa na druhé, aniž by tím zatěžoval hlavní procesor.