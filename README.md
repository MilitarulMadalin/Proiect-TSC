# InkTime

## Cuprins

1. [Schemă bloc](#1-schemă-bloc)
2. [Bill of Materials (BOM)](#2-bill-of-materials-bom)
3. [Descrierea hardware detaliată](#3-descrierea-hardware-detaliată)
4. [Mapare pini nRF52840](#4-mapare-pini-nrf52840)
5. [Design log](#5-decizii-de-design)

---

## 1. Schemă bloc

```
                  +--------------+      +-------------+
                  |   E-Paper    |      |    LiPo     |
                  |  Drive Circ. |      |   Charger   +
                  +------+-------+      +------+------+
                         |                     |
                         | EPD_PINS            | I2C, PMIC_INT       +-----------+
                         v                     |                     |   Fuel    |
  +-----------+   +--------------+             |            I2C      |   Gauge   |
  |  E-Paper  |<--+   E-Paper    |             |        +----------->|           |
  |   1.5"    |   |    Conn.     |             |        |            +-----------+
  +-----------+   +------+-------+             |        |
                         |                     |        |            +-----------+
                         |  SPI                |        |            |   USB-C   |
                         v                     v        |            |    J4     |
                  +----------------------------+--------+------+     +-----+-----+
                  |                                            |           |
                  |              nRF52840   (U1)               |<----------+
                  |                                            |  D+, D-
                  |                                            |<--------+
                  +--+--------+--------+--------+------+-------+         |
                     |        |        |        |      |                 |
                     | I2C    | I2C    | ANT1   | SWD  | 3xGPIO          | D+, D-
                     |        | HAPT.  |        |      |                 |
                     v        | EN     v        v      v           +-----+-----+
                  +-------+   |   +--------+ +-----+ +---------+   |    ESD    |
                  |  IMU  |   |   | Antena | | SWD | | Butoane |   | Protection|
                  |BMA423 |   |   |  2.4   | | J2  | +---------+   +-----------+
                  +-------+   |   +--------+ +-----+
                              v
                          +-------+
                          |Haptic |
                          |DRV2605|
                          +-------+
```

---

## 2. Bill of Materials (BOM)

| Desig. (ref.)                                                                  | Valoare / Part Number          | Funcție în circuit                           | Pachet        | Cant. | JLCPCB                                           | Datasheet                                                                                         |
|--------------------------------------------------------------------------------|--------------------------------|----------------------------------------------|---------------|-------|--------------------------------------------------|---------------------------------------------------------------------------------------------------|
| U1                                                                             | nRF52840-QIAA                  | MCU BLE 5 / 2.4 GHz, Cortex-M4F, 1 MB Flash  | aQFN73        | 1     | [C190794](https://jlcpcb.com/partdetail/C190794) | [PDF](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.7.pdf)                                 |
| IC1                                                                            | BQ25180YBGR                    | Încărcător LiPo 1-cell, programabil I2C      | DSBGA-8 0.4mm | 1     | [C965556](https://jlcpcb.com/partdetail/C965556) | [PDF](https://www.ti.com/lit/ds/symlink/bq25180.pdf)                                              |
| U3                                                                             | MAX17048G+T10                  | Fuel gauge ModelGauge 1-cell, I2C            | TDFN-8        | 1     | [C129977](https://jlcpcb.com/partdetail/C129977) | [PDF](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17048-MAX17049.pdf) |
| IC9                                                                            | RT6160AWSC                     | Buck-Boost 3V3, 2A (rail principal)          | WL-CSP-15     | 1     | [C58559](https://jlcpcb.com/partdetail/C58559)   | [PDF](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-02.pdf)                         |
| IC3                                                                            | BMA423                         | Accelerometru 3-axis, step counter, wrist-tilt | LGA-12      | 1     | [C553690](https://jlcpcb.com/partdetail/C553690) | [PDF](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds000.pdf) |
| IC2                                                                            | DRV2605YZFR                    | Driver haptic ERM/LRA, 123 efecte, I2C       | DSBGA-9       | 1     | [C92482](https://jlcpcb.com/partdetail/C92482)   | [PDF](https://www.ti.com/lit/ds/symlink/drv2605.pdf)                                              |
| Q1                                                                             | DMG2305UX-7 (20V/4.2A)         | P-MOSFET load-switch rail EPD                | SOT-23        | 1     | [C114005](https://jlcpcb.com/partdetail/C114005) | [PDF](https://www.diodes.com/assets/Datasheets/DMG2305UX.pdf)                                     |
| Q3                                                                             | SI1308EDL-T1-GE3               | N-MOSFET pentru boost charge-pump EPD        | SOT-23        | 1     | [C294574](https://jlcpcb.com/partdetail/C294574) | [PDF](https://www.vishay.com/docs/66632/si1308edl.pdf)                                            |
| D3                                                                             | USBLC6-2SC6Y                   | TVS diode array, protecție ESD pe USB D±     | SOT-23-6      | 1     | [C7519](https://jlcpcb.com/partdetail/C7519)     | [PDF](https://www.st.com/resource/en/datasheet/usblc6-2.pdf)                                      |
| D2, D4, D5                                                                     | MBR0530                        | Diodă Schottky 30V pentru charge-pump EPD    | SOD-123       | 3     | [C131850](https://jlcpcb.com/partdetail/C131850) | [PDF](https://www.onsemi.com/pdf/datasheet/mbr0520-d.pdf)                                         |
| ANT1                                                                           | 2450AT18B100E                  | Antenă ceramică chip 2.4 GHz, Johanson       | 3216          | 1     | [C76907](https://jlcpcb.com/partdetail/C76907)   | [PDF](https://www.johansontechnology.com/datasheets/2450AT18B100E.pdf)                            |
| X1                                                                             | NX2016SA 32 MHz ±10 ppm        | Cristal HF pentru radio și CPU               | 2016          | 1     | [C255982](https://jlcpcb.com/partdetail/C255982) | [PDF](https://www.ndk.com/images/products/catalog/c_NX2016SA-STD-CRA-2_e.pdf)                     |
| X2                                                                             | ABS07-120 32.768 kHz           | Cristal LF pentru RTC (low-power)            | 3215          | 1     | [C32346](https://jlcpcb.com/partdetail/C32346)   | [PDF](https://abracon.com/Resonators/abs07-120.pdf)                                               |
| L1                                                                             | 3.9 nH                         | RF match filtru π — inductor serie antenă    | 0402          | 1     | [C1789](https://jlcpcb.com/partdetail/C1789)     | —                                                                                                 |
| L3                                                                             | 15 nH                          | RF match filtru π — inductor paralel          | 0402          | 1     | [C1794](https://jlcpcb.com/partdetail/C1794)     | —                                                                                                 |
| L2                                                                             | 10 µH                          | DCDC intern nRF52840 (pin DCC)               | 0402          | 1     | [C1046](https://jlcpcb.com/partdetail/C1046)     | —                                                                                                 |
| L5                                                                             | WE-TPC 744043680, 68 µH        | Boost e-paper (rail VGH / VGL)                | 4828          | 1     | [C492815](https://jlcpcb.com/partdetail/C492815) | [PDF](https://www.we-online.com/catalog/datasheet/744043680.pdf)                                  |
| L7                                                                             | FTC252012SR47MBCA, 0.47 µH     | Inductor putere pentru RT6160                 | 2520          | 1     | [C123046](https://jlcpcb.com/partdetail/C123046) | —                                                                                                 |
| J4                                                                             | KH-TYPE-C-16P                  | Conector USB-C receptacle (16 pini)           | SMT           | 1     | [C165948](https://jlcpcb.com/partdetail/C165948) | —                                                                                                 |
| J1                                                                             | Molex 503480-2400              | Conector baterie LiPo, 2-pin pitch 1.5 mm     | SMT           | 1     | [C518439](https://jlcpcb.com/partdetail/C518439) | [PDF](https://www.molex.com/pdm_docs/sd/5034802400_sd.pdf)                                        |
| J2                                                                             | TC2030-IDC (Tag-Connect)       | Footprint debug SWD (pogo-pins, fără receptacle) | —         | 1     | (doar footprint)                                 | [PDF](https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/TC2030-IDC.pdf)              |
| SW_UP, SW_DN, SW_ENT                                                           | EVP-AKE31A                     | Buton tactil side-push 4.5×3.5 mm             | SMT           | 3     | [C719888](https://jlcpcb.com/partdetail/C719888) | [PDF](https://industrial.panasonic.com/cdbs/www-data/pdf/ATV0000/ATV0000CE11.pdf)                 |
| C11, C12                                                                       | 100 nF / 10V (X7R)             | Decuplare locală IC-uri digitale             | 0201          | 2     | [C106250](https://jlcpcb.com/partdetail/C106250) | —                                                                                                 |
| C5, C7, C8                                                                     | 100 µF / 6.3V                  | Bulk buck-boost RT6160                       | 0201          | 3     | [C19702](https://jlcpcb.com/partdetail/C19702)   | —                                                                                                 |
| C19                                                                            | 100 µF / 6.3V                  | Bulk rail VBAT                               | 0201          | 1     | [C19702](https://jlcpcb.com/partdetail/C19702)   | —                                                                                                 |
| C6, C14, C20, C21                                                              | 4.7 µF / 10V (X7R)             | Decuplare nRF (DEC*)                         | 0201          | 4     | [C19666](https://jlcpcb.com/partdetail/C19666)   | —                                                                                                 |
| C15                                                                            | 1.0 µF / 10V                   | Decuplare VREG nRF                           | 0201          | 1     | [C15849](https://jlcpcb.com/partdetail/C15849)   | —                                                                                                 |
| C39                                                                            | 10 µF / 6.3V                   | Bulk IC1 (charger)                           | 0201          | 1     | [C15850](https://jlcpcb.com/partdetail/C15850)   | —                                                                                                 |
| C43                                                                            | 4.7 µF                         | Intrare IC2 (driver haptic)                  | 0201          | 1     | [C19666](https://jlcpcb.com/partdetail/C19666)   | —                                                                                                 |
| C1-EP-DR, C24                                                                  | 10 µF                          | Filtre intrare driver EPD / RT6160           | 0201          | 2     | [C15850](https://jlcpcb.com/partdetail/C15850)   | —                                                                                                 |
| C2-EP-DR                                                                       | 4.7 µF / 25V                   | Bulk intrare boost EPD                       | 0201          | 1     | [C19666](https://jlcpcb.com/partdetail/C19666)   | —                                                                                                 |
| C25, C33                                                                       | 22 µF                          | Ieșire RT6160 (3V3)                          | 0201          | 2     | [C45783](https://jlcpcb.com/partdetail/C45783)   | —                                                                                                 |
| C17, C18                                                                       | 12 µF                          | Filtre DCDC intern nRF                       | 0201          | 2     | [C162698](https://jlcpcb.com/partdetail/C162698) | —                                                                                                 |
| C16                                                                            | 47 nF                          | Filtru control-loop RT6160                   | 0201          | 1     | [C113010](https://jlcpcb.com/partdetail/C113010) | —                                                                                                 |
| C9                                                                             | 820 pF                         | Decuplare DEC6 nRF                           | 0201          | 1     | [C1603](https://jlcpcb.com/partdetail/C1603)     | —                                                                                                 |
| C1, C2                                                                         | 12 pF (NP0 / C0G)              | Load-caps pentru X1 (32 MHz)                 | 0201          | 2     | [C1570](https://jlcpcb.com/partdetail/C1570)     | —                                                                                                 |
| C3, C4                                                                         | 1 pF                           | RF match (π) — shunt                         | 0201          | 2     | [C1552](https://jlcpcb.com/partdetail/C1552)     | —                                                                                                 |
| C10, C13, C22                                                                  | N.C. (Not Connected)           | Rezervate — neplasate la fabricație          | 0201          | 3     | (nu se lipește)                                  | —                                                                                                 |
| C23, C27, C29, C30, C31, C32, C34, C37, C38, C42                               | GRM011R60J152KE01L, 1.5 nF     | Filtre RF / bypass high-freq                 | 0201          | 10    | [C400827](https://jlcpcb.com/partdetail/C400827) | [PDF](https://www.murata.com/en-us/products/productdata/8796840689694/GCM155C71A225KE36-01A.pdf)  |
| EPD_C1, EPD_C2, EPD_C6, EPD_C7, EPD_C8, EPD_C9, EPD_C10, EPD_C11, EPD_C12      | 1 µF / 50V (X7R)               | Flying caps charge-pump pentru VGH/VGL       | 0201          | 9     | [C105948](https://jlcpcb.com/partdetail/C105948) | —                                                                                                 |
| EPD_C5                                                                         | 0.1 µF / 50V                   | Decuplare driver EPD                          | 0201          | 1     | [C14663](https://jlcpcb.com/partdetail/C14663)   | —                                                                                                 |
| R1_USB, R2_USB                                                                 | 5.1 kΩ 1% (CC1 / CC2)           | Pull-down USB-C — configurare device UFP     | 0201          | 2     | [C98220](https://jlcpcb.com/partdetail/C98220)   | —                                                                                                 |
| R1_EP_DR, R2_EP_DR                                                             | 10 kΩ 1%                       | Divider / gate pull pentru boost EPD         | 0201          | 2     | [C25744](https://jlcpcb.com/partdetail/C25744)   | —                                                                                                 |
| R_PWR_EPD, R_TYPE_SEL                                                          | 10 kΩ 1%                       | Selector tip panou EPD / enable rail         | 0201          | 2     | [C25744](https://jlcpcb.com/partdetail/C25744)   | —                                                                                                 |
| R2, R3, R4                                                                     | 0 Ω                            | Jumperi / pull-uri I2C (SDA, SCL, ALERT)     | 0201          | 3     | [C17168](https://jlcpcb.com/partdetail/C17168)   | —                                                                                                 |
| R5, R7, R8                                                                     | 10 kΩ 1%                       | Pull-up butoane SW_UP, SW_DN, SW_ENT         | 0201          | 3     | [C25744](https://jlcpcb.com/partdetail/C25744)   | —                                                                                                 |
| R9                                                                             | 10 kΩ 1%                       | MR (TS / mode-select) pe BQ25180             | 0201          | 1     | [C25744](https://jlcpcb.com/partdetail/C25744)   | —                                                                                                 |
| R17, R18                                                                       | 3.3 kΩ 1%                      | Feedback / setare tensiune RT6160            | 0201          | 2     | [C22975](https://jlcpcb.com/partdetail/C22975)   | —                                                                                                 |
| TP_3V3, TP_3.3V, TP_VREG, TP_VBAT, TP_BAT_GND, TP_GND, TP_RESET, TP_SDA, TP_SCL, TP_SWDCLK, TP_SWDIO, TP_SWO, TP_ON, TP_OP | Pad 20 mil               | Puncte de test pentru măsurători / debug     | 20R pad       | 14    | (doar footprint)                                 | —                                                                                                 |
| SJ1                                                                            | Solder jumper                  | Select hardware (bootloader / mod)            | 0201 jumper   | 1     | (doar footprint)                                 | —                                                                                                 |


### Off-board

| Componentă          | Valoare / Model                      | Legătură                                   |
|---------------------|--------------------------------------|--------------------------------------------|
| Baterie             | LiPo 483450, 3.7V, 2400 mAh          | Conector J1 (Molex 503480-2400)            |
| Panou e-paper       | GDEW0154M09, 1.54" 200×200 px, SPI   | Conector FPC 24 pini (nu e pe placă — se leagă prin cablaj) |
| Motor vibrație      | ERM 10×2.7 mm, 3V nominal            | Pini OUT+ / OUT− ai DRV2605                |

---

## 3. Descrierea hardware detaliată

### 3.1. Alimentare

Lanțul de alimentare respectă o arhitectură clasică **charger → baterie → regulator**, adaptată pentru low-power:

1. **Intrare USB-C (J4)** — 5V / max 500 mA. Liniile D+/D− sunt protejate de **USBLC6-2 (D3)**, diodă TVS cu capacitate mică care nu afectează eye-pattern-ul USB 2.0 Full-Speed.
2. **Încărcător BQ25180 (IC1)** — integrează charger liniar 1-cell LiPo cu până la 500 mA, path management, detecție over-voltage 18V și **ship-mode**. Configurabil pe I2C: curent de încărcare, prag de terminare, temperatură. Pinul **INT** este conectat la nRF pentru evenimente.
3. **Baterie LiPo 2400 mAh** conectată la J1. Pinul TS/MR al BQ25180 nu folosește termistor.
4. **Fuel gauge MAX17048 (U3)** — măsoară direct tensiunea bateriei cu algoritmul **ModelGauge**, oferind SoC și rata de descărcare. Pinul ALERT este rutat la nRF pentru notificări de low-battery.
5. **Buck-Boost RT6160 (IC9)** — convertor **step-up/step-down** la **3V3 fix**, eficiență până la 95%. Alegerea unui buck-boost e critică: o baterie LiPo ajunge la ~3.0V la descărcare, iar un LDO ar avea dropout, în timp ce un buck pur nu ar putea menține 3V3 când bateria e sub acest nivel. RT6160 oferă continuitate până la ~2.5V cu regulare menținută.
6. **Load-switch Q1 (DMG2305UX-7)** — P-MOSFET care deconectează complet rail-ul boost al e-paper-ului în stand-by.

**Rețeaua RF**: antena ceramică ANT1 e alimentată prin **filtru π** pentru match la 50 Ω și filtrare armonici. Traseul de la pinul **ANT** al nRF la antenă respectă **controlled impedance** 50 Ω.

### 3.2. Microcontroller și clock

**nRF52840** rulează cu două cristale:

- **X1 = 32 MHz (NX2016SA)** pe pinii XC1/XC2, cristal principal pentru radio și HFCLK. Toleranță ≤ 40 ppm.
- **X2 = 32.768 kHz (ABS07)** pe pinii P0.00/XL1, P0.01/XL2, clock low-power pentru RTC și sleep. Toleranță 20 ppm, consum < 1 µA.

Decuplarea rail-urilor interne folosește 4.7 µF + 100 nF + 1 µF + 100 pF conform reference design Nordic. **L4 + L6**împreună cu C14 + C15 + C20 implementează DC/DC intern care reduce consumul cu ~30% față de modul LDO.

### 3.3. Display e-paper

E-paper-ul **GDEW0154M09** are nevoie de patru rail-uri: +15V (VGH), −15V (VGL), +22V (VPP), −20V (VEE). Driver circuit-ul de pe Sheet 2 folosește:

- **Boost cu SI1308 + L5 + MBR0530** care generează VGH/VGL prin charge-pump multi-stage.
- **9× condensatori 1µF/50V** — flying caps pentru stagele charge-pump.

Controlul e pe SPI + semnale discrete: RST, BUSY, D/C, CS. Rail-ul EPD_3V3 se activează doar când se refresh-uiește display-ul; în rest, e-paper-ul menține imaginea fără alimentare.

### 3.4. Senzori și actuatori

- **BMA423 (IC3)** — accelerometru 3-axis 12-bit cu engine de pași, detecție tap/double-tap și wrist-tilt. Întreruperi: INT1, INT2. Interfața I2C, adresa 0x19.
- **DRV2605 (IC2)** — driver haptic pentru motoare ERM sau LRA. Conține **librărie pre-programată cu 123 efecte**, plus mod real-time. Activat prin HAPTIC_EN pentru a elimina complet consumul în idle. Bobina motorului e conectată la pinii OUT+/OUT−.

### 3.5. Butoane și debug

Cele 3 butoane sunt conectate GPIO cu GND, cu pull-up intern activat software. **Fără debounce hardware** , se face în firmware.

Conectorul **TC2030-IDC (J2)** e un footprint 6-pin fără receptacle fizic, debugger-ul are pogo-pins care fac contact direct pe PCB, economisind componenta și spațiu. Semnale: SWDIO, SWDCLK, SWO, RESET, 3V3, GND.

---

## 4. Mapare pini nRF52840

Mapare extrasă din schematic (InkTime v6, Sheet 1 + Sheet 2):

| Pin nRF52840    | Nume net       | Funcție firmware                | Destinație periferic          |
|-----------------|----------------|---------------------------------|-------------------------------|
| **Alimentare**  |                |                                 |                               |
| VDD / VDDH      | 3V3            | Alimentare digital              | Rail 3V3 (de la RT6160)        |
| VBUS            | VBUS           | Detecție USB plug-in            | USB 5V                         |
| DEC1..DEC6      | DEC*           | Decuplare internă               | Local 100 nF × 6               |
| DCC / DCCH      | LX1 / LX2      | DC-DC intern (SMPS)             | L4 + L6 (10 µH) + filtre      |
| **Clock**       |                |                                 |                               |
| P0.00 / XL1     | XL1            | Cristal RTC 32.768 kHz          | X2                             |
| P0.01 / XL2     | XL2            | Cristal RTC 32.768 kHz          | X2                             |
| XC1 / XC2       | XC1 / XC2      | Cristal HF 32 MHz               | X1                             |
| **Radio**       |                |                                 |                               |
| ANT             | RF             | Ieșire RF                       | filtru π → ANT1                |
| **USB**         |                |                                 |                               |
| D+              | D+             | USB 2.0 Full-Speed              | J4 (prin USBLC6-2)            |
| D-              | D-             | USB 2.0 Full-Speed              | J4 (prin USBLC6-2)            |
| **Debug**       |                |                                 |                               |
| SWDCLK          | SWDCLK         | Clock debug                     | J2 pin 4                       |
| SWDIO           | SWDIO          | Data debug                      | J2 pin 2                       |
| P0.18 / nRESET  | RESET          | Reset hardware                  | J2 pin 5 + TP_RESET           |
| **SPI (TWIM1)** — e-paper |      |                                 |                               |
| P0.13           | EPD_SCK / SCK  | Clock SPI                       | FPC pin 5                      |
| P0.14           | EPD_MOSI / MOSI| Data SPI master→slave           | FPC pin 4                      |
| P0.15           | EPD_CS         | Chip-select                     | FPC pin 6                      |
| P0.16           | EPD_DC         | Data / Command                  | FPC pin 7                      |
| P0.17           | EPD_RST        | Reset display                   | FPC pin 8                      |
| P0.19           | EPD_BUSY       | Busy-in (drive done)            | FPC pin 9                      |
| **I2C (TWIM0)** — senzori |      |                                 |                               |
| P0.24           | SDA            | I2C data                        | BMA423 · DRV2605 · MAX17048 · BQ25180 |
| P0.25           | SCL            | I2C clock                       | BMA423 · DRV2605 · MAX17048 · BQ25180 |
| **Întreruperi** |                |                                 |                               |
| P0.22           | IMU_INT1       | IMU: pași / tap                 | BMA423 pin 5                   |
| P0.23           | IMU_INT2       | IMU: wake-up                    | BMA423 pin 6                   |
| P0.20           | ALERT          | Fuel gauge low-bat              | MAX17048 pin 5                 |
| P0.21           | PMIC_INT       | Charger events                  | BQ25180 pin A1                 |
| **Butoane**     |                |                                 |                               |
| P0.04           | SW_UP          | Buton sus                       | SW_UP (push-to-GND)            |
| P0.05           | SW_DN          | Buton jos                       | SW_DN (push-to-GND)            |
| P0.06           | SW_ENT         | Buton enter                     | SW_ENT (push-to-GND)           |
| **Control**     |                |                                 |                               |
| P0.26           | HAPTIC_EN      | Enable DRV2605 + boost EPD     | IC2 pin EN / Q3 gate          |
| P0.27           | EPD_PWR        | Load-switch panou e-paper       | Q1 gate                        |

---

## 5. Decizii de design

- **Utilizarea componentelor SMD în capsule mici** Am ales respectarea strictă a dimensiunilor impuse (0201 pentru rezistențe și condensatoare ≤100nF, respectiv 0402 pentru valori mai mari) pentru a reduce dimensiunea totală a PCB-ului. Această alegere contribuie la integrarea mai ușoară în carcasă și la un design compact, esențial pentru dispozitive portabile.
- **Amplasarea componentelor doar pe layer-ul TOP** Toate componentele au fost plasate pe TOP pentru a simplifica procesul de asamblare și pentru a evita costuri suplimentare asociate montajului pe ambele fețe. În plus, această abordare reduce riscul de erori mecanice și facilitează inspecția vizuală.
- **Excluderea planului de masă sub antenă** Zona de sub antenă a fost complet decuplată de planul de masă și nu au fost rutate semnale acolo. Aceasta previne detunarea antenei și pierderile de eficiență, asigurând performanță RF optimă.
- **Silkscreen minimalist și lizibil** Am inclus doar denumirile componentelor pentru a evita aglomerarea vizuală. Un silkscreen curat ajută la identificarea rapidă a componentelor și îmbunătățește mentenanța.
- **Utilizarea unui PCB cu 4 layere** Am ales o structură pe 4 layere (TOP, GND plane, POWER plane, BOTTOM) pentru a îmbunătăți semnificativ integritatea semnalului și stabilitatea alimentării.