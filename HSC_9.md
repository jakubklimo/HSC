# Sériové sběrnice – přehled a principy (SPI, IIC, UART)

Sériové sběrnice přenášejí data bit po bitu za sebou. Používají se pro komunikaci mezi mikrokontroléry a periferiemi (senzory, displeje, paměti). Jsou to: Uart(Rs232, Rs485,Rs455), i2c, spi, usb, can, ethernet.  
Obecně se u nich řeší tyto vlastnosti:

- **Synchronizace**
  - **Synchronní**: Data se čtou přesně s tikem hodin.
  - **Asynchronní**: Nemá hodiny. Obě strany musí mít předem nastavenou **stejnou rychlost**.
- **Směr toku dat**
  - **Symplex**: Data tečou trvale jen jedním směrem.
  - **Half-Duplex**: Data tečou oběma směry, ale **střídavě**.
  - **Full-Duplex**: Data tečou oběma směry **současně**.
- **Rychlost přenosu**
  - Modulační rychlost – baud (Bd, bps)
  - Přenosová rychlost – bitů za sekundu (b/s, bps) pouze bity, které nesou informace

1. **UART (Universal Asynchronous Receiver-Transmitter)**

   - **Typ**: Asynchronní (nemá hodinový signál), vyskytuje se i USART (synchronní).
   - **Topologie**: Point-to-point (spojení dvou zařízení).
   - **Vodiče**: 2 datové vodiče (+ zem):
     - **TX** (Transmit) – vysílání.
     - **RX** (Receive) – příjem.
     - Poznámka: Zapojuje se křížem (TX jednoho na RX druhého).
   - **Princip**:
     - Jelikož chybí hodiny, obě zařízení musí mít předem nastavenou stejnou rychlost (**Baud rate**, např. 9600 Bd).
     - Komunikace začíná **Start bitem** (log. 0), následují data (obvykle 8 bitů), volitelně paritní bit (kontrola chyby) a končí se **Stop bitem** (log. 1).

2. **SPI (Serial Peripheral Interface)**

   - **Typ**: Synchronní (má hodinový signál).
   - **Topologie**: Master-Slave (jeden řídicí, více podřízených).
   - **Vodiče**: 4 vodiče (čtyřdrátová sběrnice):
     - **SCLK** (Serial Clock) – hodinový signál, generuje Master.
     - **MOSI** (Master Out Slave In) – data od Mastera k Slave.
     - **MISO** (Master In Slave Out) – data od Slave k Masterovi.
     - **SS/CS** (Slave Select / Chip Select) – výběr konkrétního zařízení.
    - **Princip**:
      - Master vybere Slave stažením jeho linky CS do nuly (aktivní v nule).
      - S každým tikem hodin se přesune jeden bit tam a jeden zpět (funguje jako posuvný registr).
      - Na sběrnici je vždy jeden Master, komunikační rychlost určuje hodinový signál – až desítky MHz, všechny signály jsou pouze jednosměrné
    - **Vlastnosti**: Velmi rychlá komunikace, Full-duplex. Nevýhodou je spotřeba pinů (každý Slave potřebuje vlastní CS drát). Určená k propojování periferií a k připojování periferií k procesoru, vhodná pouze pro velmi krátké spoje, používá se pro připojení EEPROM, RTC

3. I2C (Inter-Integrated Circuit, psáno také I²C nebo IIC)

    - **Typ**: Synchronní.
    - **Topologie**: Multi-master (sběrnicová topologie).
    - **Vodiče**: 2 vodiče:
        - **SDA** (Serial Data) – obousměrná data.
        - **SCL** (Serial Clock) – hodiny.
        - Důležité: Vyžaduje **pull-up rezistory** (vodiče jsou v klidu na log. 1).
    - **Princip**:
        - Komunikace je řízena adresou. Master pošle **Start condition**, pak **adresu zařízení** (7 bitů) a bit R/W (čtení/zápis).
        - Zařízení s touto adresou odpoví bitem **ACK** (potvrzení).
        - Data se posílají po 8 bitech, za každým bytem následuje ACK/NACK.
    - **Vlastnosti**: Half-duplex (data tečou jen jedním směrem v daný čas). Šetří piny (jen 2 dráty pro až 127 zařízení), ale je pomalejší a složitější než SPI.