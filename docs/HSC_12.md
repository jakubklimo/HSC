#  AMBA AXI: AXI stream – princip rozhraní, transakce, backpressure, přenos z více zdrojů

Advanced Extensible Interface  
Rychlé i pomalé periferie

## Princip rozhraní
- jedná se o PTP Master – Slave komunikační protokol pro přenos dat mezi komponentami v rámci hardwarových systémů
- nepoužívá bursty, ale pracuje s datovými pakety, které mohou být nekonečné a data mohou plynout souvisle
- Použití „handshake“ pro zajištění integrity dat a zabránění ztrátě dat v přípradě nesouladu rychlostí:
    - TREADY – Slave je připraven přijímat data nová data
    - TVALID – Indikuje, že Master má ověřená data připravená k odeslání

## Transakce
- Data se přenesou pouze pokud je TVALID a TREADY v 1 při náběžné hrané hodin
- To znamená, že Master nebo Slave mohou kdykoliv zastavit přenos dat, pokud že jsou zaneprázdněný
- Mezi vstupy a výstupy uvnitř Master nebo Slave nesmí být kombinační logika

## Backpressure
- Jedná se o mechanismus řízení toku, ke kterému dochází, když MASTER posílá data (TVALID=1) rychleji, než je může zpracovat SLAVE (TREADY = 0) 
- Backpressure ze SLAVE strany signalizuje MASTERU, aby zpomalil nebo se zastavil, díky tomu dokáže zabránit ztrátě dat a SLAVE udrží krok 
- Bez backpressure mechanismu by mohl být SLAVE zahlcen a data by mohla být ztracena
- Jak to funguje:
    - Backpressure je aktivovanej TREADY ze SLAVE
    - Jednoduše MASTER pošle TVALID a čeká se, než bude SLAVE TREADY

## Přenos z více zdrojů – Interconnection/CrossBar
- SLAVE přijímá signál z více MASTERŮ
- Mezi MASTERY a SLAVE se vloží logický blok ARBITER
- ARBITER pak rozhoduje, který MASTER má právo posílat data
- Jak to funguje:
    - ARBITER sleduje z MASTERŮ TVALID 
    - Na základě strategie vybere jednoho mastera a propojí jeho signál se SLAVEM
    - Ostatní MASTERY čekají tím, že se jim drží TREADY na 0
- Důležité pravidlo:
    - Mezi přepínání MASTERŮ se musí respektovat signál TLAST
    - TLAST = konec paketu
- Strategie vybírání MASTERU:
    - ROUND ROBIN:
        - Spravedlivé postupné střídání (od A ->B … ->Z)
    - FIXED PRIORITY:
        - Prioritní systém 
        - Nějakým MASTERŮM se nastaví přednost
- Identifikace zdrojů pomocí TID:
    - Jedná se o volitelný signál, kterému ARBITER přiřadí ID značku
    - SLAVE si ji pak přečte a zjistí kam má data uložit
- Sloučení dat:
    - Vytvoření bloku pro synchronizaci streamů
    - Signál TREADY jde k MASTERŮM pouze pokud mají platná data TVALID