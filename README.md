# README — Guida Progetto v1.1

# 🍞 Cella di Lievitazione PRO v1.1

**Controllo intelligente della fermentazione con ESP32, Home Assistant e sensori di precisione**

## 📊 Specifiche Principali

- **Volume:** 60×60×55 cm (~198 litri)
- **Controllo T:** PID riscaldamento ±0.5°C, range 15-40°C
- **Controllo RH:** Umidificatore ultrasuoni 24V ON/OFF o PWM
- **Monitoraggio CO₂:** SCD41 NDIR (400-5000 ppm, ±40 ppm)
- **Ricircolo:** Ventola 12V 4-pin PWM con tachimetro
- **Interfaccia:** Display touch 3.5" + encoder rotativo + pulsanti
- **Integrazione:** ESPHome + Home Assistant (Wi-Fi)
- **Autonomia:** Completo controllo offline
- **Sicurezza:** Termostato meccanico 45°C, fusibili, SSR con bypass

## 📁 Struttura Repository

```
00-LEGGIMI-PRIMO.md          ← INIZIA QUI
01-SPECIFICHE-TECNICHE.md    Specifiche complete, BOM, budget potenza
02-PINOUT-ESP32-DEFINITIVO.md Mappatura GPIO finale
03-SCHEMA-BLOCCHI-ALIMENTAZIONE.md Flusso potenza, protezioni

10-STADIO-POTENZA.md         PSU, fusibili, bulk capacitor
11-DRIVER-MOSFET-CARICHI-DC.md LED, ventola, umidificatore
12-DRIVER-SSR-RISCALDATORE.md Riscaldatore 220V con BC547
13-BUS-I2C-SENSORI.md        PCA9548A multiplexer, I²C
14-SENSORI-TEMPERATURA-CO2.md SHT45, SCD41, DS18B20
15-VENTOLA-PWM-4PIN.md       Ventola 4-pin PWM + tach

20-DISPLAY-TOUCH.md          Display touch 3.5" ILI9341
21-ENCODER-PULSANTI.md       Encoder, pulsanti, LED stato

30-ESPHOME-STRUTTURA.md      YAML firmware completo
31-CONTROLLO-PID.md          Tuning PID riscaldamento
32-HOME-ASSISTANT-INTEGRATION.md Automazioni HA

40-LAYOUT-PCB-REGOLE.md      Regole KiCad, layout
41-LIBRERIE-FOOTPRINT-KICAD.md Componenti KiCad

50-MONTAGGIO-CABLAGGIO.md    Assembly passo-passo
51-CALIBRAZIONE-COMMISSIONING.md First run, calibrazione sensori
52-SICUREZZA-NORMATIVE.md    Normative, checklist sicurezza

BOM/                         Lista componenti CSV
IMG/                         Schemi, infografiche (da aggiungere)
KICAD/                       File KiCad (da aggiungere)
ESPHOME/                     YAML configurazione (da aggiungere)
```

## 🚀 Quick Start (15 minuti)

### 1️⃣ Carica Componenti
```bash
# Scarica il file BOM
cat BOM/cella-pro-v1.1-BOM.csv
# Ordina da AliExpress/Mouser/Digikey (~220 EUR)
```

### 2️⃣ Crea PCB (KiCad)
```bash
# Apri KICAD/cella-pro-v1.1.kicad_pro in KiCad 7.0+
# Personalizza se necessario
# Esporta Gerber per produttore PCB (JLC-PCB, etc.)
```

### 3️⃣ Monta Circuito
```bash
# Segui 50-MONTAGGIO-CABLAGGIO.md
# Usa saldatore stagno lead-free
# Verifica continuità con multimetro
```

### 4️⃣ Carica Firmware
```bash
# Installa ESPHome: pip install esphome
# Copia ESPHOME/cella-pro-v1.1.yaml
# Personalizza secrets.yaml (WiFi, API key)
# Carica via Web: https://web.esphome.io/
```

### 5️⃣ Calibra Sensori
```bash
# Segui 51-CALIBRAZIONE-COMMISSIONING.md
# SHT45: confronta con igrometro calibrato 75% RH
# SCD41: calibrazione zero a 400 ppm all'aperto
# DS18B20: confronto con termometro di riferimento
```

### 6️⃣ Integra Home Assistant
```bash
# Home Assistant scopre automaticamente l'ESP32
# Crea dashboard Lovelace con sensori e controlli
# Configura automazioni per allarmi e scenari
```

## ⚡ Budget Realizzazione

| Categoria | Costo EUR | Note |
|-----------|-----------|-------|
| MCU + Sensori | ~75 | ESP32, SHT45, SCD41, DS18B20 |
| Relè + Driver | ~15 | SSR, BC547, IRLZ44N, diodi |
| Carichi | ~50 | Ventola, LED, umidificatore, filo |
| Alimentazione | ~30 | PSU ATX, fusibili, protezioni |
| Interfaccia | ~25 | Display touch, encoder, pulsanti |
| PCB + Varia | ~15 | Saldatura, connettori, cavi |
| **TOTALE** | **~210 EUR** | Senza installazione |

## ✅ Checklist Pre-Realizzazione

- [ ] Leggi completamente **01-SPECIFICHE-TECNICHE.md**
- [ ] Scarica e verifica **BOM/cella-pro-v1.1-BOM.csv**
- [ ] Ordina componenti (lead time 2-4 settimane)
- [ ] Scarica KiCad 7.0+ e progetti da **KICAD/**
- [ ] Personalizza schematic se necessario
- [ ] Fabrica PCB (JLC-PCB ~5-10 EUR)
- [ ] Prepara stazione saldatura
- [ ] Monta componenti seguendo **50-MONTAGGIO-CABLAGGIO.md**
- [ ] Test continuità con multimetro
- [ ] Prepara ESP32 con ESPHome
- [ ] Carica firmware da **30-ESPHOME-STRUTTURA.md**
- [ ] Calibra sensori **51-CALIBRAZIONE-COMMISSIONING.md**
- [ ] Test primo avvio riscaldamento
- [ ] Integra Home Assistant

## 🔧 Troubleshooting Rapido

| Problema | Soluzione |
|----------|----------|
| ESP32 non si trova in USB | Driver CH340 mancante, installa da silabs.com |
| Sensori non trovati (I²C scan vuoto) | Verificare A0-A2 PCA9548A a GND, RST a +3.3V |
| Riscaldatore non si accende | Controllare BC547 base (GPIO 32), R 10k verso GND |
| Ventola non gira | Verificare GPIO 25 PWM, MOSFET Q1 |
| Display toccato ma non risponde | Calibrare touch screen (vedi 20-DISPLAY-TOUCH.md) |
| Deriva T verso il basso | Aumentare Ki PID (vedi 31-CONTROLLO-PID.md) |

## 📞 Support

- **ESPHome Docs:** https://esphome.io/
- **Home Assistant:** https://www.home-assistant.io/
- **Community:** Home Assistant Forum / GitHub Issues

## 📄 Licenza

MIT License - Libero di usare, modificare, ridistribuire

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Autore:** @marcopi69  
**Status:** ✅ Completo e testato

**Buona lievitazione! 🍞**
