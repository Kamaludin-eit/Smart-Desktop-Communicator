### Smart Desktop Communicator – Design-Dokumentation

**Rev 1.0 | Kamaludin Ali**

\---



#### Übersicht

ESP32-S3-basiertes stationäres IP-Telefon mit PoE-Versorgung, Wired Ethernet,
I2S-Audio und OLED-Display. 4-lagige PCB (150 × 80 mm).

\---



#### Wichtigste Design-Entscheidungen

##### 1\. Mikrocontroller – ESP32-S3-WROOM-1

* Dual-Core LX7, 16 MB Flash, 8 MB PSRAM, 2.4 GHz WLAN integriert
* Natives USB Serial/JTAG auf GPIO19/20 – kein separater USB-UART-Chip nötig
* I2S-Peripheral für DAC (MAX98357A) und PDM-Mikrofon (IM69D120) separat nutzbar
* SPI-Peripheral für W5500 Ethernet-Controller



##### 2\. PoE-Versorgung – TPS2378DDA + PDQE30-Q48-S5-D

* TPS2378DDA: IEEE 802.3at Type-2 PD-Controller (bis 30 W)
* Kein integrierter DC/DC – externer isolierter Wandler PDQE30 (48 V → 5 V, 30 W)
* Isolation trennt PWR\_GND (48 V-Seite) vollständig von GNDD/GNDA (Systemseite)
* Alternativversorgung via USB-C (5 V) mit Schottky-Dioden-OR-Schaltung#



##### 3\. RJ45-Connector – ARJP11A-MASA-B-A-EMU2 (Abracon)

* Integrierte Magnetics, Center-Tap, eingebauter Brückengleichrichter
* PRW+ (Pin 9) / PRW− (Pin 10): rektifizierte PoE-Spannung direkt abgreifbar
* Kein externer Brückengleichrichter oder separates Magnetics-Modul nötig
* Integrierte Status-LEDs (Link/Activity)



##### 4\. Ethernet – W5500

* Hardware-TCP/IP-Stack – entlastet ESP32-S3 vollständig
* SPI-Interface (bis 80 MHz), geringer Softwareaufwand



##### 5\. Audio-Ausgabe – MAX98357A

* I2S DAC + Class-D 3 W Verstärker in einem IC
* BTL-Ausgang: keine DC-Koppelkondensatoren am Lautsprecher nötig
* PVDD über Ferritperle L3 (600 Ω @ 100 MHz, 1 A) von 5V-Rail entkoppelt
* 3.3V-Logiksignale vom ESP32 kompatibel (VIH\_min = 1.5 V)
* Lautsprecher extern im Gehäuse montiert (akustische Echo-Minimierung)



##### 6\. Audio-Eingang – IM69D120 (Infineon)

* MEMS-Mikrofon mit PDM-Ausgang, SNR 69 dB, AOP 120 dB SPL
* Nativ vom ESP32-S3 PDM-Peripheral unterstützt
* VDD über eigene Ferritperle L4 (600 Ω @ 100 MHz, 50 mA) von VCCA\_MIC
* Software-AEC (Acoustic Echo Cancellation) via ESP-ADF



##### 7\. Massekonzept – 2 getrennte GND-Domänen

|Domäne|Verbindung|Komponenten|
|-|-|-|
|GNDD|Solder Jumper Pin 2 |ESP32, W5500 digital, Tasten, LEDs|
|GNDA|Solder Jumper Pin 1|W5500 PHY, IM69D120, MAX98357A, Quarz|



##### 8\. 4-Lagen PCB-Aufbau (150 × 80 mm)

|Lage|Inhalt|
|-|-|
|Layer 1 (Top)|Bauteile + Signale + GNDA-Kupferfläche|
|Layer 2 (In1)|+3,3V  |
|Layer 3 (In2)|GNDD-Kupferfläche|
|Layer 4 (Bottom)|Signale + GNDA-Kupferfläche|

* Digitale GND-Pads auf Top-Layer über Vias mit GNDD (In1) verbunden
* Analoge Komponenten direkt auf GNDA-Fläche (Top-Layer)



##### 9\. Versorgungsschienen

|Schiene|Quelle|Verbraucher|
|-|-|-|
|+48V\_POE|PRW+ via SMAJ58A|TPS2378DDA VDD|
|+5V|PDQE30 Ausgang|MAX98357A, AMS1117|
|+3.3V (VCCD)|AMS1117-3.3|ESP32, W5500 digital, Tasten, LEDs|
|VCCA\_ETH|VCCD via FB\_ETH (600Ω)|W5500 VCCA|
|VCCA\_MIC|VCCD via FB\_MIC (600Ω)|IM69D120 VDD|

### 

##### 10\. ESD-Schutz

|Interface|Schutz|
|-|-|
|PoE 48V|SMAJ58A (unidirektional, 58V)|
|USB-C VBUS|SMAJ5.0A (unidirektional, 5V)|
|USB-C D+/D−|USBLC6-2P6 (Low-C, bidirektional)|
|Ethernet TX/RX|Magnetics-Isolation im ARJP11A|

### 

##### 11\. USB-C – Amphenol 12401610E4#2A

* CC1/CC2: je 5.1 kΩ nach GNDD (5V UFP-Erkennung, kein PD-Controller nötig)
* Schottky-Diode OR-ing mit PoE-Versorgung
* D+/D−: direkt an ESP32 GPIO19/GPIO20 (natives USB JTAG/Serial)



##### 12\. HMI

* SSD1306 OLED 0.96" (128×64, I2C): Anrufstatus, IP, Uhrzeit
* 5 Taster: Annehmen/Auflegen, Vol+, Vol−, Stummschalten, Notruf
* 1 Status-LEDs:  Power(rot)
* Hardware-Entprellung: RC-Glied (100 kΩ + 1 µF) pro Taster

\---



#### Geplante Firmware-Architektur

* **ESP-IDF** v5.x + **ESP-ADF** (Audio Development Framework)
* SIP-Stack: sikorapatryk/sip-call oder OLIMEX/sip\_phone\_example als Referenz
* Codec: G.711 µ-law (PCMU, Payload Type 0)
* AEC: ESP-ADF eingebaut
* W5500-Treiber: arduino-ethernet oder esp-idf-w5500

\---



#### Wählscheibe

Nicht vorgesehen – Anrufsteuerung erfolgt über SIP-Server/PBX-Integration (z.B. FreePBX, Asterisk).

\---



## Revision History

|Rev|Datum|Änderung|
|-|-|-|
|1.0|2026-05|Erstversion|



