# CELLA DI LIEVITAZIONE PRO — v1.1 Revisione Completa

**Data revisione:** Ottobre 2026  
**Versione precedente:** v1.0 Giugno 2026  
**Responsabile progetto:** @marcopi69

---

## 📋 Struttura della Documentazione

### **1. Documenti Fondamentali**
- **[`01-SPECIFICHE-TECNICHE.md`](01-SPECIFICHE-TECNICHE.md)** — Specifica tecnica completa, lista componenti BOM, budget di potenza
- **[`02-PINOUT-ESP32-DEFINITIVO.md`](02-PINOUT-ESP32-DEFINITIVO.md)** — Tabella esatta dei GPIO con assegnazioni finali
- **[`03-SCHEMA-BLOCCHI-ALIMENTAZIONE.md`](03-SCHEMA-BLOCCHI-ALIMENTAZIONE.md)** — Schema a blocchi PSU, protezioni, fusibili

### **2. Progettazione Circuiti Dettagliata**
- **[`10-STADIO-POTENZA.md`](10-STADIO-POTENZA.md)** — Alimentatore ATX, fusibili, TVS, bulk capacitor
- **[`11-DRIVER-MOSFET-CARICHI-DC.md`](11-DRIVER-MOSFET-CARICHI-DC.md)** — IRLZ44N per LED, ventola, umidificatore
- **[`12-DRIVER-SSR-RISCALDATORE.md`](12-DRIVER-SSR-RISCALDATORE.md)** — BC547, SSR-25DA, filo scaldante 220 V
- **[`13-BUS-I2C-SENSORI.md`](13-BUS-I2C-SENSORI.md)** — PCA9548A, SHT45, SCD41, OLED display
- **[`14-SENSORI-TEMPERATURA-CO2.md`](14-SENSORI-TEMPERATURA-CO2.md)** — DS18B20 1-Wire, SCD41 NDIR
- **[`15-VENTOLA-PWM-4PIN.md`](15-VENTOLA-PWM-4PIN.md)** — Ventola 4 pin con controllo PWM + tachimetro

### **3. Interfaccia Utente**
- **[`20-DISPLAY-TOUCH-LVGL.md`](20-DISPLAY-TOUCH-LVGL.md)** — Display touch singolo, framework LVGL
- **[`21-ENCODER-PULSANTI.md`](21-ENCODER-PULSANTI.md)** — Encoder 360°, pulsanti, logica input

### **4. Firmware e Software**
- **[`30-ESPHOME-STRUTTURA.md`](30-ESPHOME-STRUTTURA.md)** — Configurazione ESPHome completa
- **[`31-CONTROLLO-PID.md`](31-CONTROLLO-PID.md)** — Logica PID, interlock, protezioni software
- **[`32-HOME-ASSISTANT-INTEGRATION.md`](32-HOME-ASSISTANT-INTEGRATION.md)** — Integrazione HA, automazioni, dashboard

### **5. KiCad e Realizzazione PCB**
- **[`40-LAYOUT-PCB-REGOLE.md`](40-LAYOUT-PCB-REGOLE.md)** — Regole di layout, clearance, dissipazione termica
- **[`41-LIBRERIE-FOOTPRINT-KICAD.md`](41-LIBRERIE-FOOTPRINT-KICAD.md)** — Componenti KiCad, symbol, footprint

### **6. Installazione e Messa in Servizio**
- **[`50-MONTAGGIO-CABLAGGIO.md`](50-MONTAGGIO-CABLAGGIO.md)** — Sequenza montaggio, orientamenti, isolamenti
- **[`51-CALIBRAZIONE-COMMISSIONING.md`](51-CALIBRAZIONE-COMMISSIONING.md)** — Calibrazione sensori, test PID, first run
- **[`52-SICUREZZA-NORMATIVE.md`](52-SICUREZZA-NORMATIVE.md)** — Normative vigenti, protezioni, conformità

### **7. Allegati e Infografiche**
- **[`IMG/`](IMG/)** — Diagrammi, schemi blocchi, infografiche (formato SVG/PNG)
- **[`KICAD/`](KICAD/)** — Project file, schematic, PCB layout
- **[`ESPHOME/`](ESPHOME/)** — Configurazione YAML pre-compilata
- **[`BOM/`](BOM/)** — Lista componenti esportabili (CSV, JSON)

---

## 🔴 CAMBIAMENTI RISPETTO A v1.0

| Punto | v1.0 | v1.1 | Motivo |
|-------|------|------|--------|
| **Compressore refrigerante** | SSR + 220 V AC | ❌ Rimosso | Semplificazione; controllo termico limitato al riscaldamento |
| **Driver SSR riscaldatore** | Due topologie diverse | ✅ Unified sez. 3.4 | Correzione: topologia BC547 open-collector confermata |
| **LED di stato** | GPIO 35 (❌ input-only) | GPIO 16 ✅ | GPIO35 non ha uscita, usato 16 con decoupling |
| **Ventola ricircolo** | 2 pin + PWM low-side | 4 pin + PWM dedicato | Controllo pulito senza armoniche, tach integrato |
| **Sensore CO₂** | MH-Z19C UART ± 50 ppm | SCD41 I²C ± 40 ppm | Migliore stabilità, compensazione T/RH, I²C |
| **Display interfaccia** | 3× OLED 1.3" + PCA9548A | 1 display touch 3.5" | Interfaccia moderna, gestione touch, LVGL |
| **Bus I²C** | Muxato su 8 canali | Semplificato | Solo SHT45, SCD41, display; PCA9548A ridimensionato |
| **GPIO liberi** | Pochi | Molti | Liberati dalla rimozione di compressore e 3 OLED |
| **Pinout** | Tabella in sez. 4 | ✅ Documento dedicato | Tracciabilità e facilità di consultazione |

---

## ⚡ SPECIFICHE SINTETICHE v1.1

| Parametro | Valore |
|-----------|--------|
| **MCU** | ESP32-DevKitC-E32 (WROOM-32E) |
| **Controllo T** | PID riscaldamento via SSR + filo scaldante 90 W |
| **Monitoraggio T** | SHT45 (precisione), DS18B20×2 (backup) |
| **Controllo RH** | Umidificatore ultrasuoni 24 V (ON/OFF o PWM) |
| **Monitoraggio CO₂** | SCD41 NDIR 400–5000 ppm (I²C) |
| **Ricircolo aria** | Ventola 12 V 4-pin PWM + tach + diodo fly-back |
| **Illuminazione** | Striscia LED 12 V WS2812b o RGBW dimmerizzata |
| **Interfaccia utente** | Display touch 3.5" (ESP32-S3 oppure STM32 + ILI9341) |
| **Alimentazione** | PSU ATX 260 W (+5 V, +12 V, +24 V) |
| **Protezioni** | Fusibili slow-blow, TVS, diodi fly-back, termostato meccanico |
| **Volume interno** | 60 × 60 × 55 cm (≈ 198 litri) |
| **Integrazione** | ESPHome + Home Assistant via Wi-Fi (opzionale offline) |

---

## 🎯 Come Usare Questo Repository

### **Per iniziare rapidamente:**
1. Leggi **[`01-SPECIFICHE-TECNICHE.md`](01-SPECIFICHE-TECNICHE.md)** per capire il progetto.
2. Consulta **[`02-PINOUT-ESP32-DEFINITIVO.md`](02-PINOUT-ESP32-DEFINITIVO.md)** per i GPIO.
3. Apri i file KiCad in **[`KICAD/`](KICAD/)** per il layout.

### **Per capire i circuiti:**
- Leggi i documenti **10–15** in sequenza: cada uno è autosufficiente ma collegato.
- Usa le infografiche in **[`IMG/`](IMG/)** per visualizzazione rapida.

### **Per il firmware:**
- Vedi **[`30-ESPHOME-STRUTTURA.md`](30-ESPHOME-STRUTTURA.md)**.
- Usa il YAML in **[`ESPHOME/`](ESPHOME/)** come template.

### **Per montare il circuito:**
1. **[`40-LAYOUT-PCB-REGOLE.md`](40-LAYOUT-PCB-REGOLE.md)** — regole KiCad.
2. **[`50-MONTAGGIO-CABLAGGIO.md`](50-MONTAGGIO-CABLAGGIO.md)** — montaggio fisico.
3. **[`51-CALIBRAZIONE-COMMISSIONING.md`](51-CALIBRAZIONE-COMMISSIONING.md)** — test e avvio.

---

## ✅ Checklist Realizzazione

- [ ] Stampa/scarica BOM da **[`BOM/`](BOM/)**
- [ ] Ordina componenti verificando datasheet
- [ ] Scarica librerie KiCad per i componenti specifici
- [ ] Rivedi layout PCB in **[`KICAD/`](KICAD/)**
- [ ] Prepara stazione di saldatura
- [ ] Monta PCB seguendo **[`50-MONTAGGIO-CABLAGGIO.md`](50-MONTAGGIO-CABLAGGIO.md)**
- [ ] Carica firmware ESPHome da **[`ESPHOME/`](ESPHOME/)**
- [ ] Esegui calibrazione sensori **[`51-CALIBRAZIONE-COMMISSIONING.md`](51-CALIBRAZIONE-COMMISSIONING.md)**
- [ ] Test funzionamento con lievitazione di prova
- [ ] Integra in Home Assistant e personalizza dashboard

---

## 📞 Support e Contributi

Se trovi errori, imprecisioni o hai migliorie, puoi:
- Aprire un **Issue** nel repository
- Proporre un **Pull Request** con correzioni
- Contattare l'autore tramite GitHub

---

## 📄 Licenza

Questo progetto è rilasciato sotto **MIT License**. Sei libero di usarlo, modificarlo e ridistribuirlo secondo i termini della licenza.

---

**Buona realizzazione! 🚀**
