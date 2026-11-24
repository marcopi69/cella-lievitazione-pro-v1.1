# PINOUT ESP32 DEFINITIVO — v1.1

## GPIO Mapping Completo

| GPIO | Funzione | Direzione | Collegamento Hardware | Note |
|------|----------|-----------|----------------------|-------|
| **VIN** | Alimentazione +5V input | IN | Linea +5 V da PSU ATX | Max 12 V, regolatore interno a 3.3 V |
| **GND** | Massa comune | - | Stella di massa PCB | Centrale, vicino ai carichi ad alta corrente |
| **EN** | Reset hardware | IN | Pulsante NO verso GND + C 100nF | Pull-up interno 10k, attivo basso |
| **GPIO 0** | ⚠️ BOOT MODE | - | Lasciare floating (alto al boot) | **NON usare per controllo, riservato** |
| **GPIO 2** | ⚠️ Boot strapping | - | Lasciare floating | **NON usare per controllo** |
| **GPIO 3** | UART0 RX | IN | Debug UART (seriale PC via USB-UART) | Usato dal bootloader |
| **GPIO 1** | UART0 TX | OUT | Debug UART (seriale PC via USB-UART) | Usato dal bootloader |
| **GPIO 4** | **I²C SDA** | I/O | → PCA9548A pin 9 (SDA) | Pull-up 4.7 kΩ verso +3.3 V (bus principale) |
| **GPIO 5** | **I²C SCL** | I/O | → PCA9548A pin 1 (SCL) | Pull-up 4.7 kΩ verso +3.3 V (bus principale) |
| **GPIO 6** | PSRAM CS | - | Interno modulo | **NON usare, riservato** |
| **GPIO 7** | PSRAM/Flash | - | Interno modulo | **NON usare, riservato** |
| **GPIO 8** | PSRAM/Flash | - | Interno modulo | **NON usare, riservato** |
| **GPIO 9** | PSRAM/Flash | - | Interno modulo | **NON usare, riservato** |
| **GPIO 10** | PSRAM/Flash | - | Interno modulo | **NON usare, riservato** |
| **GPIO 11** | PSRAM/Flash | - | Interno modulo | **NON usare, riservato** |
| **GPIO 12** | ⚠️ Boot strapping | - | Lasciare floating | **NON usare per controllo** |
| **GPIO 13** | **1-Wire DATA** | I/O | Bus DS18B20 (entrambe le sonde) | Pull-up 4.7 kΩ verso +3.3 V (obbligatorio 1-Wire) |
| **GPIO 14** | **UART2 TX (Display)** | OUT | → R 1kΩ → RX Nextion/TJC | 115200 baud; breve PWM al boot (ignorato dal display) |
| **GPIO 15** | ⚠️ Boot strapping | - | Lasciare floating | **NON usare per controllo** |
| **GPIO 16** | **LED RGB di stato (WS2812)** | OUT | → R 330Ω → DIN WS2812 (alimentato +5V con diodo in serie) | Dato 3.3V, 1 filo |
| **GPIO 17** | Libero | I/O | Disponibile per espansione futura | Pienamente utilizzabile (I/O, PWM, UART, 1-Wire) |
| **GPIO 18** | **Encoder CLK** | IN | Encoder pin A (Clock) | Pull-up interno 45 kΩ abilitato in ESPHome |
| **GPIO 19** | **Encoder DT** | IN | Encoder pin B (Data) | Pull-up interno 45 kΩ abilitato in ESPHome |
| **GPIO 20** | - | - | Non esposto sul DevKitC V4 | Non presente sul connettore del modulo |
| **GPIO 21** | **Encoder SW** | IN | Pulsante encoder (pressione) | Pull-up interno 45 kΩ abilitato in ESPHome |
| **GPIO 22** | **Pulsante multifunzione** | IN | Pulsante NO verso GND | Pull-up interno 45 kΩ, logica breve/lunga in ESPHome |
| **GPIO 23** | **Pulsante luci LED** | IN | Pulsante NO verso GND | Pull-up interno 45 kΩ, toggle striscia LED |
| **GPIO 25** | **PWM Ventola** | OUT | → Gate MOSFET Q1 (ventola 12 V) | PWM a 25 kHz (oppure 1 kHz se rumorosa) |
| **GPIO 26** | **ON/OFF Umidificatore** | OUT | → Gate MOSFET Q3 (umidificatore 24 V) | PWM opzionale o digitale |
| **GPIO 27** | **UART2 RX (Display)** | IN | ← TX Nextion/TJC | 115200 baud; eventi touch dal display |
| **GPIO 32** | **SSR Riscaldatore (PID)** | OUT | → R 10kΩ → Base BC547 → SSR- | PWM o digitale, output PID |
| **GPIO 33** | **PWM LED strip** | OUT | → Gate MOSFET Q2 (LED 12 V) | PWM 20-25 kHz per dimmer, oppure digitale |
| **GPIO 34** | **Ventola TACH** | IN | Giallo ventola 4-pin | Input-only GPIO, pulse_counter in ESPHome |
| **GPIO 35** | Libero (solo ingresso) | IN | Input-only, ADC1 | Utilizzabile come ingresso (no pull-up interno) |
| **GPIO 36** | **SENSOR_VP** | IN | Input-only, ADC1 | Libero: solo lettura (no pull-up interno) |
| **GPIO 37** | - | - | Non disponibile | Fisicamente non collegato |
| **GPIO 38** | - | - | Non disponibile | Fisicamente non collegato |
| **GPIO 39** | **SENSOR_VN** | IN | Input-only, ADC1 | Libero: solo lettura (no pull-up interno) |

---

## 📊 Riepilogo Utilizzo GPIO

### In Uso (16 GPIO + debug UART0)
- **I²C:** GPIO 4 (SDA), GPIO 5 (SCL)
- **1-Wire:** GPIO 13 (DATA)
- **UART2 Display Nextion/TJC:** GPIO 14 (TX), GPIO 27 (RX)
- **PWM Uscite:** GPIO 25 (Ventola), GPIO 26 (Umidificatore), GPIO 32 (SSR), GPIO 33 (LED)
- **Digitali Uscite:** GPIO 16 (LED stato)
- **Digitali Ingressi:** GPIO 18 (Encoder CLK), GPIO 19 (Encoder DT), GPIO 21 (Encoder SW), GPIO 22 (Pulsante MF), GPIO 23 (Pulsante luci LED), GPIO 34 (Ventola TACH)
- **UART0 (debug):** GPIO 1 (TX), GPIO 3 (RX) — console via USB del DevKit

### Liberi per espansione

| GPIO | Tipo | Note |
|---|---|---|
| **17** | I/O completo | Il più versatile: uscita, PWM, UART, 1-Wire, secondo bus |
| **35** | Solo ingresso | ADC1, senza pull-up interno |
| **36** (VP) | Solo ingresso | ADC1, senza pull-up interno |
| **39** (VN) | Solo ingresso | ADC1, senza pull-up interno |

Condizionali: GPIO 2, 12, 15 (strapping) solo con cautela; GPIO 1/3 se si rinuncia al debug UART.

### Connettore di espansione J_EXP (sulla PCB)

Un unico connettore a vite/passo 2.54 mm, 8 poli, porta fuori i pin liberi per moduli esterni (sensore livello acqua, sonda di corrente, relè, buzzer, ecc.):

| Pin | Segnale | Protezione sulla PCB |
|---|---|---|
| 1 | +3V3 | Fusibile PTC non necessario (max 100 mA dal regolatore) |
| 2 | GND | - |
| 3 | EXP_IO17 (GPIO 17) | Resistenza serie 330 Ω; I/O, PWM, UART, 1-Wire |
| 4 | EXP_IN35 (GPIO 35) | Serie 1 kΩ + 100 nF verso GND, ingresso ADC |
| 5 | EXP_IN36 (GPIO 36) | Serie 1 kΩ + 100 nF verso GND, ingresso ADC |
| 6 | EXP_IN39 (GPIO 39) | Serie 1 kΩ + 100 nF verso GND, ingresso ADC |
| 7 | +5V (dopo F2) | Max 200 mA per moduli esterni |
| 8 | GND | - |

Regole d'uso:
- I segnali sono a **3.3 V**: non applicare mai 5 V diretti ai pin (usare partitore o level shifter).
- GPIO 35/36/39 sono solo ingressi e **senza pull-up interno**: eventuali pull-up/pull-down vanno sul modulo esterno.
- Per sensori I²C aggiuntivi non servono questi pin: restano liberi sei canali del PCA9548A (0, 1, 2, 5, 6, 7).
- Nel YAML, un nuovo modulo si aggiunge senza modificare l'hardware (esempio: `binary_sensor` su GPIO35 per un galleggiante).

### Riservati / non disponibili
- **Boot:** GPIO 0, 2, 12, 15 (non toccare)
- **Flash SPI:** GPIO 6-11 (modulo WROOM-32E)
- **Non esposti sul DevKitC V4:** GPIO 20, 24, 28-31, 37, 38

---

## 🔌 Dettagli Connessioni Critiche

### Bus I²C Principale (GPIO 4, 5)

```
ESP32 SDA (GPIO 4) ──┬── R 4.7kΩ ── +3.3V (pull-up bus principale)
                    │
                    └── PCA9548A pin 23 (SDA)

ESP32 SCL (GPIO 5) ──┬── R 4.7kΩ ── +3.3V (pull-up bus principale)
                    │
                    └── PCA9548A pin 22 (SCL)

PCA9548A (TSSOP-24): pin 24 VCC = +3.3V, pin 12 GND,
  pin 1 (A0), pin 2 (A1), pin 21 (A2) a GND → indirizzo 0x70,
  pin 3 (RESET) a +3.3V tramite 10 kΩ (+ 100 nF verso GND), VCC con 100 nF vicino al chip.
```

**Nota:** Pull-up unici sul bus principale. Ogni canale del PCA9548A ha i propri pull-up sui pin SCx/SDx.

### Bus 1-Wire (GPIO 13)

```
ESP32 GPIO13 ──┬── R 4.7kΩ ── +3.3V (pull-up obbligatorio 1-Wire)
               │
               ├── DS18B20 #1 DATA (cavo schermato)
               │
               └── DS18B20 #2 DATA (stesso cavo schermato)

DS18B20 GND ──── Massa comune ESP32
DS18B20 VCC ──── +3.3V (alimentazione parassita NON raccomandata, usare sempre VCC a 3.3V)
```

### Ventola 4-pin PWM (GPIO 25, 34)

```
Ventola Pin 1 (+12V) ───────────────── Linea +12V (dopo F1)
Ventola Pin 2 (GND)  ───────────────── GND comune

Ventola Pin 3 (PWM)  ───── R 220Ω ───── GPIO 25 (LEDC PWM, 25 kHz)

Ventola Pin 4 (TACH) ──┬── R 10kΩ ── +3.3V
                       └── R 1kΩ ── GPIO 34 (pulse_counter in ESPHome)
```

**Attenzione:** misurare con il tester la tensione del TACH a ventola alimentata. Se supera 3.6 V (pull-up interno a 12 V) usare il partitore 15 kΩ / 4.7 kΩ descritto in `15-VENTOLA-PWM-4PIN.md`.

### SSR Riscaldatore (GPIO 32)

```
GPIO 32 ──── R 10kΩ (base) ──┬── Base BC547
                             └── R 100kΩ ──── GND  (pull-down: SSR spento all'avvio)

BC547 Collettore ──── SSR- (polo negativo controllo SSR)

SSR+ (polo positivo controllo) ──── +5V da PSU

BC547 Emettitore ──── GND
```

**Logica:** GPIO HIGH → BC547 saturo → Collettore ≈ 0V → SSR vede +5V su SSR+ e GND su SSR- → **SSR acceso**. Con GPIO flottante (boot, reset) il pull-down sulla base tiene il BC547 spento e quindi l'SSR spento.

### MOSFET Q1 (libero)

Il MOSFET Q1 non è più usato per la ventola (vedi `15-VENTOLA-PWM-4PIN.md`, sez. 5). Resta disponibile come uscita 12 V ON/OFF di riserva, pilotabile da GPIO 17 (J_EXP).

---

## ⚡ Livelli di Tensione

| Linea | Tensione | Tolleranza GPIO | Note |
|-------|----------|-----------------|-------|
| **GPIO digitali output** | 0 / 3.3V | - | Logic-level, 40 mA max per pin |
| **GPIO digitali input** | 0 / 3.3V | 0-3.6V | Tollerante a 3.3V |
| **GPIO ADC (non usato qui)** | 0-3.3V | 0-3.6V | 12-bit |
| **I²C SDA/SCL** | 3.3V nominale | 0-5V** | Tollerante a 5V con pull-up a 3.3V |
| **1-Wire DATA** | 3.3V nominale | 0-5V** | Tollerante a 5V con pull-up a 3.3V |

**Attenzione:** Se i sensori esterni (ventola TACH, SCD41, etc.) hanno livelli a 5V, usare partitore o optoisolatore. L'ESP32 non gradisce tensioni > 3.6V permanenti sui GPIO.

---

## 🔧 Configurazione ESPHome — Snippet GPIO

```yaml
esphome:
  name: cella-lievitazione
  platform: esp32
  board: esp32dev

gpio:
  # GPIO 16: LED RGB di stato WS2812 (vedi 21-ENCODER-PULSANTI.md, sez. 4)
  - pin: 16
    id: led_stato
    mode: OUTPUT
    
  # GPIO 25: PWM Ventola
  - pin: 25
    id: pwm_fan
    mode: OUTPUT
    
  # GPIO 26: PWM/ON-OFF Umidificatore
  - pin: 26
    id: pwm_humid
    mode: OUTPUT
    
  # GPIO 32: SSR Riscaldatore
  - pin: 32
    id: ssr_heater
    mode: OUTPUT
    
  # GPIO 33: PWM LED strip
  - pin: 33
    id: pwm_led
    mode: OUTPUT
    
  # GPIO 18: Encoder CLK
  - pin: 18
    id: enc_clk
    mode: INPUT
    
  # GPIO 19: Encoder DT
  - pin: 19
    id: enc_dt
    mode: INPUT
    
  # GPIO 21: Encoder SW
  - pin: 21
    id: enc_sw
    mode: INPUT
    
  # GPIO 22: Pulsante multifunzione
  - pin: 22
    id: btn_mf
    mode: INPUT

  # GPIO 23: Pulsante luci LED
  - pin: 23
    id: btn_luci
    mode: INPUT
    
  # GPIO 34: Ventola TACH (pulse counter)
  - pin: 34
    id: fan_tach
    mode: INPUT
    
# GPIO 14/27: UART2 verso display Nextion (vedi 20-DISPLAY-NEXTION.md)
uart:
  tx_pin: 14
  rx_pin: 27
  baud_rate: 115200

i2c:
  sda: 4
  scl: 5
  scan: true
  
dallas:
  pin: 13
  update_interval: 10s
```

---

## ✅ Checklist di Validazione

- [ ] GPIO 0, 2, 12, 15: non usati per controllo (riservati boot)
- [ ] GPIO 35, 36, 39: input-only, liberi (nessuna uscita)
- [ ] GPIO 14, 27: UART2 verso display Nextion/TJC (TX con R 1kΩ)
- [ ] GPIO 23: pulsante luci LED con pull-up interno
- [ ] GPIO 4, 5: bus I²C principale con pull-up 4.7kΩ
- [ ] GPIO 13: bus 1-Wire con pull-up 4.7kΩ
- [ ] GPIO 25, 26, 32, 33: PWM/digitali per carichi DC
- [ ] GPIO 16: uscita per LED di stato (40 mA max)
- [ ] GPIO 34: ingresso tach ventola (input-only, ok)
- [ ] GPIO 18, 19, 21, 22, 23: ingressi con pull-up interni abilitati
- [ ] Nessun conflitto tra funzioni GPIO
- [ ] Tutte le resistenze pull-up dimensionate (4.7kΩ standard per I²C, 10kΩ per gate MOSFET)
- [ ] Tutti gli ESD/fly-back diodi in posizione

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Validato e pronto per KiCad
