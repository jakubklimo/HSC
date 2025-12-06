# AMBA AXI: AXI MM (lite) – parametry, kanály, transakce čtení a zápisu, rozdíl mezi AXI4 a AXI4-lite

## AXI4-LITE

Jedná se o zjednodušenou verzi AMBA AXI4  
Jednoduchá nízkokapacitní komunikace, typicky pro nastavování konfiguračních a stavových registrů

### Parametry
- Přenos pouze po slovech
- Omezená šířka (32,64 bitů)
- Adresní šířka do 64 bit
- Typicky Round Robin arbitrace
- Pomalá

### Kanály
- AXI-4 LITE je složen ze 4 signálů
- Čtení:
    - Adresní kanál – MASTER posílá adresu odkud chce číst
    - Datový kanál – SLAVE posílá přečtená data a status
- Zápis:
    - Adresní kanál – MASTER posílá adresu, kam chce zapisovat 
    - Datový kanál – MASTER posílá samotná data
    - Kanál potvrzení – SLAVE odpovídá, zda zápis proběhl v pořádku

### Transakce čtení a zápisu
- Čtení:
    - Master umístí adresu na kanál Čtení Adres a zároveň aktivuje ARVALID, čímž indikuje, že adresa je platná, a RREADY, čímž indikuje, že je připraven přijímat data od zařízení Slave.
    - Slave aktivuje ARREADY, čímž dává najevo, že je připraven přijmout adresu ze sběrnice.
    - Jelikož jsou aktivní signály ARVALID i ARREADY, při následující náběžné hraně hodinového signálu dojde k navázání spojení (handshake). Poté Master a Slave deaktivují signály ARVALID a ARREADY. (V tomto okamžiku Slave přijal požadovanou adresu).
    - Slave umístí požadovaná data na kanál Čtení Dat a aktivuje RVALID, čímž indikuje, že data v kanálu jsou platná. 
    - Jelikož jsou aktivní signály RREADY i RVALID, následující náběžná hrana hodinového signálu dokončí transakci. Signály RREADY a RVALID nyní mohou být deaktivovány.
- Zápis:
    - Master umístí adresu na kanál Zápis Adresy a data na kanál Zápis Dat. Současně aktivuje signály AWVALID a WVALID, čímž indikuje, že adresa a data na příslušných kanálech jsou platná. Master také aktivuje BREADY, čímž signalizuje, že je připraven přijmout odpověď.
    - Slave aktivuje AWREADY a WREADY na kanálech Zápis Adres a Zápis Dat.
    - Jelikož jsou na kanálech Zápis Adres i Zápis Dat přítomny signály Valid i Ready, dojde na těchto kanálech k navázání spojení (handshake) a příslušné signály Valid a Ready mohou být deaktivovány. (Poté, co proběhnou oba handshake, má Slave k dispozici adresu pro zápis i data).
    - Slave aktivuje BVALID, čímž indikuje, že na kanálu Zápis potvrzení je platná odpověď.
    - Následující náběžná hrana hodinového signálu dokončí transakci, přičemž signály Ready i Valid na kanálu Zápis potvrzení jsou v logické jedničce (high).

## AXI4-MM
Tohle je braný jako klasickej AXI4 nebo AXI4 FULL

### Parametry
- Šířka až 1024 bitů 
- Burst přenosy do 256 slov
    - FIXED , INCR, WRAP 
- Adresní šířka do 64 slov
- rychlá

### Kanály
- Čteni:
    - Adresní kanál
    - Datový kanál
- Zápis:
    - Adresní kanál
    - Datový kanál
    - Kanál potvrzení

### Transakce čtení a zápisu
- Čtení:
    - Nejprve kanál Čtení Adres pošle z Masteru do Slavu, aby nastavil adresu a nějaké řídící signály
    - Potom data pro tuto adresu jsou přeneseny ze Slavu do Masteru přes kanál Čtení Dat
- Zápis:
    - Nejprve je kanál Zápisu Adres z Masteru do Slavu, aby nastavil adresu a nějaké řídící signály
    - Potom data pro tuto adresu jsou poslány z Masteru do Slavu přes kanál Zápis Dat
    - Nakonec je poslána odpověď ze Slavu do Masteru přes kanál Zápis potvrzení pro indikaci, zda byl přenos dat úspěšný

## Rozdíl mezi AXI4 MM a AXI4-lite
- AXI-4 MM neboli AXI-4 full je určeno pro vysoce výkonný přenos dat, zatímco AXI4-Lite je navržen pro jednoduché ovládání a nastavení stavu
- AXI4-MM má burst: Master pošle pouze jednu adresu a už se navážení spojení se Slavem který automaticky inkrementuje adresu pro další data, zatímco AXI4-LITE podporuje jen SINGLE TRANSACTION, tudíž každá část dat potřebuje svůj vlastní adresní handshake
- Při spojení s více mastery AXI4-Lite funguje na pricnipu Round Robin, tudíž se postupuje postupně od A do Z, zatímco u AXI4-MM se může zvolit priorita.

Je to z týhle stránky: https://www.logic-fruit.com/blogs/digital-interfaces/axi-full-axi-lite-interfaces/#:~:text=able%20to%20access%20memory%2Dmapped,than%20the%20complete%20AXI4%20interface.