# DISPLAY TOUCH SINGOLO v1.1

## 1. Scelta Display

### Opzione A: ESP32-S3 con Touch Integrato (Consigliato)

```
Modulo: ESP32-S3 Uno + 3.5" Touch ILI9341
Vantaggi:
  - MCU + Display integrati
  - Una sola board
  - LVGL nativo
  - Costo: 25-35 EUR
Svantaggi:
  - Meno versatile se futuro diverso
  - GPIO occupati dal display
```

### Opzione B: ESP32 + Display 3.5" ILI9341 SPI (Attuale progetto)

```
Componenti:
  - ESP32-DevKitC-E32
  - Display 3.5" ILI9341 touch capacitivo
  - Interfaccia SPI

Vantaggi:
  - Massima flessibilità GPIO
  - Separazione MCU/Display
  - Upgrade facile
Svantaggi:
  - Due board separate
  - Cablaggio più lungo
  - Costo: ~35-40 EUR
```

**Scelta progetto:** Opzione B (modularità)

---

## 2. Display ILI9341 3.5" Touch

### Specifiche Display

| Parametro | Valore | Note |
|---|---|---|
| **Risoluzione** | 320 × 480 pixel | VGA + |
| **Dimensione** | 3.5 pollici | ~89 mm diagonale |
| **Profondità colore** | 16-bit 5:6:5 RGB | 65536 colori |
| **Interfaccia** | SPI 4-wire | CLK, MOSI, DC, CS |
| **Touch** | Capacitivo resistivo (opt) | Capacitivo consigliato |
| **Alimentazione** | 3.3V (regolatore integrato) | Consuma ~100mA |
| **Backlight** | LED 12V (opzionale) | Dimming PWM |
| **Connettore** | 40-pin DuPont o JST | Moduli cinesi variano |
| **Tempo risposta** | ~30 ms | Per interfaccia fluid |
| **Prezzo** | 15-25 EUR | AliExpress economico |

### Pinout ILI9341 Tipico

```
Display Touch 3.5" ILI9341:
  VCC ──────── +3.3V (regolatore su board, sopporta fino a 5V in entrata)
  GND ──────── GND
  
LCM (Display):
  CLK  ─────── SPI Clock (GPIO 18 consigliato)
  MOSI ─────── SPI MOSI (GPIO 23 consigliato)
  MISO ─────── SPI MISO (GPIO 19 consigliato)
  CS/TCS ───── Chip Select (GPIO 5 consigliato, OR con touch CS)
  DC ──────── Data/Command (GPIO 4 consigliato)
  RST ──────── Reset (GPIO 17 consigliato, opzionale se legato a EN)
  BL ──────── Backlight PWM (GPIO 16 consigliato, PWM 5kHz)
  
Touch (XPT2046 tipico):
  T_CLK ────── SPI Clock (stesso di display oppure GPIO 18)
  T_MOSI ───── SPI MOSI (stesso di display oppure GPIO 23)
  T_MISO ───── SPI MISO (stesso di display oppure GPIO 19)
  T_CS ────── Touch Chip Select (GPIO 25 consigliato)
  T_INT ────── Interrupt (GPIO 34 input-only, opzionale)
  T_IRQ ────── (a volte etichettato diversamente)
```

---

## 3. Configurazione ESPHome con LVGL

```yaml
# SPI Interface
spi:
  clk_pin: GPIO18
  mosi_pin: GPIO23
  miso_pin: GPIO19

# Display ILI9341
display:
  - platform: ili9341
    data_pins: [GPIO21, GPIO22, GPIO3, GPIO15, GPIO2, GPIO4, GPIO0, GPIO5]
    cs_pin: GPIO32
    dc_pin: GPIO27
    rst_pin: GPIO26
    backlight_pin: GPIO25  # PWM per dimming
    rotation: 0
    id: display_main

# Touch XPT2046
touch_screen:
  - platform: xpt2046
    spi_id: spi_bus
    cs_pin: GPIO33
    interrupt_pin: GPIO34
    calibration:
      x_min: 200
      x_max: 3800
      y_min: 200
      y_max: 3800
    id: touch_main

# LVGL (Light and Versatile Graphics Library)
lvgl:
  display_id: display_main
  touch_id: touch_main
  theme: simple
  pages:
    - id: page_main
      widgets:
        - type: label
          text: "Cella Lievitazione"
          align: center
        - type: gauge
          min: 15
          max: 40
          value: !lambda return id(sht45_temp).state;
          title: "T (°C)"
```

---

## 4. Layout Interfaccia Suggerito

```
┌─────────────────────────────────────┐
│    CELLA LIEVITAZIONE PRO v1.1     │
│                                     │
│  T: 28.5°C    RH: 78%    CO₂: 2400│
│  [████████████░░░░░░]     28/40°C  │
│                                     │
│  Setpoint: [+28°C-] [+28°C+]       │
│  ┌─────────────────────────────────┐│
│  │ Status: Riscaldando              ││
│  │ Ventola: 60% RPM: 1800           ││
│  │ Umidificatore: ON                ││
│  │ LED: ON (luminosità 80%)         ││
│  └─────────────────────────────────┘│
│                                     │
│  [Manuale] [Auto] [Menu] [Power]   │
└─────────────────────────────────────┘
```

---

## 5. Logica Interfaccia Touch

```yaml
binary_sensor:
  - platform: lvgl
    id: btn_power
    obj_id: btn_power_obj
    on_press:
      lambda: |-
        // Accendi/spegni cella
        id(heater_ssr).turn_off();
        id(ventola_ricircolo).turn_off();

  - platform: lvgl
    id: btn_setpoint_up
    obj_id: btn_up_obj
    on_press:
      lambda: |-
        auto current = id(climate_heat).target_temperature;
        id(climate_heat).set_target_temperature(min(current + 1.0, 40.0));

  - platform: lvgl
    id: btn_setpoint_down
    obj_id: btn_down_obj
    on_press:
      lambda: |-
        auto current = id(climate_heat).target_temperature;
        id(climate_heat).set_target_temperature(max(current - 1.0, 15.0));
```

---

## 6. Calibrazione Touch

```yaml
button:
  - platform: template
    name: "Calibra Touch Screen"
    on_press:
      lambda: |-
        ESP_LOGW("TOUCH", "Calibrazione touch in corso...");
        // Eseguire manualmente:
        // 1. Premi 4 angoli dello schermo quando richiesto
        // 2. Registra i valori XPT2046 raw
        // 3. Aggiorna calibration_points in config
```

---

## 7. Consumo Energia Display

| Componente | Consumo |
|---|---|
| ILI9341 display | ~50-80 mA @ 3.3V |
| XPT2046 touch | ~10 mA (polling) |
| Backlight LED 12V | ~100 mA (if used) |
| **TOTALE** | **~160-190 mA** |

→ Contato nel budget +5V (fusibile F2 2A: margin OK)

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo
