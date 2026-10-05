# BUS I²C — PCA9548A, SHT45, SCD41 v1.1

## 1. Architettura Bus I²C Muxato

Il bus I²C dell'ESP32 (GPIO 4 SDA, GPIO 5 SCL) è muxato tramite **PCA9548A** per gestire più sensori sullo stesso indirizzo senza conflitti.

### Vantaggi Multiplexing

```
Senza PCA9548A (PROBLEMA):
  SHT45 indirizzo 0x44 ─────┐
  SCD41 indirizzo 0x44 ─────┼─── Bus I2C unico
  OLED indirizzo 0x3C ──────┘
  
  ⚠ Due dispositivi (SHT45 + SCD41) stesso indirizzo = CONFLITTO! ❌

Con PCA9548A (SOLUZIONE):
  SHT45 (0x44) ──────── Canale 3 ─┐
  SCD41 (0x44) ──────── Canale 4 ─┼── PCA9548A (0x70) ──── Bus I2C ESP32
  OLED (0x3C) ────────── Display ─┘
  
  ✅ Ogni dispositivo su canale separato, stesso indirizzo OK!
```

---

## 2. PCA9548A — Specifiche

| Parametro | Valore | Note |
|---|---|---|
| **Tipo** | I²C 8-channel multiplexer | Switch 1:1 o demultiplexing |
| **Canali** | 8 (0-7) | Selezionabili via I²C |
| **Indirizzo I²C** | 0x70 (A0=A1=A2=GND) | Configurabile con pin A0-A2 |
| **Frequenza** | 100 kHz a 400 kHz | Supporta Fast mode |
| **Alimentazione** | 3.3V a 5V | Tollerante |
| **Corrente** | ~1 mA inattivo, <5 mA attivo | Efficiente |
| **Canale singolo ON** | Solo 1 alla volta | Commutazione automatica |
| **Reset** | Pin RST attivo basso | Pull-up 1k verso +3.3V |
| **Tempo switch** | <100 ns | Instant switching |

---

## 3. Circuito Completo I²C + PCA9548A

```
ESP32 GPIO 4 (SDA) ────┬─── R 4.7kΩ ─── +3.3V (pull-up bus principale)
                      │
                  ┌───┴──┬─── PCA9548A (U6, addr. 0x70)
                  │      │    Pin 9: SDA IN
                  │      │    Pin 1: SCL IN
ESP32 GPIO 5 (SCL) ────┬─── R 4.7kΩ ─── +3.3V (pull-up bus principale)
                      │
                  ┌───┴──┴─── A0/A1/A2: GND (indirizzo 0x70)
                  │          RST: +3.3V tramite R 1kΩ
                  │          VCC: +3.3V
                  │          GND: GND
                  │
     ┌────────────┼────────────┬────────────────────────────────────────┐
     │            │            │                                        │
     │            │            │                                        │

Canale 0 (libero):       Canale 3 (SHT45):         Canale 4 (SCD41):
  SC0/SD0                  SC3/SD3                   SC4/SD4
  (pull-up R 4.7k)         (pull-up R 4.7k)         (pull-up R 4.7k)
                      ┌─ R 4.7kΩ ─ +3.3V      ┌─ R 4.7kΩ ─ +3.3V
                      │                        │
                  ┌───┼──┐                 ┌───┼──┐
                  │SHT45 │                 │SCD41 │
                  │      │                 │      │
                  │ SDA: SD3               │ SDA: SD4
                  │ SCL: SC3               │ SCL: SC4
                  │ VCC: +3.3V             │ VCC: +3.3V
                  │ GND: GND               │ GND: GND
                  │ addr:0x44              │ addr:0x62
                  └──────┘                 └──────┘

     │            │            │                                        │
     │            │            │                                        │

Canali 1, 2: OLED displays (se usati)
Canali 5, 6, 7: Riservati per espansione futura
```

---

## 4. Configurazione ESPHome

```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
  frequency: 100kHz
  id: bus_a

# Multiplexer PCA9548A
tca9548a:
  i2c_id: bus_a
  address: 0x70
  id: pca_multiplexer

# Sensore SHT45 su Canale 3
sensor:
  - platform: sht4x
    i2c_id: pca_multiplexer
    i2c_address: 0x44
    channel: 3
    update_interval: 10s
    
    temperature:
      name: "Temperatura (SHT45)"
      id: sht45_temp
      filters:
        - offset: 0.0
    
    humidity:
      name: "Umidità Relativa (SHT45)"
      id: sht45_rh

# Sensore SCD41 su Canale 4
  - platform: scd4x
    i2c_id: pca_multiplexer
    i2c_address: 0x62
    channel: 4
    update_interval: 10s
    
    co2:
      name: "CO₂ Cella (SCD41)"
      id: scd41_co2
    
    temperature:
      name: "T Sonda CO₂ (SCD41)"
      id: scd41_temp
    
    humidity:
      name: "RH Sonda CO₂ (SCD41)"
      id: scd41_rh
```

---

## 5. Pull-up Resistori: Configurazione

### Bus Principale (ESP32 → PCA9548A)

```
GPIO 4 (SDA) ──── R 4.7kΩ ──── +3.3V
GPIO 5 (SCL) ──── R 4.7kΩ ──── +3.3V

Pull-up singoli, montati vicino all'ESP32.
Lunghezza piste: < 10 cm per minimizzare crosstalk.
```

### Canale 3 (SHT45)

```
SC3 (SDA) ──── R 4.7kΩ ──── +3.3V
SD3 (SCL) ──── R 4.7kΩ ──── +3.3V

Montati vicino al modulo PCA9548A.
```

### Canale 4 (SCD41)

```
SC4 (SDA) ──── R 4.7kΩ ──── +3.3V
SD4 (SCL) ──── R 4.7kΩ ──── +3.3V

Montati vicino al modulo PCA9548A.
```

### Considerazioni Capacitive

```
Ogni canale ha capacità parassita dai cavi.
Con 4.7kΩ + capacità ~50pF:
  τ = R × C = 4.7kΩ × 50pF = 235ns
  f_cut = 1/(2πτ) ≈ 678 kHz ✅ (ben sopra 400 kHz I²C)

Nessun problema con pull-up 4.7kΩ standard.
```

---

## 6. Assegnazione Canali Definitiva

| Canale | Device | Indirizzo | Stato | Note |
|--------|--------|-----------|-------|-------|
| **0** | Libero | - | Disponibile | Espansione futura |
| **1** | Libero | - | Disponibile | Espansione futura |
| **2** | Libero | - | Disponibile | Espansione futura |
| **3** | SHT45 | 0x44 | ✅ Usato | T/RH primario |
| **4** | SCD41 | 0x62 | ✅ Usato | CO₂ primario |
| **5** | Libero | - | Disponibile | Secondo SHT45 (T gradiente) |
| **6** | Libero | - | Disponibile | Sensore pressione BMP280 |
| **7** | Libero | - | Disponibile | Espansione futura |

---

## 7. Troubleshooting I²C

| Sintomo | Causa | Soluzione |
|--------|-------|----------|
| SHT45 non trovato (scan I2C vuoto) | PCA9548A non commuta a canale 3 | Verificare A0/A1/A2 a GND, RST a +3.3V |
| SCD41 indirizzo sbagliato (0x44 invece di 0x62) | Canale sbagliato o sensore non connesso | Verificare canale 4, piste SCL/SDA |
| Clock stretching lento | Pull-up troppo alti | Ridurre a 3.3kΩ se necessario (raro) |
| Errore CRC SHT45 | Cavo schermato mancante o EMI | Aggiungere schermatura cavo, separare da +12V |
| Sensore occasionalmente scompare | Contatto intermittente | Verificare saldatura connettore JST |

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo
