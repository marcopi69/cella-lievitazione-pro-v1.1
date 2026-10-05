# DRIVER SSR — RISCALDATORE 220V v1.1

## 1. Panoramica Circuito SSR

Il filo scaldante a 220V AC (90W) è controllato tramite un **Solid State Relay (SSR-25DA)** pilotato da un transistor **BC547** in configurazione **open-collector**, alimentato dal **GPIO 32** dell'ESP32 tramite PID.

### Topologia Scelta: BC547 Open-Collector

Questa è la topologia **corretta e definitiva** della v1.1 (corregge errori di v1.0):

```
GPIO 32 (3.3V logic) ─────┬────── R 10kΩ (base) ─────┬────── Base BC547
                          │                         │
                          └─────────────────────────┴────── Gate-control

BC547 Saturato (GPIO HIGH):
  Collettore ─────┬────── SSR- (controllo negativo) ─┬──── Verso SSR
                  │                                  │
                  └────── R 100kΩ (pull-down) ───────┴──── Verso GND
                  
  Emettitore ─────────────────────────────── GND (Massa comune)
  
SSR+ (controllo positivo) ────────────────── +5V DA PSU ATX (sempre)
```

---

## 2. Circuito Completo SSR con Protezioni

```
╔════════════════════════════════════════════════════════════════════════════╗
║                         CIRCUITO DRIVER SSR COMPLETO                       ║
╚════════════════════════════════════════════════════════════════════════════╝

ESP32 GPIO32 (3.3V PWM/logic output)
        │
        ├───────────── R 10kΩ (R_base) ──────────┬──── Base BC547 (Q5)
        │                                        │
        │                                   ┌────┴────┐
        │                                   │ BC547    │
        │                                   │ NPN BJT  │
        │                                   │          │
        │    +5V_SYS (PSU ATX) ─────────────┼──────────┼───── SSR+ (pin 2)
        │         │                         │          │
        │         │                    Coll.(High)   Base
        │         │                         │          │
        │         └─────────────────────────┼──────────┘
        │                                   │
        │                                   ├───── R 100kΩ (R_pull) ───── GND
        │                                   │
        │                              Emett.(Base gnd)
        │                                   │
        │                                   └───── GND_SYS
        │
        └─────────────── SSR- (pin 1) ────── (Controllo negativo)
        
[Nota: SSR ha controllo a due poli: SSR+ sempre a +5V, SSR- commutato da BC547]

╔════════════════════════════════════════════════════════════════════════════╗
║                         LINEA RISCALDATORE 220V AC                         ║
╚════════════════════════════════════════════════════════════════════════════╝

Rete 220V AC ──┬────── Fusibile F_HEAT (1A SB, 250V) ──────┬──── Fase (L)
               │                                           │
               │                                      ┌────┴────┐
               │                                      │ SSR-25DA │
               │                                      │  AC 220V │
               │                                      │          │
               │    Controllo SSR (da BC547) ────────┼──────────┼─── SSR- : GND ctrl
               │         SSR+ ─────────────────────┼──────────┼─── SSR+ : +5V
               │                                      │          │
               │                                  AC1 │ AC2      │
               │                                   (in│)        (out)
               │                                      │          │
               │                                      └────┬─────┘
               │                                           │
               │                                      ┌────┴─────┐
               │                                      │ Filo      │
               │                                      │ Scaldante │
               │                                      │ 90W 220V  │
               │                                      │ (+ termost.
               │                                      │  meccanico)
               │                                      └────┬─────┘
               │                                           │
               └───────────────────────────────────────────┴──── Neutro (N)
```

---

## 3. Specifiche Componenti

### 3.1 SSR-25DA (Solid State Relay)

| Parametro | Valore | Note |
|---|---|---|
| **Tipo** | Relay stato solido 25A | Nessuna parte meccanica mobile |
| **Linea AC** | 220-240V AC, 50/60 Hz | Uso standard domestico |
| **Corrente massima** | 25 A AC | Filo 90W @ 0.41A è ≬ 6% della capacità |
| **Controllo DC** | 3-32 V DC | Accettato range ampio |
| **Corrente di controllo** | ~8 mA @ +5V (tipica) | Definita dal datasheet SSR specifico |
| **Tempo commutazione** | ~10 ms (zero-crossing) | Massimo 1/2 ciclo AC |
| **Dissipazione termica** | ~1W @ 0.41A (tipica) | Vedi calcolo sotto |
| **Temperatura lavoro** | 0-80°C | SSR montato in scatola ben ventilata |
| **Isolamento** | 2000 V galvanico | Perfetto isolamento lato AC / lato DC |

### 3.2 Transistor BC547

| Parametro | Valore | Note |
|---|---|---|
| **Tipo** | NPN BJT | Switching application |
| **Vce(sat)** | ~0.2 V @ Ic = 100mA | Saturation voltage |
| **Vbe** | ~0.7 V @ Ic = 100mA | Base-emitter forward voltage |
| **hFE** | 100-600 (min 100) | Current gain |
| **Ic(max)** | 100 mA | Corrente collettore massima |
| **Potenza dissipata** | << 100 mW | Molto bassa per il nostro uso |
| **Vce(max)** | 50 V (nel nostro caso ~5V) | Ampio margine |
| **Package** | TO-92 | Facile da saldare |

### 3.3 Resistor di Base: R_base (10 kΩ)

**Calcolo:**

```
VGPIO (HIGH) = 3.3 V
VBE (BC547 ON) = 0.7 V
Corrente base richiesta: I_base = (Ic desiderato) / hFE

Per Ic = 200 mA (controllo SSR @ 10 mA):
  I_base_richiesta = 200 mA / 100 = 2 mA (minimo, con hFE=100)

R_base = (V_GPIO - V_BE) / I_base
       = (3.3 V - 0.7 V) / 2 mA
       = 2.6 V / 2 mA
       = 1.3 kΩ

✅ Scegliere R_base = 10 kΩ CONSERVATIVA
   R_base fornisce: I_base = 2.6V / 10kΩ = 0.26 mA
   Questo è ancora sufficiente perché hFE(BC547) ≥ 100 → Ic ≥ 26 mA
   Ampiamente sopra la richiesta SSR (~8 mA)
```

### 3.4 Resistor Pull-Down: R_col (100 kΩ)

**Scopo:** Mantenere SSR disattivato se GPIO è flottante (es. durante boot ESP32).

```
Quando BC547 è OFF (GPIO LOW o flottante):
  Il collettore tira il potenziale verso GND tramite R_col
  Corrente attraverso R_col: I = 5V / 100kΩ = 50 µA
  Potenza dissipata: P = 5V × 50µA = 250 µW (negligible)
  
Quando BC547 è ON (GPIO HIGH, BC547 saturo):
  Collettore va a GND (V_ce_sat ≈ 0.2V)
  Corrente attraverso R_col: I = 0.2V / 100kΩ = 2 µA (negligible)
  Potenza dissipata: P = 0.2V × 2µA = 0.4 µW (negligible)

✅ R_col = 100kΩ va bene: non influisce sulla saturazione, fornisce solo pull-down di sicurezza
```

---

## 4. Verifiche di Funzionamento

### 4.1 Corrente di Controllo SSR

**Scenario:** GPIO 32 HIGH (3.3V), BC547 saturo.

```
SSR richiede corrente di controllo @ +5V: ~8 mA (da datasheet)

Nostra corrente fornita:
  I_base (GPIO → R_base) = (3.3V - 0.7V) / 10kΩ = 0.26 mA
  Ic (BC547, con hFE=100) = 0.26mA × 100 = 26 mA
  
Corrente attraverso SSR-:
  I_SSR = (5V - V_ce_sat) / (R_col in parallelo con SSR-)
  ≈ (5V - 0.2V) / 100kΩ ≈ 48 µA (NO! questa è sbagliata)
  
[CORRIZIONE: La corrente SSR non passa attraverso R_col quando BC547 è saturo.
R_col è SOLO un pull-down. La corrente di controllo SSR viene fornita
dall'SSR stesso tramite il suo circuito interno. In questo caso:

SSR vede: SSR+ = +5V sempre, SSR- = Collettore BC547 (quando ON, GND)
Differenza = +5V - 0V = +5V
SSR ha resistenza controllo interna (tipica ~500Ω)
Corrente controllo = 5V / 500Ω ≈ 10 mA ✓ OK]

✅ Corrente di controllo sufficiente. SSR si attiverà con certezza.
```

### 4.2 Margine di Saturazione BC547

```
Corrente base fornita: 0.26 mA
Corrente base minima per hFE=100: I_base_min = Ic / 100 = (corrente SSR) / 100

SSR-25DA richiede ~10mA controllo:
  I_base_min = 10mA / 100 = 0.1 mA
  
Nostra I_base = 0.26 mA > 0.1 mA ✓

Margine: 0.26 / 0.1 = 2.6× ✅ Abbondante (minimo 2× richiesto)
```

### 4.3 Tensione di Controllo SSR

```
SSR+ = +5V (sempre collegato)
SSR- = Collettore BC547 quando ON ≈ 0.2V (V_ce_sat)

Differenza tensione controllo = 5V - 0.2V = 4.8V

SSR-25DA accetta 3-32V DC ✓ 4.8V è ben dentro il range
```

---

## 5. Protezioni e Sicurezza

### 5.1 Fusibile Linea Riscaldatore (F_HEAT 1A SB)

**Calcolo:**

```
Filo scaldante: 90W @ 220V AC
Corrente nominale: I = 90W / 220V ≈ 0.41A
Fusibile scelto: 1A Slow Blow

Margine: 1A / 0.41A ≈ 2.4× ✓

Slow Blow tollera transitoriamente fino a 2-3A per 5-10 secondi
senza intervenire (ideale per carichi resistivi senza spunto)
```

### 5.2 Termostato Meccanico di Sicurezza ⚠️

**ESSENZIALE: Protezione contro SSR "incollato" (guasto MOSFET interno)**

```
Problema: Se SSR rimane acceso in cortocircuito,
          il software non lo sa e il filo può surriscaldarsi indefinitamente.

Soluzione: Termostato meccanico bimetallico
           - Interviene a ~45-50°C (temper. massima cell)
           - Apre il circuito di alimentazione AC
           - Completamente indipendente dall'elettronica
           
Montaggio: In serie al neutro del riscaldatore,
           fisicamente vicino al filo scaldante
           
Specifiche: Bimetallico, reset manuale, 45°C ± 5°C
```

### 5.3 Isolamento e Layout

- **PCB principale:** Circuiti a bassa tensione (3.3V, 5V, 12V, 24V)
- **Scatola separata 220V:** SSR, fusibile, morsettiere AC, termostato
- **Separazione galvanica:** Almeno 3 mm clearance tra piste AC e DC
- **Cablaggio 220V:** H05VV-F sezione ≥ 1.5 mm² per 90W @ 220V

---

## 6. Configurazione ESPHome

```yaml
# GPIO 32: SSR Riscaldatore (output PID)
output:
  - platform: ledc
    pin: GPIO32
    frequency: 1000Hz        # PWM lento, compatibile con SSR
    min_power: 0.0
    max_power: 1.0
    id: heater_ssr
    
climate:
  - platform: pid
    name: "Controllo Temperatura Riscaldamento"
    sensor: sht45_temp       # Sensore SHT45 (T/RH)
    default_target_temperature: 28
    min_temperature: 15
    max_temperature: 40
    
    heat_output: heater_ssr
    
    # Tuning PID - adattare al volume specifico (198L)
    control_parameters:
      kp: 0.5           # Proporzionale
      ki: 0.02          # Integrale (drift slow)
      kd: 0.1           # Derivativo (damping)
      
    # Isteresi per evitare oscillazioni
    visual:
      - min: 15
        max: 40
```

### Logica Soft Interlock

```yaml
automation:
  # Se T > 45°C → Riscaldatore OFF + Allarme
  - trigger:
      platform: numeric_state
      entity_id: sensor.sht45_temperature
      above: 45
    action:
      - output.turn_off: heater_ssr
      - logger.log:
          level: ERROR
          message: "Temperatura CRITICA > 45°C! Riscaldatore spento, controllare termostato meccanico"
```

---

## 7. Checklist Montaggio e Test

### Montaggio PCB
- [ ] Saldare R_base 10kΩ tra GPIO32 e Base BC547
- [ ] Saldare R_col 100kΩ tra Collettore BC547 e GND
- [ ] Saldare BC547 verticale (Gate-Drain-Source corretti)
- [ ] Saldare condensatore 100nF vicino a BC547 per decoupling
- [ ] Saldare due fili verso connettore SSR (SSR+, SSR-)
- [ ] Verificare con multimetro: base flottante → collettore flottante

### Test Software
- [ ] Carica firmware ESPHome con output SSR GPIO32
- [ ] Accendi seria console e dai comando: `output.turn_on heater_ssr`
- [ ] Verifica con multimetro GPIO32: deve andare a 3.3V
- [ ] Se SSR emette click, il circuito funziona ✓
- [ ] Dai comando: `output.turn_off heater_ssr` e verifica GPIO32 → 0V

### Test Hardware 220V ⚠️
- [ ] **SOLO CON SUPERVISOR ESPERTO**: Collegamento AC
- [ ] Verifica isolamento: SSR riguardato, cavi separati da circuito DC
- [ ] Alimenta PSU e ESP32
- [ ] Con multimetro AC (o pinza), misura corrente linea riscaldatore OFF: ~0A
- [ ] Con ESPHome, accendi: `output.turn_on heater_ssr`
- [ ] Misura corrente: deve essere ~0.4A ✓
- [ ] Spegni: `output.turn_off heater_ssr`
- [ ] Corrente ritorna a 0A ✓

---

## 8. Troubleshooting

| Sintomo | Causa Probabile | Soluzione |
|--------|---|---|
| SSR non si attiva (no click) | GPIO32 non va HIGH | Verificare firmware ESPHome, GPIO assignment |
| GPIO32 HIGH ma SSR no attiva | BC547 saturato ma corrente SSR insufficiente | Ridurre R_base a 4.7kΩ (raddoppia I_base) |
| Filo sempre caldo (SSR incollato) | Guasto interno SSR (raro) | Termostato meccanico interviene, sostituire SSR |
| Instabilità PID (oscillazioni) | Ki troppo alto o Kd troppo basso | Ridurre Ki a 0.01, aumentare Kd a 0.2 |
| Riscaldamento lento | Kp troppo basso oppure volume eccessivo | Aumentare Kp a 1.0, verificare isolamento cella |

---

## 9. Parametri Finali (Riepilogo)

| Elemento | Valore | Range Accettabile | Note |
|---|---|---|---|
| **R_base** | 10 kΩ | 4.7-22 kΩ | Conservativa, garantisce saturazione |
| **R_col** | 100 kΩ | 10-470 kΩ | Pull-down, non critico |
| **Freq PWM GPIO32** | 1 kHz | 100 Hz - 5 kHz | SSR tollera bene, non critico |
| **SSR+ (controllo+)** | +5V sempre | 3-32V DC | Diretto da PSU ATX |
| **SSR- (controllo-)** | Collettore BC547 | 0V (ON) - flottante (OFF) | Commutato da transistor |
| **Fusibile F_HEAT** | 1A Slow Blow 250V | 0.5-1.5A | Protezione contro cortocircuito AC |
| **Termostato meccanico** | 45°C ± 5°C reset manuale | 40-50°C | **ESSENZIALE**, fail-safe |

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo, pronto per realizzazione
