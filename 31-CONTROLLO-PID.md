# CONTROLLO PID — TUNING E LOGICA v1.1

## 1. Equazione PID (Riscaldamento)

```
Output = Kp * e(t) + Ki * ∫e(t)dt + Kd * de(t)/dt

Dove:
  e(t) = setpoint - temperatura_attuale
  Kp = guadagno proporzionale
  Ki = guadagno integrale
  Kd = guadagno derivativo
  Output ∈ [0, 1] (0% a 100% PWM → SSR duty cycle)
```

## 2. Parametri di Tuning Iniziali

| Parametro | Valore | Range | Note |
|---|---|---|---|
| **Kp** | 0.5 | 0.3-2.0 | Risposta istantanea |
| **Ki** | 0.02 | 0.0-0.1 | Eliminazione drift (slow) |
| **Kd** | 0.1 | 0.0-0.5 | Damping oscillazioni |
| **Setpoint min** | 15°C | - | Limite minimo sicuro |
| **Setpoint max** | 40°C | - | Limite massimo (termostato 45°C) |
| **Tolleranza** | ±0.5°C | - | Isteresi per evitare switching |

## 3. Tuning Pratico (Ziegler-Nichols Semplificato)

### Fase 1: Tuning Kp senza Ki e Kd

```yaml
climate:
  - platform: pid
    name: "Controllo Temperatura"
    sensor: sht45_temp
    default_target_temperature: 28
    
    heat_output: heater_ssr
    
    control_parameters:
      kp: 1.0    # Inizia alto
      ki: 0.0    # Disabilita integrale
      kd: 0.0    # Disabilita derivativo
```

**Test:**
- Imposta setpoint 28°C
- Osserva il comportamento:
  - Se temperatura oscilla eccessivamente → riduci Kp (0.7, 0.5)
  - Se riscaldamento è troppo lento → aumenta Kp (1.5, 2.0)
  - Obiettivo: raggiungere setpoint in ~5-10 minuti senza oscillare

### Fase 2: Aggiungi Kd per damping

```yaml
control_parameters:
  kp: 0.5     # Dopo tuning fase 1
  ki: 0.0     # Ancora disabilitato
  kd: 0.1     # Aggiungi damping
```

**Test:**
- Osserva se le oscillazioni si riducono
- Se è ancora oscillante → aumenta Kd (0.2, 0.3)
- Se è troppo smorzato → riduci Kd (0.05)

### Fase 3: Aggiungi Ki per eliminare drift

```yaml
control_parameters:
  kp: 0.5
  ki: 0.02    # Integrale piccolo (lento)
  kd: 0.1
```

**Test lungo termine (30+ minuti):**
- Verifica che temperatura rimanga stabile al setpoint
- Se continua a derivare verso il basso → aumenta Ki (0.05, 0.1)
- Se oscilla lentamente → riduci Ki (0.01, 0.005)

---

## 4. Criteri di Valutazione

### Buon Tuning

```
- Tempo di assestamento: 5-15 minuti da setpoint lontano
- Overshoot: < 1°C oltre setpoint
- Errore a regime: < 0.2°C
- Stabilità: oscillazione < 0.1°C
```

### Problemi Comuni

| Sintomo | Causa | Soluzione |
|--------|-------|----------|
| Oscilla continuamente | Kp troppo alto | Riduci Kp |
| Raggiunge setpoint ma continua a salire (overshoot) | Kd insufficiente | Aumenta Kd |
| Deriva lentamente verso il basso | Ki insufficiente | Aumenta Ki |
| Non raggiunge mai setpoint | Potenza riscaldatore insufficiente | Verifica cablaggio/fusibile |
| Salta improvvisamente | Sensore difettoso | Verificare SHT45 letture |

---

## 5. Controllo Offline (senza Home Assistant)

ESPHome PID funziona **completamente offline**. Non necessita HA per operare.

```yaml
climate:
  - platform: pid
    name: "Controllo Temperatura Riscaldamento"
    sensor: sht45_temp
    default_target_temperature: 28  # Default se offline
    min_temperature: 15
    max_temperature: 40
    heat_output: heater_ssr
    control_parameters:
      kp: 0.5
      ki: 0.02
      kd: 0.1
    # Nessun'altra configurazione: opererà autonomamente
```

**Interazione tramite:**
- Home Assistant (se connesso)
- Web server ESPHome locale (http://IP-ESP32/)
- Encoder/pulsanti fisici sulla cella

---

## 6. Integrazione HA per Regolazione Setpoint

```yaml
# In ESPHome:
climate:
  - platform: pid
    id: climate_heat
    # ...

# In Home Assistant (YAML automazione):
automation:
  - alias: "Aumenta setpoint encoder"
    trigger:
      platform: event
      event_type: encoder_clockwise
    action:
      service: climate.set_temperature
      target:
        entity_id: climate.controllo_temperatura_riscaldamento
      data:
        temperature: "{{ states('input_number.setpoint') | float + 0.5 }}"
```

---

## 7. Note Finali

- **Tuning è iterativo**: non è una scienza esatta, richiede osservazione
- **Tempo di risposta sensore**: SHT45 è veloce (~50ms), nessun problema
- **Isteresi**: ESPHome supporta `deadband` per evitare switching continuo
- **Protezione**: termostato meccanico a 45°C agisce indipendentemente dal PID

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo
