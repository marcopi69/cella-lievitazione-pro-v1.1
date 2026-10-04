# CELLA DI LIEVITAZIONE PRO v1.1 — SPECIFICHE TECNICHE COMPLETE

## 1. Panoramica del Progetto

La cella di lievitazione PRO è un sistema completamente autonomo per il controllo della fermentazione di impasti da forno, con volume interno di **60×60×55 cm (≈ 198 litri)**. Il sistema offre:

- ✅ Controllo **preciso della temperatura** (±0.5°C) tramite riscaldamento resistivo e monitoraggio
- ✅ **Monitoraggio dell'umidità relativa** tramite umidificatore a ultrasuoni 24 V
- ✅ **Monitoraggio CO₂** in tempo reale (sensore SCD41, range 400–5000 ppm, ±40 ppm)
- ✅ **Ricircolo dell'aria** tramite ventola 12 V PWM 4-pin con tachimetro
- ✅ **Illuminazione interna** tramite striscia LED RGB/RGBW 12 V dimmerizzata
- ✅ **Interfaccia touch** singola, moderna e intuitiva
- ✅ **Integrazione Home Assistant** via Wi-Fi (o completamente offline)

### 1.1 Obiettivi di Progetto

| Obiettivo | Come | Risultato |
|-----------|------|----------|
| Controllo T preciso | PID riscaldamento + sensori SHT45 + DS18B20 | ±0.5°C sul setpoint |
| Controllo RH | Umidificatore ON/OFF o PWM controllato da RH | Mantenimento 60–95% RH |
| Monitoraggio CO₂ | SCD41 I²C, reading ogni 1 s | Precisione ±40 ppm |
| Ricircolo aria | Ventola PWM 0–100% regolabile | Omogeneità T/RH in cella |
| Lievitazione ottimale | Dashboard Home Assistant con curve T/RH/CO₂ | Gestione professionale del processo |
| Sicurezza | Protezioni hardware + software interlock | Nessun rischio incendio / sovraccarico |

---

## 2. Lista Componenti (BOM) — v1.1

### 2.1 MCU e Comunicazione

| Componente | Qt. | Part Number / Specifica | Funzione | Note |
|-----------|-----|------|---------|-------|
| **ESP32-DevKitC-E32** | 1 | ESP32-WROOM-32E, 38 pin | MCU principale | ⚠ Oppure ESP32-S3 con touch integrato |
| **Oscillatore esterno** | - | 40 MHz integrato | Clock MCU | - |

### 2.2 Sensori T/RH e CO₂

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **SHT45** | 1 | Sensore T/RH I²C, ±0.1°C / ±1.5% RH | Monitoraggio preciso T/RH | Canale 3 PCA9548A |
| **DS18B20** | 2 | Sensore 1-Wire impermeabile | Backup temperatura, sonde zona | GPIO 13 bus singolo |
| **SCD41** | 1 | Sensore NDIR 400–5000 ppm, ±40 ppm | Monitoraggio CO₂ fermentazione | Canale 4 PCA9548A, I²C, compensazione T/RH |

### 2.3 Bus I²C e Multiplexing

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **PCA9548A** | 1 | I²C 8-ch multiplexer, addr. 0x70 | Multiplexing I²C per sensori | Canale 3 = SHT45, Canale 4 = SCD41 |
| **Resistori pull-up I²C** | - | 4.7 kΩ 1/4 W | Bus principale ESP32 → PCA9548A | Singola coppia vicino a ESP32 |

### 2.4 Display Interfaccia Utente

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **Display touch capacitivo** | 1 | 3.5" ILI9341 (SPI) oppure ESP32-S3 integrato | Interfaccia principale | Opzione 1: ESP32 + ILI9341 / Opzione 2: ESP32-S3 All-in-One |
| **Resistori pull-up touch** | - | 100 kΩ (se SPI) | Stabilizzazione linee dati | - |

### 2.5 Encoder e Pulsanti

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **Encoder rotativo 360°** | 1 | 20 posizioni/giro, 5 pin | Menu navigazione, cambio setpoint | GPIO 18, 19, 21 |
| **Pulsante reset** | 1 | Normalmente aperto NO | Reset hardware ESP32 | Collegato EN, con condensatore 100 nF |
| **Pulsante multifunzione** | 1 | Normalmente aperto NO | LED ON/OFF (breve), Umidif. ON/OFF (lungo) | GPIO 22 |

### 2.6 Driver MOSFET — Carichi DC 12 V e 24 V

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **IRLZ44N** | 3 | MOSFET N-ch logic-level, Rds_on < 22 mΩ | Striscia LED / Ventola / Umidificatore | Rth_ja ≈ 62°C/W, τ_turn < 100 ns |
| **R_gate** | 3 | 220 Ω 1/4 W | Resistenza serie gate | Protezione da picchi |
| **R_pull_gate** | 3 | 10 kΩ 1/4 W | Pull-down gate MOSFET | Impedisce stato flottante al boot |
| **D (fly-back ventola)** | 1 | 1N5819 Schottky | Protezione BEMF ventola | Anodo → Drain, Catodo → +12 V |
| **D (fly-back umidif.)** | 1 | 1N5819 Schottky | Protezione BEMF umidificatore | Anodo → Drain, Catodo → +24 V |

### 2.7 Driver SSR — Riscaldatore 220 V

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **BC547** | 1 | NPN BJT, hFE > 100 | Driver SSR, open-collector | GPIO 32 (HEATER_SSR) |
| **R_base BC547** | 1 | 10 kΩ 1/4 W | Limitazione corrente base | I_b ≈ 1.3 mA @ 3.3 V |
| **R_col BC547** | 1 | 100 kΩ 1/4 W | Pull-down collettore (sicurezza) | Mantiene SSR– a GND se ESP32 offline |
| **SSR-25DA** | 1 | Solid State Relay, 25 A, 3÷32 V ctrl | Pilotaggio filo scaldante 220 V | Dissipatore ≥ 1 cm² |
| **Filo scaldante** | 1 | Autoregolante 18 W/m × 5 m = 90 W @ 220 V | Elemento riscaldante | Montaggio a spirale uniforme |

### 2.8 Ventola Ricircolo 12 V — 4 Pin PWM

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **Ventola DC 12 V** | 1 | 4-pin PWM (PWM, +12V, GND, TACH), ≤ 0.5 A | Ricircolo aria | GPIO 25 PWM, GPIO 34 TACH |
| **Partitore TACH** | - | 10 kΩ + 4.7 kΩ | Adattamento livello 12 V → 3.3 V | Opzionale se ventola output a 3.3 V |

### 2.9 Umidificatore Ultrasuoni 24 V

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **Modulo umidificatore** | 1 | Transducer ultrasuoni 24 V, ≤ 1.5 A | Produzione vapore | GPIO 26 (ON/OFF o PWM) |
| **Serbatoio acqua** | 1 | Capacità ≥ 500 ml | Riserva acqua | Cambio giornaliero consigliato |

### 2.10 Illuminazione Interna 12 V

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **Striscia LED** | 1 | WS2812B (RGB) oppure RGBW, 12 V, ≤ 1.5 A | Illuminazione cella | GPIO 33 PWM, oppure DIN per WS2812B |

### 2.11 Alimentazione

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **Alimentatore ATX** | 1 | 260 W, +5 V / +12 V / ±12 V / +24 V | Alimentazione sistema | Cortocircuitare PS_ON (pin 16, verde) con GND (pin 15, nero) oppure usare resistenza 1 kΩ |
| **Connettore J2 (PSU)** | 1 | Morsettiera KF350 5.08 mm 3 pin | Ingresso +5V, +12V, GND | Corrente max 10 A per pin |

### 2.12 Protezioni Alimentazione

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **TVS 1.5KE24A** | 1 | Diodo TVS unipolare, clamp 24 V | Protezione picchi +12 V | Catodo → +12 V, Anodo → GND |
| **Fusibile F1** | 1 | Slow Blow 2 A, 250 V, 5×20 mm | Protezione linea +12 V | In serie dopo TVS |
| **Fusibile F2** | 1 | Slow Blow 2 A, 250 V, 5×20 mm | Protezione linea +5 V | In serie dopo regolatore PSU |
| **Portafusibili** | 2 | Certificati 250 V, 5×20 mm | Supporto fusibili | Panel-mount oppure PCB-mount |

### 2.13 Condensatori — Filtro e Bulk

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **C1** | 1 | 100 µF, 25 V, elettrolitico | Bulk capacitor +5 V | Vicino pin VIN ESP32 |
| **C2** | 1 | 100 nF, ceramico | Decoupling +5 V | Vicino ogni IC alimentato a 5 V |
| **C3** | 2 | 470 µF, 25 V, elettrolitico | Bulk capacitor +12 V (ventola) | Dopo TVS, prima di Q1 |
| **C4** | 2 | 100 nF, ceramico | Decoupling +12 V | Vicino MOSFET gate |
| **C5** | 1 | 470 µF, 35 V, elettrolitico | Bulk capacitor +24 V (umidif.) | Linea dedicata |
| **C6** | 1 | 100 nF, ceramico | Decoupling +24 V | Vicino connettore umidificatore |
| **C_boot** | 1 | 100 nF, ceramico | Decoupling pulsante RESET | Tra EN e GND |

### 2.14 Varie

| Componente | Qt. | Specifica | Funzione | Note |
|-----------|-----|---------|---------|-------|
| **LED di stato** | 1 | LED 3 mm rosso oppure bicolore | Indicazione stato sistema | GPIO 16 |
| **R_LED** | 1 | 1 kΩ 1/4 W | Resistenza serie LED | I_led ≈ 1.3 mA @ 3.3 V |
| **Connettore J_OLED** | 1 | JST 4-pin oppure DuPont | Interfaccia display touch (SPI/I²C) | Adatto al tipo di display scelto |
| **Connettori J_DS18B20** | 2 | JST 3-pin | Interfaccia sonde temperatura | Cavo schermato ≤ 5 m |
| **Connettore J_MH-SCD41** | 1 | JST 4-pin | Interfaccia sensore CO₂ (I²C) | Cavo non schermato, ≤ 1 m |
| **Connettore J_Encoder** | 1 | JST 5-pin o DuPont | Interfaccia encoder rotativo | GND, +3.3V, CLK, DT, SW |
| **Connettore J_Ventola** | 1 | KF350 2-pin oppure Molex | Alimentazione ventola 12 V | Corrente 0.5 A |
| **Connettore J_LED_Strip** | 1 | Molex 2-pin oppure saldato | Alimentazione striscia LED 12 V | Corrente ≤ 1.5 A |
| **Connettore J_SSR** | 1 | Morsettiera 2-pin KF350 | Controllo SSR (SSR+/SSR–) | Corrente controllo ≤ 20 mA |
| **Cavi jumper** | - | 22 AWG, vari colori | Interconnessioni PCB | Per breadboarding oppure prototipo |

---

## 3. Budget di Corrente e Potenza — Versione Finale

### 3.1 Linea +5 V

| Carico | Corrente tipica | Corrente picco | Potenza | Note |
|--------|-----------------|-----------------|---------|-------|
| ESP32 (Wi-Fi attivo) | 240 mA | 500 mA | 1.2 W | Conteggiare sempre il picco |
| SHT45 | 1 mA | 5 mA | 5 mW | I²C pull-up |
| SCD41 | 5 mA | 10 mA | 25 mW | I²C pull-up, lettura |
| Display touch (SPI) | 100 mA | 150 mA | 0.5 W | Refresh display |
| LED di stato | 2 mA | 2 mA | 7 mW | Sempre acceso |
| Encoder + pulsanti | 1 mA | 1 mA | 3 mW | Pull-up interni |
| **TOTALE +5 V** | **~350 mA** | **~680 mA** | **~3.4 W** | **Fusibile F2: 2 A ✅** |

⚠️ **Nota:** La linea +5 V è da PSU ATX. Il regolatore interno ESP32 (VIN → 3.3V) consuma i 240 mA per Wi-Fi. Capacitor +5V da 100 µF maniene stabile la tensione.

### 3.2 Linea +12 V

| Carico | Corrente tipica | Corrente picco | Potenza | Note |
|--------|-----------------|-----------------|---------|-------|
| Striscia LED (max) | 1.0 A | 1.5 A | 18 W | Solo se luci al massimo |
| Ventola ricircolo | 0.3 A | 0.5 A | 6 W | Dipende da RPM |
| **TOTALE +12 V DC** | **~1.3 A** | **~2.0 A** | **~24 W** | **Fusibile F1: 2 A ✅** |

### 3.3 Linea +24 V

| Carico | Corrente tipica | Corrente picco | Potenza | Note |
|--------|-----------------|-----------------|---------|-------|
| Umidificatore ultrasuoni | 0.8 A | 1.5 A | 20 W | Intermittente, PWM opzionale |
| **TOTALE +24 V** | **~0.8 A** | **~1.5 A** | **~20 W** | **Fusibile 1 A ✅** |

### 3.4 Linea 220 V AC

| Carico | Corrente nominale | Corrente spunto | Potenza | Note |
|--------|-------------------|-----------------|---------|-------|
| Filo scaldante 90 W | 0.41 A | Nessuno (resistivo) | 90 W | Fusibile F_HEAT: 1 A Slow Blow ✅ |

### 3.5 Riepilogo Totale

| Linea | Corrente totale | Potenza totale | Fusibile |
|-------|-----------------|-----------------|----------|
| +5 V | ~680 mA (picco) | ~3.4 W | 2 A SB |
| +12 V | ~2.0 A (picco) | ~24 W | 2 A SB |
| +24 V | ~1.5 A (picco) | ~20 W | 1 A SB |
| **220 V AC** | **0.41 A** | **90 W** | **1 A SB** |
| **CONSUMO TOTALE** | **~4.6 A @ 12 V equiv.** | **~140 W** | **PSU ATX 260 W ✅** |

✅ **Il PSU ATX da 260 W eroga tipicamente 15–20 A sulla linea +12 V combinata. Siamo abbondantemente nei limiti.**

---

## 4. Architettura del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                    CELLA DI LIEVITAZIONE PRO                 │
│                     60×60×55 cm (198 L)                      │
└─────────────────────────────────────────────────────────────┘
                              │
                 ┌────────────┼────────────┐
                 │            │            │
        ┌────────▼─────┐  ┌───▼──────┐  ┌─▼────────┐
        │  RISCALDAMENTO│ │RICIRCOLO │  │UMIDIFIC. │
        │  FILO SCALDAN │ │VENTOLA   │  │ULTRASUON │
        │  90 W 220 V AC│ │12 V PWM  │  │24 V      │
        └────────┬─────┘  └───┬──────┘  └─┬────────┘
                 │            │          │
        ┌────────▼─────────────▼──────────▼──────┐
        │    MODULO CONTROLLO (PCB PRINCIPALE)   │
        │                                        │
        │  ESP32-DevKitC-E32 (WROOM-32E)        │
        │  • GPIO 32: SSR riscaldatore (BC547)  │
        │  • GPIO 25: PWM ventola (IRLZ44N)     │
        │  • GPIO 26: ON/OFF umidificatore      │
        │  • GPIO 33: PWM LED (IRLZ44N)         │
        │  • GPIO 16: LED di stato              │
        │                                        │
        │  I²C BUS (GPIO 4 SDA, GPIO 5 SCL)    │
        │  ├─ PCA9548A (0x70)                  │
        │  │  ├─ Ch 3: SHT45 (0x44)             │
        │  │  └─ Ch 4: SCD41 (0x44)             │
        │  ├─ Display touch (SPI oppure I²C)    │
        │                                        │
        │  1-Wire BUS (GPIO 13)                 │
        │  ├─ DS18B20 #1 (sonda zona)           │
        │  └─ DS18B20 #2 (backup)               │
        │                                        │
        │  GPIO 25: Ventola PWM + tach (GPIO34) │
        │  GPIO 18/19/21: Encoder + pulsanti    │
        │  GPIO 22: Pulsante multifunzione      │
        │                                        │
        │  Alimentazione: PSU ATX 260 W         │
        │  (+5V, +12V, +24V)                    │
        └────────┬─────────────────────────────┘
                 │
        ┌────────▼────────┐
        │   SENSORI       │
        │ • SHT45 (T/RH)  │
        │ • SCD41 (CO₂)   │
        │ • DS18B20×2 (T) │
        │ • Ventola TACH  │
        └─────────────────┘
                 │
        ┌────────▼────────┐
        │  HOME ASSISTANT  │
        │  (Wi-Fi, opt.)   │
        └──────────────────┘
```

---

## 5. Protezioni e Sicurezza

### 5.1 Protezioni Hardware

| Protezione | Componente | Funzione |
|-----------|-----------|----------|
| Sovratensione +12 V | TVS 1.5KE24A | Clamp a 24 V, assorbe picchi transitori |
| Sovracorrente +12 V | Fusibile F1 2A SB | Interrompe se corrente > 2 A per > 5 s |
| Sovracorrente +5 V | Fusibile F2 2A SB | Interrompe se corrente > 2 A per > 5 s |
| Sovracorrente 220 V | Fusibile F_HEAT 1A SB | Protezione linea riscaldatore |
| BEMF ventola | Diodo 1N5819 fly-back | Assorbe spike induttivo al turnoff MOSFET |
| BEMF umidif. | Diodo 1N5819 fly-back | Assorbe spike induttivo al turnoff MOSFET |
| Sottoalimentazione | Condensatori bulk 100µF+470µF | Stabilizzazione tensione, noise filtering |
| Riscaldatore runaway | ⚠️ **TERMOSTATO MECCANICO** | **Interviene se T > 45°C** |
| SSR guasto | Interlock software | Se SSR si incolla, cortocircuito filo → fusibile interviene |
| Corto circuito GPIO | Resistenza serie gate 220 Ω | Limita corrente picco |

### 5.2 Protezioni Software

| Protezione | Come | Funzione |
|-----------|------|----------|
| Interlock T/RH | Lambda ESPHome | Riscaldatore e umidificatore non simultanei |
| Protezione T max | Sensor + automation | Se T > 45°C → riscaldatore OFF + allarme |
| Protezione T min | Sensor + automation | Se T < 15°C → allarme (cella non riscaldata) |
| Protezione RH max | Sensor + automation | Se RH > 95% → umidificatore OFF forzato |
| Protezione CO₂ | Sensor + automation | Se CO₂ > 4500 ppm → ventola ON al 100% |
| Watchdog ESP32 | Built-in | Reset automatico se sistema non risponde |
| Monitoraggio Wi-Fi | ESPHome fallback AP | Se router non raggiungibile, accetti connessioni AP per debug |
| Log errori | Syslog | Registrazione anomalie su Home Assistant |

---

## 6. Prossimi Passi

1. ✅ Leggi **`02-PINOUT-ESP32-DEFINITIVO.md`** per il mapping GPIO finale
2. ✅ Leggi **`03-SCHEMA-BLOCCHI-ALIMENTAZIONE.md`** per capire il flusso di potenza
3. ✅ Consulta i documenti **10–15** per i circuiti specifici
4. ✅ Scarica il file BOM **`BOM/cella-pro-v1.1-BOM.csv`**
5. ✅ Crea progetto KiCad usando **`KICAD/`** come riferimento
6. ✅ Progetta il layout PCB seguendo **`40-LAYOUT-PCB-REGOLE.md`**
7. ✅ Ordina componenti e fabrica PCB
8. ✅ Monta seguendo **`50-MONTAGGIO-CABLAGGIO.md`**
9. ✅ Calibra sensori con **`51-CALIBRAZIONE-COMMISSIONING.md`**
10. ✅ Integra in Home Assistant

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Revisione completa, pronto per realizzazione
