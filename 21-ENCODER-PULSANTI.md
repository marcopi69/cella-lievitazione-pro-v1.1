# ENCODER E PULSANTI v1.1

## 1. Encoder Rotativo 360° (Navigazione UI sul display Nextion)

Il display Nextion è touch; l'encoder lo affianca come comando "elastico": il comportamento dipende dalla posizione del cursore sul display (vedi 20-DISPLAY-NEXTION.md §4).

| Stato UI | Rotazione | Click encoder |
|---|---|---|
| Navigazione (`ui_edit=false`) | Sposta il cursore tra i campi | Entra in modifica del campo |
| Modifica (`ui_edit=true`) | Cambia il valore del campo | Conferma ed esce |

Campi selezionabili (`ui_cursor`): 0 = setpoint T, 1 = ventola %, 2 = luci %, 3 = setpoint RH.

### Specifiche

| Parametro | Valore | Note |
|---|---|---|
| **Tipo** | Meccanico 20 pos/rev | Detenti tattili |
| **Risoluzione** | 2 impulsi per click | 20 posizioni totali |
| **Frequenza max** | ~50 Hz (3000 rot/min) | Sufficiente per menu |
| **Alimentazione** | +3.3V | Pull-up interni ESP32 |
| **Corrente** | ~1 mA | Contatti meccanici |
| **Durabilità** | 100.000+ rotazioni | Affidabile |
| **Pinout** | 5-pin: CLK, DT, SW, VCC, GND | Standard |

### Circuito di Collegamento

```
Encoder 5-pin (vista frontale):
  ┌─ CLK (A)  ─── GPIO 18 (con pull-up interno 45kΩ ESP32)
  ├─ DT (B)   ─── GPIO 19 (con pull-up interno 45kΩ ESP32)
  ├─ SW       ─── GPIO 21 (con pull-up interno 45kΩ ESP32)
  ├─ VCC      ─── +3.3V
  └─ GND      ─── GND
  
Schema:
  GPIO 18 (CLK) ────┬─── 10nF (decoupling opzionale) ──── GND
                    │
                    └─ Encoder CLK
                    
  GPIO 19 (DT) ─────┬─── 10nF (decoupling opzionale) ──── GND
                    │
                    └─ Encoder DT
                    
  GPIO 21 (SW) ─────┬─── 10nF (decoupling opzionale) ──── GND
                    │
                    └─ Encoder Switch
```

```yaml
globals:
  - id: ui_cursor
    type: int
    initial_value: "0"
  - id: ui_edit
    type: bool
    initial_value: "false"

sensor:
  - platform: rotary_encoder
    id: setpoint_encoder
    pin_a: GPIO18
    pin_b: GPIO19
    resolution: 2
    on_clockwise:
      - lambda: |-
          if (!id(ui_edit)) { id(ui_cursor) = (id(ui_cursor) + 1) % 4; }
          else {
            switch (id(ui_cursor)) {
              case 0: {  // setpoint T
                auto call = id(climate_heat).make_call();
                call.set_target_temperature(std::min(id(climate_heat).target_temperature + 0.5f, 40.0f));
                call.perform(); break; }
              case 1: {  // ventola
                auto call = id(ventola_ricircolo).make_call();
                call.set_speed(std::min(id(ventola_ricircolo).speed + 5, 100));
                call.perform(); break; }
              case 2: {  // luci
                auto call = id(led_strip).make_call();
                call.set_brightness(std::min(id(led_strip).current_values.get_brightness() + 0.05f, 1.0f));
                call.perform(); break; }
            }
          }
    on_anticlockwise:
      - lambda: |-
          if (!id(ui_edit)) { id(ui_cursor) = (id(ui_cursor) + 3) % 4; }
          else {
            switch (id(ui_cursor)) {
              case 0: {
                auto call = id(climate_heat).make_call();
                call.set_target_temperature(std::max(id(climate_heat).target_temperature - 0.5f, 15.0f));
                call.perform(); break; }
              case 1: {
                auto call = id(ventola_ricircolo).make_call();
                call.set_speed(std::max(id(ventola_ricircolo).speed - 5, 25));
                call.perform(); break; }
              case 2: {
                auto call = id(led_strip).make_call();
                call.set_brightness(std::max(id(led_strip).current_values.get_brightness() - 0.05f, 0.0f));
                call.perform(); break; }
            }
          }

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO21
      mode: INPUT_PULLUP
      inverted: true
    name: "Pulsante Encoder"
    id: btn_encoder
    filters:
      - delayed_on: 10ms
      - delayed_off: 10ms
    on_press:
      - lambda: |-
          id(ui_edit) = !id(ui_edit);
```

---

## 2. Pulsante Multifunzione (GPIO 22)

### Logica Pressione

```
Pressione breve (< 1 s):  Annulla modifica / torna alla Home del display
Pressione lunga (> 2 s):  Toggle umidificatore
```

> La funzione "toggle luci" è passata al pulsante dedicato su GPIO 23 (§2b).

### Circuito

```
GPIO 22 ──────┬─── 10nF (decoupling) ──── GND
              │
              └─ Pulsante NO verso GND
              
Pull-up interno: 45kΩ (ESP32)
```

### Configurazione ESPHome

```yaml
binary_sensor:
  - platform: gpio
    pin: GPIO22
    name: "Pulsante Multifunzione"
    id: btn_multi
    filters:
      - invert:  # NO button, inverted logic
      - delayed_on: 10ms  # Debounce
      - delayed_off: 10ms
    
    on_click:
      # Pressione breve (< 1 s): annulla modifica e vai alla Home
      min_length: 50ms
      max_length: 1000ms
      then:
        - lambda: |-
            id(ui_edit) = false;
            id(ui_cursor) = 0;
            id(nextion_disp).goto_page("home");
    
    on_multi_click:
      # Pressione lunga (> 2 s)
      clicks:
        - min_length: 2000ms
          max_length: 5000ms
          then:
            - logger.log:
                level: INFO
                message: "Umidificatore Toggle"
            - if:
                condition:
                  lambda: return id(humid_pwm).state > 0.01;
                then:
                  - output.turn_off: humid_pwm
                else:
                  - output.turn_on: humid_pwm
```

---

## 2b. Pulsante Luci LED (GPIO 23)

Pulsante dedicato: un tocco accende/spegne la striscia LED, anche senza guardare il display.

```
GPIO 23 ──────┬─── 10nF (decoupling) ──── GND
              │
              └─ Pulsante NO verso GND

Pull-up interno: 45kΩ (ESP32)
```

```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO23
      mode: INPUT_PULLUP
      inverted: true
    name: "Pulsante Luci LED"
    id: btn_luci
    filters:
      - delayed_on: 10ms
      - delayed_off: 10ms
    on_press:
      - light.toggle: led_strip
```

---

## 3. Pulsante Reset (Hardware EN)

### Scopo

Reset hardware ESP32 (riavvio completo).

### Circuito

```
ESP32 EN pin ──┬─── Pulsante NO verso GND
               │
               └─── 100nF (debounce) ──── GND
               
Pull-up interno: ESP32 ha ~10-15kΩ interno verso 3.3V
```

### Comportamento

```
- Pressione breve (~100ms): Riavvio ESP32
- Tenuto premuto per 3+ s: Alcuni moduli accedono a bootloader (raro)
```

---

## 4. LED di Stato (GPIO 16)

### Indicazioni Luminose

```
Stato Sistema         | Lampeggio LEd
──────────────────────┼──────────────────────
Operativo normale     | Lampeggio lento (1 Hz) verde
Riscaldamento attivo  | Continuo acceso arancio/giallo
Raffreddamento        | Continuo acceso blu (se compressore)
Allarme T > 45°C      | Lampeggio veloce (5 Hz) rosso
Allarme RH > 95%      | Lampeggio veloce (3 Hz) arancio
Allarme CO₂ > 4500ppm| Lampeggio lento rosso
Errore sensore        | Lampeggio saltuario + log
Offline (no Wi-Fi)    | Lampeggio lentissimo (0.5 Hz)
```

### Configurazione ESPHome

```yaml
output:
  - platform: ledc
    pin: GPIO16
    frequency: 1000Hz
    min_power: 0.0
    max_power: 1.0
    id: led_status
    power_supply: !extend
      auto_on: true

light:
  - platform: monochromatic
    output: led_status
    name: "LED Stato"
    id: led_status_light

automation:
  # Lampeggio normale operativo
  - trigger: time_interval
    interval: 1s
    then:
      - if:
          condition: api.connected  # Se connesso a HA
          then:
            - light.turn_on:
                id: led_status_light
                brightness: 0.5
      - delay: 500ms
      - light.turn_off: led_status_light
  
  # Allarme T max
  - trigger:
      platform: numeric_state
      entity_id: sensor.sht45_temperature
      above: 45
    then:
      # Lampeggio veloce rosso
      - repeat:
          count: 10
          then:
            - light.turn_on:
                id: led_status_light
                brightness: 1.0
                red: 1.0
                green: 0.0
                blue: 0.0
            - delay: 100ms
            - light.turn_off: led_status_light
            - delay: 100ms
```

---

## 5. Matrice Ingressi Finale

| GPIO | Funzione | Tipo | Segnale | Pull-up |
|------|----------|------|---------|----------|
| 18 | Encoder CLK | IN | 0/3.3V | Interno 45kΩ |
| 19 | Encoder DT | IN | 0/3.3V | Interno 45kΩ |
| 21 | Encoder SW | IN | 0/3.3V | Interno 45kΩ |
| 22 | Pulsante MF | IN | 0/3.3V | Interno 45kΩ |
| 23 | Pulsante luci LED | IN | 0/3.3V | Interno 45kΩ |
| 14 | UART2 TX (Display) | OUT | UART 3.3V | - |
| 27 | UART2 RX (Display) | IN | UART 3.3V | - |
| 34 | Ventola TACH | IN | 0/3.3V | No (input-only) |
| 16 | LED di stato | OUT | 0-3.3V PWM | - |
| EN | Reset HW | IN | 0/3.3V | Interno 10-15kΩ |

---

## 6. Debounce e Filtering

```yaml
# Consigliato per pulsanti meccanici
filters:
  - invert:  # Se sono active-low (normalmente chiusi)
  - delayed_on: 10ms
  - delayed_off: 10ms
  - delayed_on_off: 10ms
```

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo
