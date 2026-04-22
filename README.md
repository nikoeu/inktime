# Proiect Smartwatch - Documentație Hardware

## 1. Diagramă Bloc

Mai jos este prezentată arhitectura sistemului și modul în care comunică principalele module:

    +-------------------+
    |                   |
    |   BQ25180YBGR     | <==== I2C ====> 
    | (PMIC & Charger)  |                 |
    |                   | --- VREG 3.3V --+
    +-------------------+                 |
                                          |
    +-------------------+                 v
    |                   |
    |                   | <==== I2C ====> IMU (Accel/Gyro)
    |     nRF52840      |
    |  (SoC Principal)  | <==== SPI ====> Ecran E-Paper (EPD)
    |                   |
    |                   | ==== GPIO ====> Driver / Motor Haptic
    +-------------------+

**Conexiuni principale:**

* **Alimentare:** Port USB-C ➜ BQ25180 (PMIC) ➜ Baterie Li-Po & VREG (3.3V System Power). MCU-ul se alimentează din pinul VREG.
* **Periferice I2C:** PMIC-ul (Control/Status) și IMU-ul împart aceeași magistrală I2C.
* **Periferice SPI:** E-Paper Display (EPD) este conectat pe magistrala SPI dedicată.
* **I/O Adițional:** 3 Butoane tactile, Interfața SWD pentru programare și controlul motorului haptic.

## 2. Bill of Materials (BOM)

Tabelul de mai jos conține modulele și componentele principale folosite în designul PCB-ului. Din considerente de formatare pentru recenzie, link-urile directe au fost omise, componentele fiind disponibile standard în catalogul JLC Parts.

| Componentă / Modul | Rol în sistem | Sursă | Documentație |
|---|---|---|---|
| **nRF52840** | Microcontroler principal (BGA) cu BLE | JLC Parts | Datasheet Producător |
| **BQ25180YBGR** | PMIC / Încărcător baterie Li-Po | JLC Parts | Datasheet Producător |
| **LSM6DS3 / MPU6050** | Senzor de mișcare (IMU - Accel/Giroscop) | JLC Parts | Datasheet Producător |
| **Modul EPD SPI** | Conector și suport pentru ecranul E-Paper | JLC Parts | Datasheet Producător |
| **Modul Haptic** | Driver / Motor Vibrații pentru feedback tactil | JLC Parts | Datasheet Producător |
| **Conector USB-C** | Intrare alimentare (VBUS) | JLC Parts | Datasheet Producător |
| **Butoane Tactile SMD** | Interacțiune utilizator (GND pe apăsare) | JLC Parts | Datasheet Producător |

## 3. Descrierea Funcționalității Hardware

Sistemul hardware este proiectat în jurul SoC-ului **nRF52840**, ales pentru capabilitățile sale excelente de Bluetooth Low Energy (BLE) și consumul redus, ideal pentru un dispozitiv purtabil.

* **Managementul Puterii (PMIC):** Am folosit integratul **BQ25180**, un PMIC și încărcător liniar extrem de compact. Acesta preia tensiunea de 5V din portul USB (VBUS), gestionează încărcarea sigură a bateriei (VBAT) și furnizează o tensiune stabilizată sistemului prin pinul VREG/SYS (stabilită la 3.3V). Comunicația cu MCU-ul se face prin magistrala I2C pentru a citi starea bateriei și a configura curentul de încărcare.
* **Interfața cu utilizatorul:** Afișajul este realizat printr-un ecran E-Paper, conectat via magistrala **SPI**. Acest tip de display a fost ales datorită consumului zero de energie pentru reținerea imaginii (esențial pentru autonomia unui ceas). Navigarea se face prin 3 butoane fizice (active low), iar feedback-ul tactil este asigurat de un modul haptic.
* **Senzoristică:** Modulul IMU comunică cu MCU-ul tot prin interfața **I2C** (partajată cu PMIC-ul) și dispune de pini de întrerupere hardware (INT1/INT2) pentru a "trezi" microcontrolerul doar la detectarea mișcării (ex: ridicarea mâinii), economisind astfel baterie.
* **Consum:** În mod *Deep Sleep*, consumul microcontrolerului împreună cu PMIC-ul este de ordinul microamperilor (uA). La activarea BLE și a ecranului (refresh), consumul crește temporar. Sistemul este proiectat astfel încât perifericele să poată fi adormite sau oprite independent.

## 4. Maparea Pinilor nRF52840

Pinii au fost aleși strategic pentru a optimiza rutarea pe PCB sub pachetul BGA, evitând pe cât posibil intersecțiile majore.

| Pin / Label | Funcție asociată | Justificare / Protocol |
|---|---|---|
| **SDA / SCL** | I2C Data & Clock | Magistrala I2C principală. Conectează MCU la BQ25180 și la IMU. |
| **EPD_CS** | Chip Select (Ecran) | Controlează activarea comunicării SPI cu ecranul E-Paper. |
| **EPD_DC** | Data / Command (Ecran) | Indică ecranului dacă datele trimise pe SPI sunt setări sau pixeli. |
| **EPD_RST** | Reset (Ecran) | Reset hardware dedicat pentru inițializarea display-ului. |
| **EPD_BUSY** | Busy State (Ecran) | Intrare în MCU; previne trimiterea de date noi în timp ce ecranul face refresh. |
| **IMU_INT1 / INT2** | Întreruperi Senzor | Trezesc MCU-ul din starea de sleep la detectarea mișcării. |
| **PMIC_INT** | Întrerupere Baterie | Alertează MCU-ul la conectarea cablului USB sau low-battery. |
| **HAPTIC_EN** | Enable Vibrații | Pin GPIO/PWM pentru controlul duratei și intensității vibrațiilor. |
| **RESET** | System Reset | Conectat la pad-urile de test (TP) pentru depanare via SWD. |
| **SWDIO / SWDCLK** | Interfață Programare | Pini dedicați pentru programarea și depanarea nRF52840. |

## 5. Jurnal de Design (Decizii, Probleme și Compromisuri)

Rutarea acestui PCB a reprezentat o provocare majoră din cauza densității mari de componente specifice unui smartwatch și a pachetului BGA al microcontrolerului. Următoarele decizii tehnice au fost luate și asumate pentru a asigura un design funcțional:

1. **Tranziția la un Stack-up cu 4 Straturi (4-Layer PCB):**
   * Datorită aglomerării extreme de pe stratul Top și a numărului mare de airwires rezultate din constrângerile mecanice, am decis utilizarea a 4 straturi.
   * **Layer 2 (Inner 1)** a fost transformat integral într-un plan de masă solid (**GND Polygon Pour**), asigurând ecranare electromagnetică și trasee de întoarcere scurte.
   * **Layer 15 (Inner 2)** a fost utilizat ca plan de alimentare (**3V3 / VREG**), curățând placa de necesitatea rutării zecilor de fire de putere.

2. **Compromisuri acceptate în DRC (Via-in-Pad și Overlap):**
   * **Problemă:** Pinii interni ai microcontrolerului nRF52840 (BGA) erau imposibil de rutat folosind tehnica tradițională ("dog-bone"), spațiul dintre pini fiind insuficient.
   * **Soluție:** Am apelat la tehnica **Via-in-Pad** (Drill: 0.15mm, Annular Ring: 0.075mm pentru a atinge diametrul total de 0.3mm acceptat de fabrica JLC).
   * **Decizie Asumată:** Am acceptat și aprobat manual erorile de tip *Clearance* și *Overlap* generate de soft pe pad-urile BGA-ului, acestea fiind o consecință intenționată (aprobată în prealabil) a utilizării tehnicii Via-in-Pad.

3. **Rutarea firelor de putere prin Vias:**
   * Deși bunele practici recomandă ca traseele de putere principale să fie menținute pe un singur strat continuu pentru a evita rezistența parazită, **am fost nevoit să trag 3 cabluri de putere (grosime 0.3mm) prin vias** pe straturile inferioare. Aglomerarea fizică din jurul zonei PMIC-ului și conectorului USB nu a permis altă soluție de escape routing exclusiv pe stratul TOP.

4. **Amplasarea Antenei (Zona de Restrict):**
   * Antena Bluetooth a fost plasată pe marginea PCB-ului conform cerințelor. Am desenat un decupaj specific în conturul plăcii (Board Outline) și am aplicat reguli de *Keepout* (excludere a poligoanelor de cupru) pe toate cele 4 straturi sub corpul antenei, interzicând rutarea semnalelor pe dedesubt pentru a respecta normele de design RF.

## 6. Limitări curente / Stadiul proiectului

* Designul PCB este complet rutat și trece verificările mecanice principale.
* **Componentă lipsă:** Datorită focusului pe rezolvarea constrângerilor complexe de rutare hardware (BGA + densitate mare de componente) și a timpului alocat asamblării cablajului, **partea de modelare 3D (designul carcasei smartwatch-ului) nu a fost realizată** în acest stadiu al proiectului.
