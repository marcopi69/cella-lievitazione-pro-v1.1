# CELLA DI LIEVITAZIONE PRO — v1.1 Revisione Completa

**Data revisione:** Ottobre 2026  
**Versione precedente:** v1.0 Giugno 2026  
**Autore:** Progetto collaborativo ESP32 + ESPHome + Home Assistant

---

## 📋 Struttura della Documentazione

### **1. Documenti Fondamentali (Leggi Prima)**
- **`01-SPECIFICHE-TECNICHE.md`** — Specifica tecnica completa, lista componenti BOM, budget di potenza
- **`02-PINOUT-ESP32-DEFINITIVO.md`** — Tabella esatta dei GPIO con assegnazioni finali
- **`03-SCHEMA-BLOCCHI-ALIMENTAZIONE.md`** — Schema a blocchi PSU, protezioni, fusibili

### **2. Progettazione Circuiti Dettagliata**
- **`10-STADIO-POTENZA.md`** — Alimentatore ATX, fusibili, TVS, bulk capacitor
- **`11-DRIVER-MOSFET-CARICHI-DC.md`** — IRLZ44N per LED, ventola, umidificatore
- **`12-DRIVER-SSR-RISCALDATORE.md`** — BC547, SSR-25DA, filo scaldante 220 V
- **`13-BUS-I2C-SENSORI.md`** — PCA9548A, SHT45, SCD41, display touch
- **`14-SENSORI-TEMPERATURA-CO2.md`** — DS18B20 1-Wire, SCD41 NDIR
- **`15-VENTOLA-PWM-4PIN.md`** — Ventola 4 pin con controllo PWM + tachimetro

### **3. Interfaccia Utente**
- **`20-DISPLAY-TOUCH.md`** — Display touch singolo
- **`21-ENCODER-PULSANTI.md`** — Encoder 360°, pulsanti, logica input

### **4. Firmware e Software**
- **`30-ESPHOME-STRUTTURA.md`** — Configurazione ESPHome completa
- **`31-CONTROLLO-PID.md`** — Logica PID, interlock, protezioni software
- **`32-HOME-ASSISTANT-INTEGRATION.md`** — Integrazione HA, automazioni

### **5. KiCad e Realizzazione PCB**
- **`40-LAYOUT-PCB-REGOLE.md`** — Regole di layout, clearance, dissipazione
- **`41-LIBRERIE-FOOTPRINT-KICAD.md`** — Componenti KiCad, symbol, footprint

### **6. Installazione e Messa in Servizio**
- **`50-MONTAGGIO-CABLAGGIO.md`** — Sequenza montaggio, orientamenti
- **`51-CALIBRAZIONE-COMMISSIONING.md`** — Calibrazione sensori, test PID
- **`52-SICUREZZA-NORMATIVE.md`** — Normative vigenti, protezioni

---

## 🔴 PRINCIPALI CAMBIAMENTI v1.1 vs v1.0

| Elemento | v1.0 | v1.1 | Motivo |
|----------|------|------|--------|
| Compressore refrigerante | SSR + 220 V AC | ❌ Rimosso | Semplificazione; controllo thermico solo riscaldamento |
| Driver SSR | Due topologie diverse | ✅ Unified (3.4) | Correzione: BC547 open-collector confermata |
| LED stato | GPIO 35 ❌ | GPIO 16 ✅ | GPIO35 è input-only |
| Ventola | 2 pin + PWM low-side | 4 pin + PWM dedicato | Controllo pulito, tach integrato |
| Sensore CO₂ | MH-Z19C UART | SCD41 I²C ✅ | Stabilità, compensazione T/RH, I²C nativo |
| Display | 3× OLED + PCA9548A | 1 touch 3.5" ✅ | Interfaccia moderna, LVGL, user-friendly |
| GPIO usati | Tutti | Molti liberi | Semplificazione architettura |

---

## ⚡ SPECIFICHE SINTETICHE v1.1

| Parametro | Valore |
|-----------|--------|
| **MCU** | ESP32-DevKitC-E32 (WROOM-32E) oppure ESP32-S3 con touch integrato |
| **Controllo T** | PID riscaldamento via SSR + filo scaldante autoregolante 90 W |
| **Monitoraggio T** | SHT45 (precisione ±0.1°C), DS18B20×2 (backup) |
| **Controllo RH** | Umidificatore ultrasuoni 24 V (ON/OFF o PWM) |
| **Monitoraggio CO₂** | SCD41 NDIR 400–5000 ppm, ±40 ppm (I²C, con compensazione T/RH) |
| **Ricircolo aria** | Ventola 12 V 4-pin PWM + tach, 0–100% speed control |
| **Illuminazione** | Striscia LED 12 V RGB/RGBW dimmerizzata |
| **Interfaccia utente** | Display touch capacitivo 3.5" (ESP32-S3 oppure ILI9341+STM32) |
| **Alimentazione** | PSU ATX 260 W (+5 V, +12 V, +24 V) |
| **Protezioni** | Fusibili slow-blow, TVS, diodi fly-back, termostato meccanico |
| **Volume interno** | 60 × 60 × 55 cm (≈ 198 litri) |
| **Integrazione** | ESPHome + Home Assistant (Wi-Fi, offline-capable) |

---

## 🎯 Come Usare Questo Repository

### **Per chi vuole capire il progetto (quick start):**
1. Leggi **`01-SPECIFICHE-TECNICHE.md`**
2. Guarda le infografiche in **`IMG/`** (schema blocchi)
3. Consulta **`02-PINOUT-ESP32-DEFINITIVO.md`**

### **Per chi vuole realizzare il circuito:**
1. Scarica BOM da **`BOM/`** e ordina componenti
2. Scarica file KiCad da **`KICAD/`** e modificali
3. Leggi **`40-LAYOUT-PCB-REGOLE.md`** per il layout
4. Fabrica il PCB (oppure ordina prototipazione)

### **Per chi vuole capire i circuiti specifici:**
- Leggi i documenti **10–15** in sequenza
- Ogni documento è autonomo ma collegato agli altri
- Usa le infografiche SVG per visualizzazione rapida

### **Per il firmware:**
1. **`30-ESPHOME-STRUTTURA.md`** — panoramica
2. **`ESPHOME/`** — YAML pre-compilato da usare come template
3. **`31-CONTROLLO-PID.md`** — tuning PID specifico

### **Per montare e testare:**
1. **`50-MONTAGGIO-CABLAGGIO.md`** — sequenza passo-passo
2. **`51-CALIBRAZIONE-COMMISSIONING.md`** — first run e calibrazione
3. **`52-SICUREZZA-NORMATIVE.md`** — checklist sicurezza

---

## ✅ Checklist Pre-Realizzazione

- [ ] Leggi **`01-SPECIFICHE-TECNICHE.md`** interamente
- [ ] Scarica BOM da **`BOM/cella-pro-v1.1-BOM.csv`**
- [ ] Verifica disponibilità componenti (lead time, prezzi)
- [ ] Scarica librerie KiCad per componenti specifici
- [ ] Crea nuovo progetto KiCad con strutture da **`KICAD/`**
- [ ] Esamina schematic di riferimento
- [ ] Pianifica il layout PCB
- [ ] Ordina PCB o prepara breadboard per prototipo
- [ ] Raccogli strumenti: saldatore, multimetro, oscilloscopio (opzionale)
- [ ] Prepara stazione di saldatura e componenti

---

## 📊 Struttura File Repository

```
cella-lievitazione-pro-v1.1/
├── 00-LEGGIMI-PRIMO.md                    # Questo file
├── 01-SPECIFICHE-TECNICHE.md              # Specifiche complete
├── 02-PINOUT-ESP32-DEFINITIVO.md          # GPIO mapping
├── 03-SCHEMA-BLOCCHI-ALIMENTAZIONE.md     # PSU overview
├── 10-STADIO-POTENZA.md                   # Alimentazione
├── 11-DRIVER-MOSFET-CARICHI-DC.md         # LED, ventola, umidificatore
├── 12-DRIVER-SSR-RISCALDATORE.md          # Riscaldatore 220V
├── 13-BUS-I2C-SENSORI.md                  # I2C, PCA9548A, sensori
├── 14-SENSORI-TEMPERATURA-CO2.md          # DS18B20, SCD41
├── 15-VENTOLA-PWM-4PIN.md                 # Ventola con tach
├── 20-DISPLAY-TOUCH.md                    # Display touch
├── 21-ENCODER-PULSANTI.md                 # Input devices
├── 30-ESPHOME-STRUTTURA.md                # ESPHome config
├── 31-CONTROLLO-PID.md                    # PID logic
├── 32-HOME-ASSISTANT-INTEGRATION.md       # HA integration
├── 40-LAYOUT-PCB-REGOLE.md                # PCB design rules
├── 41-LIBRERIE-FOOTPRINT-KICAD.md         # KiCad libraries
├── 50-MONTAGGIO-CABLAGGIO.md              # Assembly
├── 51-CALIBRAZIONE-COMMISSIONING.md       # Calibration & test
├── 52-SICUREZZA-NORMATIVE.md              # Safety & compliance
├── IMG/                                    # Infografiche SVG/PNG
│   ├── 01-schema-blocchi-psu.svg
│   ├── 02-pinout-esp32.svg
│   ├── 03-driver-mosfet.svg
│   ├── 04-driver-ssr.svg
│   ├── 05-bus-i2c.svg
│   ├── 06-ventola-4pin.svg
│   └── ...
├── KICAD/                                  # File KiCad
│   ├── cella-pro-v1.1.kicad_pcb
│   ├── cella-pro-v1.1.kicad_sch
│   ├── cella-pro-v1.1.kicad_pro
│   └── footprints/
├── ESPHOME/                                # Firmware
│   ├── cella-pro-v1.1.yaml
│   ├── secrets.yaml.example
│   └── packages/
├── BOM/                                    # Lista componenti
│   ├── cella-pro-v1.1-BOM.csv
│   ├── cella-pro-v1.1-BOM.json
│   └── cella-pro-v1.1-BOM.xlsx
└── README.md                               # Overview generale
```

---

## 🔗 Link Utili

- [ESPHome Docs](https://esphome.io/)
- [ESP32 Datasheet](https://www.espressif.com/en/products/socs/esp32/resources)
- [SCD41 Datasheet](https://www.sensirion.com/en/environmental-sensors/gas-sensors/carbon-dioxide-sensors/scd-4x/)
- [Home Assistant](https://www.home-assistant.io/)
- [KiCad](https://kicad.org/)

---

**Buona realizzazione! 🚀**
