# VENTOLA 4-PIN PWM + TACHIMETRO v1.1

## 1. Ventola 4-Pin vs 2-Pin vs 3-Pin

| Configurazione | Fili | Controllo | Feedback | Vantaggi | Svantaggi |
|---|---|---|---|---|---|
| **2-Pin** | +12V, GND | ON/OFF o PWM low-side | NO | Economica | No feedback RPM, velocità fissa PWM |
| **3-Pin** | +12V, GND, TACH | ON/OFF o PWM low-side | Sì (TACH) | Feedback RPM | No PWM dedicato, velocità tramite PWM alimentazione |
| **4-Pin** ✅ | +12V, GND, PWM, TACH | PWM dedicato 25kHz | Sì (TACH) | **PWM pulito, feedback RPM** | **Leggermente più costosa** |

### Scelta Progetto: **Ventola 4-Pin**

Motivi:
- Controllo PWM **dedicato** (pin 3), separato dall'alimentazione
- RPM feedback (pin 4 TACH) per monitoraggio affidabilità
- Commutazione pulita, senza interferenze EMI
- Costo minimo differenziale rispetto a 3-pin

---

## 2. Pinout Ventola 4-Pin Standard

```
Ventola 4-Pin vista da davanti (verso la griglia):

  ┌─────────────────┐
  │  ◯ ◯ ◯ ◯        │
  │  1 2 3 4        │
  └─────────────────┘
  
Pin 1: +12V DC rosso  (alimentazione positiva)
Pin 2: GND nero       (massa)
Pin 3: PWM verde      (ingresso PWM 25 kHz, 5V logic)
Pin 4: TACH giallo    (uscita open-collector, ~5V pull-up interno ventola)
```

### Specifiche Alimentazione

- **+12V:** Sempre presente, alimentazione motore
- **GND:** Sempre presente
- **PWM in:** Ingresso PWM a 25 kHz, livello 5V (tollerante a 3.3V)
- **TACH out:** Open-collector, 2 impulsi per giro (2 magneti rotore)

---

## 3. Circuito Controllo PWM (GPIO 25)

### Schema Semplificato

```
ESP32 GPIO 25 (PWM output, 3.3V) ──┬──── R 220Ω ──────┬──── Ventola PWM (pin 3)
                                   │                  │
                              Gate driver       Capacitor bypass
                                   │              (opzionale, 100nF)
                                   │
                              Pull-down optional
                              (no se ventola ha pull-down interno)
```

### Configurazione ESPHome PWM

```yaml
output:
  - platform: ledc
    pin: GPIO25
    frequency: 25000Hz        # 25 kHz standard per ventole PC
    min_power: 0.0            # 0% = OFF
    max_power: 1.0            # 100% = velocità massima
    id: fan_pwm
    
fan:
  - platform: speed
    output: fan_pwm
    name: "Ventola Ricircolo Cella"
    id: ventola_ricircolo
    
    # Velocità minima per mantenere rotazione
    min_power: 0.25           # 25% PWM minimo per evitare stallo
```

---

## 4. Circuito Tachimetro (GPIO 34)

### Segnale TACH Ventola

```
Ventola ha: Open-collector TACH con pull-up interno ~10-20kΩ a +12V
           1 impulso per mezzo giro = 2 impulsi per giro completo
           
Frequenza @ 2000 RPM: f = (2000 rpm / 60 s) × 2 impulsi = ~67 Hz
Frequenza @ 4000 RPM: f = (4000 rpm / 60 s) × 2 impulsi = ~133 Hz
```

### Partitore di Livello (adattamento 12V → 3.3V)

La ventola emette a ~12V (con pull-up interno a 12V), ma ESP32 GPIO accetta max 3.6V.

**Soluzione: Partitore resistivo**

```
Ventola TACH (giallo, ~12V open-collector) ──┬──── R 10kΩ ──────┬──── GND
                                              │                  │
                                              └──── R 4.7kΩ ─────┼──── GPIO 34 (3.3V)
                                                                 │
                                              Tensione divisore: Vout = 12V × 4.7k / (10k + 4.7k)
                                                         Vout ≈ 3.7V (appena sopra 3.3V max)
                                                         
⚠️ NOTA: 3.7V è leggermente sopra 3.6V max tollerato.
Migliore: Usare 15kΩ + 4.7kΩ per Vout = 2.9V
```

### Versione Robusta (con Optoisolatore)

Se vuoi isolamento galvanico completo (consigliato in ambienti con molto rumore EMI):

```
Ventola TACH ──────────┬──── R 1kΩ ────┬──── Anodo LED Optoisolatore (es. 4N25)
                       │               │
                    Pull-up        Interno
                  (ventola ha)      optoisolatore
                       │               │
                       └───── GND ─────┘

Emettitore optoisolatore ──┬──── R 10kΩ ──── +3.3V (pull-up)
                            │
                          Base driver
                            │
                      Collettore ──── GPIO 34
```

### Configurazione ESPHome Tachimetro

```yaml
sensor:
  - platform: pulse_counter
    pin: GPIO34
    name: "Ventola RPM"
    id: ventola_rpm
    unit_of_measurement: "RPM"
    accuracy_decimals: 0
    update_interval: 5s
    
    # Conversione: 2 impulsi per giro
    filters:
      - multiply: 0.5       # Perché pulse_counter conta impulsi, non giri
```

**Calcolo RPM:**
```
Pulse counter @ GPIO34 legge impulsi al secondo (Hz)
Ventola: 2 impulsi per giro

RPM = (impulsi_per_secondo / 2) × 60
    = (Hz / 2) × 60
    = Hz × 30
    
ESPHome `multiply: 0.5` trasforma:
  pulse_count (impulsi in 5s) → impulsi_al_secondo
  impulsi_al_secondo × 0.5 × 60 = RPM
  
⚠️ Verificare con datasheet ventola specifica (alcuni hanno 1 impulso/giro)
```

---

## 5. Circuito Completo Ventola 4-Pin

```
+12V_SYS (da PSU ATX) ──────┬─────────────────────────────────────┬──── Ventola +12V (rosso)
                            │                                     │
                       Bulk cap                                   │
                      (470µF già                                  │
                       montato                                    │
                       per +12V)                                  │
                            │                                     │
╔═══════════════════════════╩═════════════════════════════════════╩═════════════════════════════╗
║                          DRIVER MOSFET VENTOLA                                               ║
╚═════════════════════════════════════════════════════════════════════════════════════════════════╝

                            │
                       ┌────┴────┐
                       │ Fly-back │  1N5819 Schottky
                       │  Diode   │  (Anodo→Drain, Catodo→+12V)
                       │ 1N5819   │
                       └────┬────┘
                            │
                    ┌───────┴───────┐
                    │   Q1 IRLZ44N  │
                    │  (Drain here) │
                    │               │
 GPIO 25 (PWM) ────┼─── R 220Ω ────┼──── Gate
  25 kHz            │               │
  0-3.3V            │               │   (Source)
                    │               ├──── GND via R 10kΩ
                    │               │
                    │               │
                    └───────┬───────┘
                            │
                        GND_SYS


╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║                        SEGNALE TACHIMETRO (FEEDBACK RPM)                                       ║
╚═════════════════════════════════════════════════════════════════════════════════════════════════╝

Ventola TACH (giallo) ──┬───── R 15kΩ ─────┬──── GND
  Open-collector        │                  │
  ~12V pull-up          └──── R 4.7kΩ ─────┼──── GPIO 34 (Pulse Counter)
  interno               Partitore           │     Livello ~3V IN
                        2.9V OUT            │
                                        Decoupling
                                        100nF (opz.)
```

---

## 6. Specifiche Ventola e Calcolo RPM

### Ventola Tipica 60mm 4-Pin (Esempio: Noctua, be quiet!, Corsair)

| Parametro | Valore Tipico | Range |
|---|---|---|
| Voltaggio | 12 V DC | 10-13.2 V |
| Corrente @ nominal | 0.3 A | 0.15-0.5 A |
| RPM nominal | 2000 RPM | 1000-4000 RPM |
| PWM input | 25 kHz | 20-30 kHz |
| PWM min valid | 25% | 10-30% |
| Pressione statica | 1-3 mmH2O | Varia |
| Noise | 15-25 dB | Silent a low speed |
| TACH impulsi | 2 per rotazione | 1-2 (verificare datasheet) |
| Segnale TACH | Open-collector 12V | Up to ~5V |

### Calcolo Velocità Ventola

```
Ventola 60mm @ 2000 RPM nominale con 2 impulsi/giro

@ PWM 50% (velocità media ~1000 RPM):
  Frequenza TACH = (1000 rpm / 60) × 2 impulsi = 33.3 Hz
  Conteggio pulse_counter in 5 secondi = 33.3 × 5 = ~167 impulsi
  ESPHome calcola: 167 × multiply(0.5) = 83.5 RPM ... NO! ❌
  
  Correzione: multiply deve essere × 30 (non 0.5)
  (impulsi in 5s) / 5 = Hz
  Hz / 2 impulsi = giri/s
  giri/s × 60 = giri/min = RPM
  
  Quindi: [impulsi in 5s] × (1/5) × (1/2) × 60 = [impulsi in 5s] × 6
  
  ESPHome multiply: 6
```

### Configurazione Corretta ESPHome

```yaml
sensor:
  - platform: pulse_counter
    pin: GPIO34
    name: "Ventola RPM"
    id: ventola_rpm
    unit_of_measurement: "RPM"
    accuracy_decimals: 0
    update_interval: 5s
    count_mode:
      rising: INCREMENT   # Conta ogni rising edge (impulsi)
    
    filters:
      - multiply: 6       # Converti impulsi_5s → RPM
      # Alternativa se non vuoi il filtro:
      # - lambda: return x * 6;  // impulsi in 5s → RPM
```

---

## 7. Logica PID Ventola (Controllo Velocità)

### Caso 1: Controllo PWM Proporzionale a RH

```yaml
climate:
  # Umidificatore già controllato separatamente
  # Ventola aumenta con RH per favorire ricircolo
  
automation:
  - trigger:
      platform: numeric_state
      entity_id: sensor.sht45_humidity
      above: 70
    action:
      - fan.turn_on: ventola_ricircolo
      - fan.set_speed:
          id: ventola_ricircolo
          speed: !lambda |
            # RH 70% → 50% PWM
            # RH 85% → 100% PWM
            auto rh = id(sht45_humidity).state;
            auto speed = (rh - 70) / (85 - 70);  // Normalizza 0-1
            return constrain(speed, 0.0, 1.0);
```

### Caso 2: Controllo PWM Minimo per Omogeneità

```yaml
switch:
  - platform: template
    name: "Ventola Ricircolo (SIMPLE)"
    lambda: |-
      return id(ventola_ricircolo).state > 0.0;
    turn_on_action:
      - fan.set_speed:
          id: ventola_ricircolo
          speed: 0.8  # 80% sempre (compromesso:
    turn_off_action:              # rumore vs omogeneità)
      - fan.turn_off: ventola_ricircolo
```

---

## 8. Monitoraggio e Allarmi

### Allarme Ventola Ferma (TACH = 0)

```yaml
automation:
  - trigger:
      platform: numeric_state
      entity_id: sensor.ventola_rpm
      below: 100
      for: 10s
    condition:
      # Solo se ventola dovrebbe essere ON (PWM > 25%)
      template: "{{ state_attr('fan.ventola_ricircolo', 'percentage') > 25 }}"
    action:
      - logger.log:
          level: WARNING
          message: "Ventola ferma! Verificare connessione/alimentazione"
      - notify.send:
          message: "🚨 Ventola ricircolo cella non gira. Controllare immediatamente!"
```

### Dashboard Home Assistant

```yaml
entity:
  - type: gauge
    entity: sensor.ventola_rpm
    title: "RPM Ventola"
    min: 0
    max: 4000
    
  - type: button
    entity: fan.ventola_ricircolo
    tap_action:
      action: call-service
      service: fan.set_percentage
      service_data:
        entity_id: fan.ventola_ricircolo
        percentage: !if
          - condition: state
            entity_id: fan.ventola_ricircolo
            state: "on"
          then: 0    # OFF
          else: 100  # 100%
```

---

## 9. Checklist Montaggio e Test

### Montaggio PCB
- [ ] Saldare R 220Ω tra GPIO25 e Gate MOSFET Q1
- [ ] Saldare R 10kΩ pull-down tra Gate e GND
- [ ] Saldare 1N5819 fly-back: Anodo→Drain Q1, Catodo→+12V
- [ ] Saldare R 15kΩ e R 4.7kΩ partitore TACH (oppure R 10kΩ + 10kΩ se 3V limite rigoroso)
- [ ] Saldare 100nF decoupling su TACH (opzionale, consigliato)
- [ ] Connettore ventola 4-pin JST o Molex:
  - [ ] Rosso +12V → Linea +12V PSU
  - [ ] Nero GND → Massa PCB
  - [ ] Verde PWM → GPIO 25 tramite R 220Ω
  - [ ] Giallo TACH → Partitore R 15k/4.7k → GPIO 34

### Test Software
- [ ] Carica firmware ESPHome con fan e pulse_counter
- [ ] Apri console Serial, dai comando: `fan.turn_on ventola_ricircolo`
- [ ] Verifiche GPIO25 con multimetro: deve visualizzare PWM ~50% (default)
- [ ] Accendi ventola (dovrebbe girare se PWM > 25%)
- [ ] Verifica console: sensor.ventola_rpm deve mostrare RPM approssimativo
- [ ] Comanda: `fan.set_percentage ventola_ricircolo 100` → Ventola max speed
- [ ] RPM deve aumentare, ad es. da 1500 a 3000+ RPM
- [ ] Comanda: `fan.turn_off ventola_ricircolo` → Ventola OFF, RPM → 0

### Calibrazione Moltiplicatore RPM
- [ ] Accendi ventola al 50% PWM
- [ ] Conta manualmente i giri per 10 secondi con tachimetro ottico oppure
- [ ] Usa formazione: RPM_display × 2 / 60 = impulsi/secondo
- [ ] Se RPM_display non corrisponde, regola multiply: nuova_mult = old_mult × (RPM_atteso / RPM_display)

---

## 10. Troubleshooting

| Sintomo | Causa Probabile | Soluzione |
|--------|---|---|
| Ventola non gira | PWM non presente su pin 3 | Verificare GPIO25, cablaggio, R 220Ω |
| Ventola gira ma lentamente | PWM troppo basso | Aumentare percentuale, verificare min_power |
| RPM = 0 sempre | TACH non collegato/rotto | Verificare GPIO34, partitore R, connessione |
| RPM troppo alto (falso) | Multiply sbagliato | Ricalibrare, verificare datasheet ventola |
| Rumore PWM (fischio) | Frequenza 25 kHz troppo bassa per MOSFET | Aumentare a 50 kHz se ventola lo tollera |
| Ventola non si accende sotto 30% PWM | Ventola ha min_power interno | Normale, verificare datasheet ventola |

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo, pronto per realizzazione
