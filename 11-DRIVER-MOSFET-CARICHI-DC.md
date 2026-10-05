# DRIVER MOSFET — CARICHI DC (LED, Ventola, Umidificatore) v1.1

## 1. Panoramica Driver MOSFET

Tre carichi DC a bassa tensione (+12V e +24V) sono controllati tramite MOSFET N-channel logic-level IRLZ44N in configurazione **low-side switching**, con uscita PWM o digitale dai GPIO dell'ESP32.

### Vantaggi IRLZ44N

| Caratteristica | Valore | Vantaggi |
|---|---|---|
| **Tecnologia** | Logic-level N-ch | Si accende completamente a VGS = 3.3V (no bootstrap) |
| **Rds_on** | < 22 mΩ @ 5V | Bassissima dissipazione termica (<< 0.1W tipico) |
| **Vth** | 1.0–2.0 V | Soglia bassa, compatibile 3.3V direttamente |
| **τ_turn** | < 100 ns | Commutazione rapida, adatta PWM 25 kHz |
| **Rth_ja** | ~62 °C/W | Contenuta, no dissipatore per bassi carichi |
| **Package** | TO-220 | Facile da montare, dissipatore standard |
| **Costo** | Economico | ~0.50‒1 EUR / pezzo |

---

## 2. Circuito Base — Low-Side Switching

```
        +12V o +24V (source di potenza)
             │
        ┌────┴────┐
        │  Carico L  │  (LED, ventola, umidificatore)
        │  (R, motor)│
        │            │
        └────┬────┘  ← Drain MOSFET
             │
        ┌────▼────┐
        │  Q (IRLZ44N)│  ← MOSFET Gate-Source
        │            │
        └────┬────┘  ← Source MOSFET
             │
           GND (Massa comune ESP32)
```

### Principio di Funzionamento

- **GPIO HIGH (3.3V):** VGS = 3.3V → MOSFET satura → Rds_on ≈ 22 mΩ → Drain ≈ GND → **Carico ON**
- **GPIO LOW (0V):** VGS = 0V → MOSFET OFF → Drain flottante → **Carico OFF**

---

## 3. Circuito Completo con Protezioni

```
+VDD (+12V o +24V) ──┬─────────────────────────────────────────────┬──────────────┬──────────────┐
           │                                                │                         │
           │                                                │                         │
      ┌────▼────────────────────────────────────────┬────▼─────┐  ┌────▼─────┐
      │ Carico (LED, ventola, umidif.) │         │ Fly-back  │  │ Bulk cap  │
      │                                │         │ Diode    │  │ (se motor)│
      │                                │         │ 1N5819   │  │           │
      └────┬────────────────────────────────────────┘  │ Anodo └──────────┘
           │  (Drain Q)                        │   |
           │                                   │   |
           │                                   └─── Catodo (verso +VDD)
           │
      ┌────▼────────────────────────────────────────┐
      │                Gate Resistor              │  (protegge da ESD)
      │ GPIO ── R_gate 220Ω ── Gate Q    │
      │                                        │
      └────┬────────────────────────────────────────┘
           │
      ┌────▼────────────────────────────────────────┐
      │                Pull-down Resistor        │  (mantiene gate al boot)
      │        R_pull 10kΩ verso GND           │
      │                                        │
      └────┬────────────────────────────────────────┘
           │
      ┌────▼────────────────────────────────────────┐
      │              Source MOSFET              │  (Emettitore Q)
      │                  GND                     │
      └────────────────────────────────────────┘
```

---

## 4. Specifiche per Carico: Striscia LED 12V (GPIO 33)

### Parametri Carico

| Parametro | Valore | Note |
|---|---|---|
| Tensione nominale | 12 V DC | Linea +12V da PSU ATX |
| Corrente massima | 1.5 A | A luminosità 100% |
| Corrente tipica | 0.8–1.0 A | A luminosità 70–80% |
| Tipo | WS2812B RGB o RGBW | PWM dimmer oppure protocollo digitale DIN |
| Frequenza PWM | 20–25 kHz | Invisibile all'occhio, no sfarfallio |
| GPIO | 33 | PWM output LEDC |

### Circuito LED Strip

```
+12V_SYS (dopo bulk cap + TVS) ──┬─────────────────────────────┐
                              │                                      │
                         ┌────▼──────────────────────────────┐  Anodo
                         │    Fly-back Diode (1N5819)             │  oppure
                         │    ≥ 0.5A, Vf 0.35V                  │  induttanza
                         │    (opzionale per LED, consigliato)   │
                         └────┬──────────────────────────────┘  Catodo
                             │
                         ┌────▼──────────────────────────────┐
                         │     Striscia LED 12V                 │
                         │     (RGB/RGBW, < 1.5A)                │
                         │                                       │
                         └────┬──────────────────────────────┘
                             │ (Drain Q2 IRLZ44N)
                             │
 GPIO 33 (PWM 25kHz) ── R 220Ω ── Gate Q2 IRLZ44N
                             │
                        ┌────▼──────────────────────────────┐
                        │       Pull-down 10kΩ verso GND       │
                        │                                       │
                        └────┬──────────────────────────────┘
                             │
                           GND_SYS
```

### Parametri Circuito

| Elemento | Valore | Calcolo/Note |
|---|---|---|
| **R_gate** | 220 Ω | Limitazione ESD, (3.3V-0.2V)/I_gate < 50mA |
| **R_pull** | 10 kΩ | Impedisce flottamento gate al boot |
| **Fly-back diode** | 1N5819 | Opzionale per LED (non ci sono spike induttivi importanti) |
| **PWM freq** | 25 kHz | Invisibile, ottimale per LED |
| **Duty cycle** | 0–100% | Controllo luminosità 0–100% |

---

## 5. Specifiche per Carico: Ventola Ricircolo 12V (GPIO 25)

### Parametri Carico

| Parametro | Valore | Note |
|---|---|---|
| Tensione nominale | 12 V DC | Linea +12V da PSU ATX |
| Corrente nominale | 0.3 A | A regime, ventilatore silenzioso |
| Corrente massima | 0.5 A | A 100% PWM |
| Corrente di spunto | 0.8 A per 10–50 ms | All'accensione, motore spazzolato |
| Tipo | Ventola CC con cuscinetti a sfere | 4-pin PWM + tach |
| Frequency PWM | 25 kHz | Riduce rumore commutazione MOSFET |
| GPIO PWM | 25 | PWM LEDC |
| GPIO TACH | 34 | Pulse counter per misura RPM |

### Circuito Ventola (Low-Side PWM)

```
+12V_SYS ──┬─────────────────────────────────────────────────┐
      │                                                           │
      │                                                     ┌─────┴─────┐
      │                                                     │    Fly-back  │
      │                                                     │    1N5819    │
      │                                                     │    Anodo     │
      │                                                     └────┴─────┘
      │                                                          │
      │                                                   ┌────▼──────┐
      │                                                   │  Ventola   │
      │                                                   │  (+12V,    │
      │                                                   │  GND,      │
      │                                                   │  PWM, TAC) │
      │                                                   │            │
      └─────────────────────────────────────────────────┴───────────┘
                                                                Drain Q1

GPIO 25 (PWM 25kHz) ── R 220Ω ── Gate Q1 (IRLZ44N) ── R 10kΩ pull-down ─ GND
```

### Parametri Circuito

| Elemento | Valore | Calcolo/Notes |
|---|---|---|
| **R_gate** | 220 Ω | Limitazione di corrente, protezione ESD |
| **R_pull** | 10 kΩ | Pull-down gate, impedisce flottamento al boot |
| **Fly-back diode** | 1N5819 Schottky | **CRITICO**: protegge da picchi BEMF del motore |
| **Fly-back position** | Anodo drain, catodo +12V | Assorbe picchi induttivi durante turn-off |
| **PWM freq** | 25 kHz | Riduce rumore acustico del MOSFET |
| **Duty cycle** | 0–100% | Controllo velocità ventola 0–100% |
| **Spunto (10–50ms)** | 0.8A max | Fusibile F1 (2A SB) tollera benissimo |

### Calcolo Dissipazione Ventola

```
Potenza nominale: P = 0.3A × 12V = 3.6W
Rds_on IRLZ44N: 22 mΩ @ 5V, ≈ 25-30 mΩ @ 3.3V
Potenza dissipata MOSFET (a 0.3A): P_diss = (0.3A)^2 × 25mΩ ≈ 2.25mW

Elevazione T: ΔT = 2.25mW × 62°C/W ≈ 0.14°C

✅ Dissipatore NO necessario. MOSFET può stare su pad di rame 0.5 cm² per dissipazione passiva.
```

---

## 6. Specifiche per Carico: Umidificatore Ultrasuoni 24V (GPIO 26)

### Parametri Carico

| Parametro | Valore | Note |
|---|---|---|
| Tensione nominale | 24 V DC | Linea dedicata +24V (esterno o PSU ATX) |
| Corrente nominale | 0.8 A | A pieno vapor fuori |
| Corrente massima | 1.5 A | Picco transiente all'accensione |
| Potenza nominale | ~20 W | 24V × 0.8A |
| Controllo | ON/OFF o PWM | Intermittente, consigliato PWM per risparmiare |
| GPIO | 26 | PWM/Digitale output |
| Duty cycle | 0–100% | 0% = no vapore, 100% = massima produzione |

### Circuito Umidificatore

```
+24V_SYS ──┬─────────────────────────────────────────────────────┐
      │                                                                │
      │                                                         ┌─────┴─────┐
      │                                                         │    Fly-back  │
      │                                                         │    1N5819    │
      │                                                         │    Anodo     │
      │                                                         └────┴─────┘
      │                                                              │
      │                                                       ┌────▼──────┐
      │                                                       │ Umidif.   │
      │                                                       │ Ultrasoni │
      │                                                       │ 24V       │
      │                                                       │ < 1.5A    │
      └─────────────────────────────────────────────────────┴──────────┐
                                                                   Drain Q3

GPIO 26 (PWM/dig) ── R 220Ω ── Gate Q3 (IRLZ44N) ─ R 10kΩ pull-down ─ GND

[Nota: Umidificatore spesso ha regolatore 24V integrato; se disponibile,
 considerare di collegare GND umidificatore direttamente a GND PSU
 per evitare loop di massa]
```

### Parametri Circuito

| Elemento | Valore | Note |
|---|---|---|
| **R_gate** | 220 Ω | ESD protection |
| **R_pull** | 10 kΩ | Pull-down gate |
| **Fly-back diode** | 1N5819 | Protezione picchi trasducer ultrasuoni |
| **PWM freq** | 10–25 kHz | Inaudibile, controllabile in PWM |
| **Duty cycle** | 0–100% | Intermittenza per gestire RH |
| **Bulk cap +24V** | 470 µF | Stabilizzazione linea +24V |

### Calcolo Dissipazione Umidificatore

```
Corrente nominale: 0.8A @ 24V
Rds_on IRLZ44N: ≈ 25–30 mΩ @ 3.3V
Potenza dissipata MOSFET: P_diss = (0.8A)^2 × 25mΩ ≈ 16mW

Elevazione T: ΔT = 16mW × 62°C/W ≈ 1°C

✅ Dissipatore NO necessario. MOSFET su pad di rame 0.5 cm².
```

---

## 7. Protezioni e Dissipazione Termica

### Fly-back Diodes: Critical!

⚠️ **Il diodo fly-back è ESSENZIALE per ventola e umidificatore**, perché contengono carichi induttivi (motori, trasduttori ultrasuoni).

**Perché?**
- Quando il MOSFET si spegne, l'induttanza del carico genera un picco di tensione opposto (BEMF)
- Senza protezione, questo picco può distruggere il MOSFET (Drain-Source breakdown) o il GPIO
- Il diodo 1N5819 assorbe il picco facendo circolare la corrente induttiva

**Specifiche diodo:**
- **1N5819 Schottky:** Vf ≈ 0.35V, τ < 100ns, If ≥ 1A per ventola
- **Polarità:** Anodo verso Drain MOSFET, Catodo verso +VDD (linea di potenza)
- **Posizionamento:** Il più vicino possibile al MOSFET, < 5cm di pista

### Dissipatori MOSFET

| Carico | Corrente | P_diss | Rth_ja (no diss.) | ΔT | Dissipatore |
|---|---|---|---|---|---|
| LED | 1.5 A | ~50 mW | 62°C/W | ~3°C | ❌ No |
| Ventola | 0.5 A | ~6 mW | 62°C/W | ~0.4°C | ❌ No |
| Umidificatore | 1.5 A | ~56 mW | 62°C/W | ~3.5°C | ❌ No |

**Conclusione:** Nessun dissipatore necessario per nessun MOSFET (carichi DC bassi). Montare MOSFET su pad di rame ≥ 0.5 cm² per dissipazione passiva.

---

## 8. Configurazione ESPHome

```yaml
output:
  # GPIO 33: LED Strip PWM dimmer
  - platform: ledc
    pin: GPIO33
    frequency: 25000Hz
    id: led_pwm
    
  # GPIO 25: Ventola PWM
  - platform: ledc
    pin: GPIO25
    frequency: 25000Hz
    id: fan_pwm
    
  # GPIO 26: Umidificatore PWM/ON-OFF
  - platform: ledc
    pin: GPIO26
    frequency: 10000Hz
    id: humid_pwm

light:
  # LED Strip (brightnessON/OFF)
  - platform: monochromatic
    output: led_pwm
    name: "LED Strip"
    id: led_strip
    
fan:
  # Ventola (speed control 0-100%)
  - platform: speed
    output: fan_pwm
    name: "Ventola Ricircolo"
    id: ventola_ricircolo

switch:
  # Umidificatore ON/OFF semplice
  - platform: output
    name: "Umidificatore"
    id: umidificatore
    output: humid_pwm
    
  # Oppure, se vuoi PWM dimmer:
  # - platform: template
  #   name: "Umidificatore PWM"
  #   lambda: |-
  #     return id(humid_pwm).state > 0.01;
  #   turn_on_action:
  #     - output.set_level:
  #         id: humid_pwm
  #         level: 0.8  # 80% PWM
  #   turn_off_action:
  #     - output.set_level:
  #         id: humid_pwm
  #         level: 0.0
```

---

## 9. Checklist Realizzazione MOSFET

- [ ] Saldare R_gate 220Ω tra GPIO e Gate MOSFET (vicino al pin)
- [ ] Saldare R_pull 10kΩ tra Gate e GND (pull-down)
- [ ] Saldare 1N5819 fly-back tra Drain e +VDD (anodo Drain, catodo +VDD)
- [ ] Verificare orientamento MOSFET (Gate-Drain-Source corretti)
- [ ] Montare MOSFET su pad di rame ≥ 0.5 cm² (dissipazione passiva)
- [ ] Verificare diodo voltaggi: Drain verso +VDD < 30V picco (ok per 12V, 24V)
- [ ] Testare con multimetro: Gate LOW → Drain flottante, Gate HIGH → Drain = GND
- [ ] Carica firmware ESPHome e prova PWM con multimetro o oscilloscopio

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo, pronto per realizzazione
