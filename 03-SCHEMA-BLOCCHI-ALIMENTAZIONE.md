# SCHEMA A BLOCCHI — ALIMENTAZIONE v1.1

## 1. Overview Flusso di Potenza

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           RETE 220V AC 50Hz                              │
│                        (con protezione RCD opzionale)                    │
└─────────────────────────────────┬──────────────────────────────────────┘
                                  │
                  ┌───────────────┴──────────────┐
                  │                              │
            ┌─────▼─────┐                ┌──────▼──────┐
            │ PSU ATX   │                │ FUSIBILE    │
            │ 260 W     │                │ F_HEAT 1A   │
            │ +5V       │                │ (Riscald.)  │
            │ +12V      │                └──────┬──────┘
            │ +24V      │                       │
            └─┬───┬───┬─┘                 ┌─────▼─────┐
              │   │   │                   │    SSR    │
              │   │   │                   │  25A AC   │
              │   │   │                   │(Riscald.) │
         +5V  │   │   │  +24V             └─────┬─────┘
              │   │   │                         │
    ┌─────────┘   │   └────────────┐     ┌──────▼───────┐
    │             │                │     │ Filo Scaldante
    │         +12V│                │     │ 90W @ 220V AC
    │             │                │     └──────────────┘
    │             │        ┌───────▼────────┐
    │             │        │ UMIDIFICATORE  │
    │             │        │ 24V DC <= 1.5A │
    │             │        └────────────────┘
    │             │
    │  ┌──────────┴──────────┬──────────┐
    │  │                     │          │
  ┌─▼──▼──┐           ┌──────▼──┐  ┌───▼────┐
  │ ESP32  │           │ VENTOLA │  │ LED    │
  │ (3.3V) │           │ 12V PWM │  │ 12V    │
  │        │           └────┬────┘  └────┬───┘
  │        │                │            │
  │        │           ┌────▼────┐  ┌────▼───┐
  │        │           │ MOSFET  │  │ MOSFET │
  │        │           │ Q1      │  │ Q2     │
  │        │           │ PWM/dig │  │ PWM/dig│
  │        │           └────┬────┘  └────┬───┘
  │        │                │            │
  │  ┌─────▼────┐      ┌────▼───┐  ┌────▼───┐
  │  │ I²C BUS  │      │+12V_SYS│  │+12V_SYS│
  │  │          │      └────────┘  └────────┘
  │  │ SHT45    │     (con bulk cap + TVS)
  │  │ SCD41    │
  │  └──────────┘     ┌──────────────┐
  │  ┌─────────────┐  │  1-Wire BUS  │
  │  │  Encoder    │  │              │
  │  │  Pulsanti   │  │ DS18B20 x2   │
  │  └─────────────┘  └──────────────┘
  │
  │  ┌──────────────────────┐
  │  │ Display Touch 3.5"    │
  │  │ (SPI oppure I²C)      │
  │  └──────────────────────┘
  └────────────────────────────┘

```

---

## 2. Linea +5V (da PSU ATX)

### Componenti sulla Linea

```
PSU ATX +5V OUT ────┬──── Fusibile F2 (2A SB) ────┬──── +5V_SYS
                    │                              │
                    └─ Protezione cortocircuito   └──┬─── R_carico
                                                     │
                                              ┌──────┴──────┐
                                              │             │
                                         ┌────▼──┐    ┌────▼──┐
                                         │ C1    │    │ C2    │
                                         │ 100µF │    │ 100nF │
                                         │ 25V   │    │       │
                                         └───┬───┘    └───┬───┘
                                             │            │
                                         GND_SYS ────GND_SYS
```

### Carichi Alimentati @ +5V

| Carico | Corrente | Protezione | Note |
|--------|----------|-----------|-------|
| ESP32 VIN | 240 mA | Regolatore interno | Linea critica, vicino a VIN |
| PCA9548A VCC | 5 mA | Sopra-corrente F2 | Alimentato da +3.3V ESP32 |
| SHT45 VCC | 2 mA | Sopra-corrente F2 | Alimentato da +3.3V ESP32 |
| SCD41 VCC | 5 mA | Sopra-corrente F2 | Alimentato da +3.3V ESP32 |
| Display touch VCC | 100 mA | Sopra-corrente F2 | Picco durante refresh |
| LED di stato | 2 mA | Sopra-corrente F2 | Tramite resistenza 1kΩ |
| SSR+ (controllo) | < 1 mA | BC547 driver | Non attraversa fusibile diretto |
| **TOTALE** | **~350 mA** | **Fusibile 2A** | **✅ Margin 5.7x** |

### Protezioni +5V

- **F2 Fusibile 2A SB (Slow Blow):** Interviene se corrente > 2A per più di 5-10 secondi
- **Condensatori:** C1 (100µF bulk) + C2 (100nF ceramic) per stabilità e rumore
- **Decoupling:** 100nF ceramico vicino a ogni IC alimentato a 5V (PCA9548A, SCD41, display)

---

## 3. Linea +12V (da PSU ATX)

### Circuito di Protezione

```
PSU ATX +12V OUT ────┬──── TVS 1.5KE24A ──── +12V_protected
                    │     (clamp a 24V)
                    │
            Protezione picchi/ESD
                    │
             ┌──────▼──────┐
             │ F1 (2A SB)  │ ← Fusibile slow-blow, protezione sovracorrente
             └──────┬──────┘
                    │
          ┌─────────┴─────────┐
          │                   │
      ┌───▼────┐          ┌───▼────┐
      │  C3    │          │  C4    │
      │ 470µF  │          │ 100nF  │
      │ 25V    │          │        │
      └───┬────┘          └───┬────┘
          │ (bulk cap)        │ (ceramic)
          │                   │
     GND_SYS ───────────────GND_SYS
```

### Carichi Alimentati @ +12V

| Carico | Corrente | MOSFET Driver | Protezione | Note |
|--------|----------|---------------|-----------|-------|
| **Striscia LED** | 1.0 A tipico | IRLZ44N Q2 (GPIO 33 PWM) | Fly-back integrato | Massima corrente da considerare |
| **Ventola ricircolo** | 0.3 A tipico | IRLZ44N Q1 (GPIO 25 PWM) | 1N5819 fly-back | PWM 25 kHz, tach su GPIO 34 |
| **TOTALE** | **~1.3 A** | Separate | **Fusibile 2A SB** | **✅ Margin 1.5x** |

### Driver MOSFET per +12V

```
+12V_SYS ────┬──────────────┬───────────────┐
             │              │               │
        ┌────▼──────┐  ┌────▼──────┐  ┌────▼──────┐
        │  Striscia │  │  Ventola   │  │ Diodo FW  │
        │   LED     │  │ Ricircolo  │  │ (1N5819)  │
        │          │  │            │  └────┬──────┘
        └────┬──────┘  └────┬──────┘       │
             │              │              │
        ┌────▼──────┐  ┌────▼──────┐       │
        │ Drain Q2  │  │ Drain Q1  │  Anodo
        │ IRLZ44N   │  │ IRLZ44N   │  (verso +12V)
        └────┬──────┘  └────┬──────┘
             │              │
      ┌──────▴──────┐  ┌─────▴──────┐
      │    GND      │  │    GND     │
      └─────────────┘  └────────────┘
```

### Protezioni +12V

- **TVS 1.5KE24A:** Protezione picchi, clamp a 24V (catodo verso +12V, anodo verso GND)
- **F1 Fusibile 2A SB:** Sovracorrente, interviene lentamente (5-10 s) per tollerare spunti normali
- **Condensatori:** C3 (470µF bulk) + C4 (100nF ceramic) per stabilità
- **Diodi fly-back 1N5819:** Su ogni carico induttivo (ventola, relay), protezione BEMF

---

## 4. Linea +24V (da PSU ATX o alimentatore esterno)

### Opzione A: PSU ATX con linea +24V nativa (rara)

```
PSU ATX +24V OUT ──── Fusibile 1A SB ──── +24V_SYS ──┬──── C5 (470µF) ──┬── GND
                                                     │                  │
                                              Umidificatore         Decoupl.
                                              MOSFET Q3              C6
                                              (GPIO 26)              100nF
```

### Opzione B: Alimentatore switching 24V esterno dedicato (consigliato)

```
Alimentatore 24V/2A ──┬──── Fusibile 1A SB ──── +24V_SYS
(esterno)            │
                 Protezione
                     │
                ┌────▼────┐
                │   F_24  │
                │ 1A SB   │
                └────┬────┘
                     │
              ┌──────┴─────┐
              │            │
          ┌───▼──┐    ┌────▼──┐
          │ C5   │    │ C6    │
          │470µF │    │ 100nF │
          │35V   │    │       │
          └───┬──┘    └───┬───┘
              │           │
           GND_SYS ───GND_SYS
```

### Carichi Alimentati @ +24V

| Carico | Corrente | Driver | Protezione | Note |
|--------|----------|--------|-----------|-------|
| **Umidificatore ultrasuoni** | 0.8 A tipico | IRLZ44N Q3 (GPIO 26 PWM/digit) | 1N5819 fly-back | Intermittente, PWM per risparmiare energia |
| **TOTALE** | **~0.8 A** | - | **Fusibile 1A SB** | **✅ Margin 1.25x** |

---

## 5. Linea 220V AC — Riscaldatore

### Schema Completo

```
┌──────────────────────────────────────────────────────────────────┐
│                     RETE 220V AC 50 Hz                           │
│                (con RCD 30mA opzionale CONSIGLIATO)             │
└────────┬──────────────────────────────────────────────┬──────────┘
         │ FASE (L)                                    │ NEUTRO (N)
         │                                             │
    ┌────▼──────────┐                            ┌─────▼──────────┐
    │ Fusibile      │                            │ Nessun          │
    │ F_HEAT 1A SB  │                            │ interruttore    │
    │ 250V 5×20mm   │                            │ (sempre passivo)│
    └────┬──────────┘                            └────────────────┘
         │
    ┌────▼──────────┐
    │   SSR-25DA    │ ← Solid State Relay 25A
    │  AC 220V      │   Controllato da BC547 (GPIO 32)
    │               │
    │ Pin AC1 ◄─────┼─ Ingresso da Fase (L)
    │ Pin AC2 ──────┼─► Uscita verso Filo Scaldante
    └────┬──────────┘   (con protezione termica meccanica)
         │
    ┌────▼──────────┐
    │  Filo Scaldan │ 90W @ 220V, autoregolante 18W/m × 5m
    │  Termostato   │ ◄─── TERMOSTATO MECCANICO di sicurezza
    │  meccanico    │     (interviene se T > 45°C)
    └────┬──────────┘
         │
    ┌────▼──────────┐
    │ Neutro (N)    │ ← Ritorno a rete
    └───────────────┘
```

### Driver SSR (a bassa tensione, 5V DC)

```
GPIO 32 (3.3V logic) ──── R 10kΩ ──── Base BC547 (Q5)

BC547 Saturato:
  Collettore ──┬── R 100kΩ (pull-down) ──── GND
               │
               └── SSR- (controllo negativo)
  Emettitore ──── GND

SSR+ ──── +5V da PSU ATX (sempre collegato)
SSR- ──── Collettore BC547 (commutato)
```

### Protezioni 220V AC

| Protezione | Componente | Funzione |
|-----------|-----------|----------|
| **Protezione differenziale** | RCD 30mA (opzionale, consigliato) | Interviene se corrente Fase ≠ Neutro |
| **Protezione sovracorrente** | Fusibile F_HEAT 1A SB 250V | Slow-blow, tolera spunti resistivi |
| **Protezione termica hardware** | **Termostato meccanico** | **ESSENZIALE: interviene se T > 45°C** |
| **Controllo SSR** | BC547 + R 10kΩ base | Pilotaggio affidabile, fail-safe (SSR spento se GPIO flottante) |
| **Isolamento** | Scatola separata certificata IP40 | Separazione cella da circuiti ad alta tensione |

---

## 6. Piano di Terra (GND)

### Stella di Massa Centralizzata

```
                ┌─────────────────┐
                │  STAR GROUNDING │  ← Punto centrale massa PCB
                │   (0V riferimento)│
                └────┬────┬───────┘
                     │    │
         ┌───────────┘    └────────┬──────────┐
         │                         │          │
    ┌────▼────┐          ┌────────▼──┐   ┌───▼───┐
    │ ESP32   │          │ PSU ATX   │   │ Carichi
    │ GND     │          │ GND       │   │ DC GND
    │         │          │ (negativo)│   │
    └─────────┘          └───────────┘   └───────┘
```

### Linee di Massa

- **Massa PSU → Stella:** Linea singola, spessa (> 2mm), bassa impedenza
- **Massa carichi → Stella:** Raccogliere i GND di tutti i MOSFET in un punto prima della stella
- **Massa sensori → Stella:** Separare se possibile per evitare crosstalk su I²C/1-Wire
- **Massa 220V separata:** Non mescolare GND della scatola 220V con la massa del PCB (separazione fino al PSU)

---

## 7. Dissipazione Termica — SSR @ 220V

### Calcolo Potenza Dissipata

```
Filo scaldante: 90W @ 220V AC
Corrente: I = 90W / 220V ≈ 0.41A
Tensione drop tipico SSR-25DA: V_drop ≈ 1.5V @ 0.4A

Potenza dissipata: P_diss = V_drop × I = 1.5V × 0.41A ≈ 0.6W

Rth_ja (SSR nude): ≈ 40°C/W
Elevazione T: ΔT = 0.6W × 40°C/W ≈ 24°C

Se ambiente: 25°C → T_SSR ≈ 49°C → Dissipatore sconsigliato ma non critico
Se ambiente: 35°C → T_SSR ≈ 59°C → Dissipatore da 10 cm² minimo consigliato
```

### Dissipatore Consigliato

- **Alluminio anodizzato:** minimo 10 cm² di superficie
- **Altezza fin:** ≥ 10 mm per convezione naturale
- **Grasso termico:** applicare tra SSR e dissipatore (K ≈ 1-2 W/mK)
- **Isolamento:** se dissipatore su telaio metallico conduttore, isolante termico tra SSR e telaio

---

## 8. Sommario Protezioni

| Linea | Protezione Primaria | Protezione Secondaria | Fail-Safe |
|-------|-------------------|--------------------|----------|
| **+5V** | Fusibile F2 2A SB | Condensatori bulk/ceramic | Se F2 salta: niente ESP32, circuito OFF |
| **+12V** | Fusibile F1 2A SB + TVS | Diodi fly-back MOSFET | Se F1 salta: no LED, no ventola, circuito parziale |
| **+24V** | Fusibile 1A SB | 1N5819 fly-back | Se salta: no umidificatore |
| **220V AC** | Fusibile F_HEAT 1A SB | **Termostato meccanico** | **Se SSR rimane acceso: termostato interviene** |

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo, pronto per realizzazione
