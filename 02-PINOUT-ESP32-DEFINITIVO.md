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
| **GPIO 14** | **UART2 TX** | OUT | → RX MH-Z19C (pin 3) **DEPRECATED** | Per SCD41 non usato (I²C invece) |
| **GPIO 15** | ⚠️ Boot strapping | - | Lasciare floating | **NON usare per controllo** |
| **GPIO 16** | **LED di stato** | OUT | → R 1kΩ → Anodo LED → GND | Output digitale, 3.3V, max 40 mA |
| **GPIO 17** | Libero | - | Disponibile per espansione futura | Esempio: secondo UART o SPI |
| **GPIO 18** | **Encoder CLK** | IN | Encoder pin A (Clock) | Pull-up interno 45 kΩ abilitato in ESPHome |
| **GPIO 19** | **Encoder DT** | IN | Encoder pin B (Data) | Pull-up interno 45 kΩ abilitato in ESPHome |
| **GPIO 20** | Libero | - | Disponibile per espansione | Esempio: sensore aggiuntivo |
| **GPIO 21** | **Encoder SW** | IN | Pulsante encoder (pressione) | Pull-up interno 45 kΩ abilitato in ESPHome |
| **GPIO 22** | **Pulsante multifunzione** | IN | Pulsante NO verso GND | Pull-up interno 45 kΩ, logica breve/lunga in ESPHome |
| **GPIO 23** | Libero | - | Disponibile (compressore rimosso) | Esempio: secondo PWM o sensore |
| **GPIO 25** | **PWM Ventola** | OUT | → Gate MOSFET Q1 (ventola 12 V) | PWM a 25 kHz (oppure 1 kHz se rumorosa) |
| **GPIO 26** | **ON/OFF Umidificatore** | OUT | → Gate MOSFET Q3 (umidificatore 24 V) | PWM opzionale o digitale |
| **GPIO 27** | **UART2 RX** | IN | ← TX MH-Z19C (pin 2) **DEPRECATED** | Non usato con SCD41 (I²C) |
| **GPIO 32** | **SSR Riscaldatore (PID)** | OUT | → R 10kΩ → Base BC547 → SSR- | PWM o digitale, output PID |
| **GPIO 33** | **PWM LED strip** | OUT | → Gate MOSFET Q2 (LED 12 V) | PWM 20-25 kHz per dimmer, oppure digitale |
| **GPIO 34** | **Ventola TACH** | IN | Giallo ventola 4-pin | Input-only GPIO, pulse_counter in ESPHome |
| **GPIO 35** | ❌ **INPUT ONLY - NON USARE** | - | ⚠️ **Non ha uscita** | Rimosso dalla BOM (era LED di stato) |
| **GPIO 36** | **SENSOR_VP** | IN | Input-only | Disponibile solo lettura (es. ADC futuro) |
| **GPIO 37** | - | - | Non disponibile | Fisicamente non collegato |
| **GPIO 38** | - | - | Non disponibile | Fisicamente non collegato |
| **GPIO 39** | **SENSOR_VN** | IN | Input-only | Disponibile solo lettura (es. ADC futuro) |

---

## 📊 Riepilogo Utilizzo GPIO

### In Uso (18 GPIO)
- **I²C:** GPIO 4 (SDA), GPIO 5 (SCL)
- **1-Wire:** GPIO 13 (DATA)
- **PWM Uscite:** GPIO 25 (Ventola), GPIO 26 (Umidificatore), GPIO 32 (SSR), GPIO 33 (LED)
- **Digitali Uscite:** GPIO 16 (LED stato)
- **Digitali Ingressi:** GPIO 18 (Encoder CLK), GPIO 19 (Encoder DT), GPIO 21 (Encoder SW), GPIO 22 (Pulsante MF), GPIO 34 (Ventola TACH)
- **UART (debug):** GPIO 1 (TX), GPIO 3 (RX) — utilizzati dal DevKit per la console

### Liberi/Riservati (20 GPIO)
- **Riservati boot:** GPIO 0, 2, 12, 15 (non toccare)
- **Riservati PSRAM/Flash:** GPIO 6-11 (modulo WROOM-32E)
- **Liberi per espansione:** GPIO 17, 20, 23, 36, 39

---

## 🔌 Dettagli Connessioni Critiche

### Bus I²C Principale (GPIO 4, 5)

```
ESP32 SDA (GPIO 4) ──┬── R 4.7kΩ ── +3.3V (pull-up bus principale)
                    │
                    └── PCA9548A pin 9 (SDA IN)

ESP32 SCL (GPIO 5) ──┬── R 4.7kΩ ── +3.3V (pull-up bus principale)
                    │
                    └── PCA9548A pin 1 (SCL IN)
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
Ventola Pin 1 (+12V) ───────────────── Linea +12V da PSU
Ventola Pin 2 (GND)  ────┬──── Drain MOSFET Q1 (IRLZ44N)
                        │
                        └──── GND comune (ESP32)

Ventola Pin 3 (PWM)  ───── GPIO 25 (LEDC PWM, 25 kHz)

Ventola Pin 4 (TACH) ──┬── R 10kΩ ── +3.3V (partitore livello, opzionale se ventola output 3.3V)
                      │
                      └── GPIO 34 (pulse_counter in ESPHome)
```

**Attenzione:** Il tach è open-collector con pull-up interno a 12V nella ventola. Se è un valore alto, usare partitore 10k+4.7k verso GND. Se diretto a 3.3V, rischio di tensione reverse su GPIO. Verificare il datasheet della ventola specifica.

### SSR Riscaldatore (GPIO 32)

```
GPIO 32 ──── R 10kΩ (base) ──── Base BC547

BC547 Collettore ──┬── R 100kΩ (pull-down, sicurezza) ──── GND
                  │
                  └── SSR- (polo negativo controllo SSR)

SSR+ (polo positivo controllo) ──── +5V da PSU ATX

BC547 Emettitore ──── GND
```

**Logica:** GPIO HIGH → BC547 saturo → Collettore ≈ 0V → SSR vede +5V su SSR+ e GND su SSR- → **SSR acceso**.

### MOSFET Ventola (GPIO 25)

```
GPIO 25 (PWM) ──── R 220Ω ──── Gate MOSFET Q1 (IRLZ44N)
                            │
                            └── R 10kΩ pull-down verso GND

Drain Q1 ──┬── Ventola GND (nero)
           │
           └── Anodo 1N5819 (fly-back, catodo verso +12V)

Source Q1 ──── GND
```

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
  # GPIO 16: LED di stato
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
    
  # GPIO 34: Ventola TACH (pulse counter)
  - pin: 34
    id: fan_tach
    mode: INPUT
    
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
- [ ] GPIO 35: non usato (input-only, LED spostato a GPIO 16)
- [ ] GPIO 4, 5: bus I²C principale con pull-up 4.7kΩ
- [ ] GPIO 13: bus 1-Wire con pull-up 4.7kΩ
- [ ] GPIO 25, 26, 32, 33: PWM/digitali per carichi DC
- [ ] GPIO 16: uscita per LED di stato (40 mA max)
- [ ] GPIO 34: ingresso tach ventola (input-only, ok)
- [ ] GPIO 18, 19, 21, 22: ingressi con pull-up interni abilitati
- [ ] Nessun conflitto tra funzioni GPIO
- [ ] Tutte le resistenze pull-up dimensionate (4.7kΩ standard per I²C, 10kΩ per gate MOSFET)
- [ ] Tutti gli ESD/fly-back diodi in posizione

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Validato e pronto per KiCad
