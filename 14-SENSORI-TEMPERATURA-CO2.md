# BOM — LISTA COMPONENTI COMPLETA v1.1

## Formato CSV (importabile in JLCPCB, Mouser, Digikey)

```csv
Quantity,Part Number,Designator,Value,Footprint,Description,Link/Supplier,Unit Cost EUR,Total EUR
1,ESP32-DEVKITC-E32,U1,,DIP-38,ESP32-DevKitC-E (WROOM-32E),Espressif/AliExpress,12.00,12.00
1,SHT45,U2,SHT45-I2C,,Sensore T/RH I2C precisione,Sensirion/Mouser,10.50,10.50
1,SCD41,U3,SCD41-I2C,,Sensore CO2 NDIR I2C,Sensirion/Mouser,35.00,35.00
2,DS18B20,U4 U5,DS18B20-DIP8,,Sensore T 1-Wire impermeabile,Maxim/AliExpress,2.50,5.00
1,PCA9548A,U6,PCA9548A-TSSOP16,,I2C Multiplexer 8 canali,TI/Mouser,3.50,3.50
3,IRLZ44N,Q1 Q2 Q3,IRLZ44N-TO220,,MOSFET N-ch logic-level 55V,IR/Mouser,1.20,3.60
1,BC547,Q5,BC547-TO92,,Transistor NPN BJT,Various/AliExpress,0.15,0.15
1,SSR-25DA,K1,SSR-25DA,,Solid State Relay AC 220V 25A,Various/AliExpress,8.50,8.50
1,Encoder Rotativo 360,SW1,ENCODER-5PIN,,Encoder meccanico 5 pin,AliExpress,2.00,2.00
1,Pulsante Multifunzione,SW2,PUSHBUTTON-2PIN,,Pulsante NO 6x6mm,AliExpress,0.30,0.30
1,Pulsante Reset,SW3,PUSHBUTTON-2PIN,,Pulsante NO 6x6mm,AliExpress,0.30,0.30
1,LED di Stato,LED1,LED-3MM-RED,,LED 3mm rosso,AliExpress,0.10,0.10
1,Display Touch,U7,ILI9341-SPI,,Display touch 3.5 inch 320x480 SPI,AliExpress,15.00,15.00
1,Ventola 4-Pin 12V,FAN1,NOCTUA-60MM-PWM,,Ventola ricircolo 12V 4-pin PWM,Noctua/AliExpress,20.00,20.00
1,Modulo Umidificatore,HUMID1,ULTRASONIC-24V,,Umidificatore ultrasuoni 24V,AliExpress,12.00,12.00
1,Striscia LED 12V,LED_STRIP,WS2812B-60LED,,Striscia LED RGB/RGBW 12V 60 LED,AliExpress,8.00,8.00
1,PSU ATX 260W,PSU1,ATX-260W,,Alimentatore ATX 260W +5V +12V +24V,Behuang/AliExpress,18.00,18.00
1,Fusibile 2A SB 250V,F1,5x20-2A-SB,,Fusibile Slow Blow 2A 250V 5x20mm,AliExpress,0.20,0.20
1,Fusibile 2A SB 250V,F2,5x20-2A-SB,,Fusibile Slow Blow 2A 250V 5x20mm,AliExpress,0.20,0.20
1,Fusibile 1A SB 250V,F_HEAT,5x20-1A-SB,,Fusibile Slow Blow 1A 250V 5x20mm (riscaldatore 220V),AliExpress,0.20,0.20
1,TVS 1.5KE24A,D1,1.5KE24A-DO201,,Diodo TVS unipolare protezione picchi,AliExpress,0.30,0.30
2,Diodo 1N5819,D2 D3,1N5819-DO41,,Diodo Schottky fly-back,AliExpress,0.10,0.20
1,Condensatore 100µF 25V,C1,100µF-25V-RADIAL,,Condensatore elettrolitico bulk +5V,AliExpress,0.15,0.15
1,Condensatore 100µF 25V,C2,100µF-25V-RADIAL,,Condensatore elettrolitico bulk +12V,AliExpress,0.15,0.15
2,Condensatore 470µF 25V,C3 C4,470µF-25V-RADIAL,,Condensatore elettrolitico bulk +12V (ventola) e +24V,AliExpress,0.20,0.40
1,Condensatore 470µF 35V,C5,470µF-35V-RADIAL,,Condensatore elettrolitico bulk +24V (umidificatore),AliExpress,0.25,0.25
6,Condensatore 100nF Ceramico,C_X,100nF-50V-CERAMIC,,Condensatore ceramico decoupling,AliExpress,0.05,0.30
3,Resistore 220Ω 1/4W,R_gate,220-1/4W,,Resistenza gate MOSFET protezione ESD,AliExpress,0.02,0.06
6,Resistore 10kΩ 1/4W,R_pull,10k-1/4W,,Resistenza pull-up/pull-down,AliExpress,0.02,0.12
4,Resistore 4.7kΩ 1/4W,R_I2C,4.7k-1/4W,,Resistenza pull-up I2C bus,AliExpress,0.02,0.08
2,Resistore 4.7kΩ 1/4W,R_TACH,4.7k-1/4W,,Resistenza partitore tachimetro,AliExpress,0.02,0.04
1,Resistore 1kΩ 1/4W,R_LED,1k-1/4W,,Resistenza LED di stato,AliExpress,0.02,0.02
1,Resistore 15kΩ 1/4W,R_TACH_H,15k-1/4W,,Resistenza partitore tachimetro (alto),AliExpress,0.02,0.02
1,Resistore 100kΩ 1/4W,R_PULL_COL,100k-1/4W,,Resistenza pull-down collettore BC547,AliExpress,0.02,0.02
1,Filo Scaldante,HEATER,H05-5M-18W,,Filo scaldante autoregolante 5m 18W/m = 90W,AliExpress,12.00,12.00
1,Termostato Meccanico,THERMOSTAT,BIMETAL-45C,,Termostato bimetallico 45°C reset manuale (CRITICO),AliExpress,3.00,3.00
1,Cavo Schermato 1-Wire,CABLE_1W,22AWG-SHIELD-5M,,Cavo schermato 1-Wire DS18B20 5m,AliExpress,2.00,2.00
1,Connettore JST 3-pin,J_DS1,JST-3PIN-2.54,,Connettore sensore DS18B20,AliExpress,0.30,0.30
1,Connettore JST 3-pin,J_DS2,JST-3PIN-2.54,,Connettore sensore DS18B20,AliExpress,0.30,0.30
1,Connettore JST 4-pin,J_OLED,JST-4PIN-2.54,,Connettore display touch,AliExpress,0.30,0.30
1,Connettore JST 5-pin,J_ENC,JST-5PIN-2.54,,Connettore encoder,AliExpress,0.40,0.40
1,Connettore JST 5-pin,J_PSU,JST-5PIN-2.54,,Connettore PSU +5V +12V +24V GND,AliExpress,0.40,0.40
1,Connettore Molex 2-pin,J_VENTOLA,MOLEX-2PIN-5.08,,Connettore ventola 12V,AliExpress,0.50,0.50
1,Connettore Molex 2-pin,J_LED,MOLEX-2PIN-5.08,,Connettore striscia LED 12V,AliExpress,0.50,0.50
1,Connettore KF350 2-pin,J_HUMID,KF350-2PIN-5.08,,Connettore umidificatore 24V,AliExpress,0.30,0.30
1,Connettore KF350 2-pin,J_SSR,KF350-2PIN-5.08,,Connettore SSR controllo,AliExpress,0.30,0.30
1,Portafusibili 5x20,FH1,5x20-FUSEHOLDER,,Portafusibili per F1 F2,AliExpress,0.80,0.80
1,Portafusibili 5x20,FH_HEAT,5x20-FUSEHOLDER,,Portafusibili per F_HEAT (220V board separata),AliExpress,0.80,0.80
1,PCB Prototipo eurocard,PCB,EUROCARD-160x100,,PCB monofaccia rame,JLC-PCB/Modellismo,5.00,5.00
1,Kit saldatura stagno,SOLDER,LEAD-FREE-250G,,Stagno piombo-free 250g,AliExpress,3.00,3.00
1,Kit morsettiere 220V,TB_AC,MORSETTI-3PIN-10A,,Morsettiera ingresso fase/neutro/terra per board 220V,AliExpress,2.00,2.00
1,Dissipatore SSR alluminio,HEATSINK,ALUMINUM-50x50,,Dissipatore SSR minimo 10 cm² (opzionale se T < 35°C),AliExpress,2.00,2.00
1,Grasso termico,THERMAL,ARCTIC-MX-6,,Grasso termico silicone per SSR,AliExpress,2.00,2.00
1,Cablaggio vario,WIRE-KIT,22AWG-10COLORS,,Cavi jumper 22 AWG 10 colori 100m,AliExpress,5.00,5.00
1,Connettori dupont,DUPONT-KIT,DUPONT-20PIN,,Set connettori dupont (opzionale per prototipo),AliExpress,1.00,1.00

SUMTOTAL (componenti),,,,,,Subtotale,217.72
,,,,,,,
Note aggiuntive:,,,,,
- Costi da AliExpress/Mouser/Digikey cambiano per ordini grandi,
- Lead time: verificare disponibilità SCD41 (raro) e ESP32 (talvolta out-of-stock),
- PSU ATX 260W può essere usato vecchio/ricondizionato da rottamazione PC (~5-8€),
- Filo scaldante può essere ordinato custom da fornitore cinese (~10-15€ per 5m),
- Ventola Noctua è costosa ma silenziosa; alternative economiche: Be Quiet (~12€) o generiche (~5€),
- Display touch 3.5": verificare compatibilità con ST7789 o ILI9341 (driver ESPHome),
```

## Riepilogo Costi

| Categoria | Costo EUR | Note |
|---|---|---|
| **MCU + Sensori** | ~75 | ESP32, SHT45, SCD41, DS18B20x2 |
| **Relè + Driver** | ~15 | SSR-25DA, BC547, IRLZ44N, diodi |
| **Carichi** | ~50 | Ventola, LED, umidificatore, filo scaldante |
| **Alimentazione** | ~30 | PSU ATX, fusibili, protezioni, capacitori |
| **Interfaccia** | ~25 | Display touch, encoder, pulsanti, LED |
| **Connettori + PCB** | ~12 | Morsettiere, connettori, PCB prototipo |
| **TOTALE** | **~207-220 EUR** | Senza lavoro installazione/cablaggio |

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo, pronto per realizzazione
