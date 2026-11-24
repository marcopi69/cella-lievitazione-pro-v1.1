# ESPHOME CONFIGURAZIONE COMPLETA v1.1

```yaml
# Cella di Lievitazione PRO v1.1
# Configurazione ESPHome completa
# Data: Ottobre 2026

esphome:
  name: cella-lievitazione
  friendly_name: "Cella Lievitazione PRO"
  project:
    name: "marcopi69.cella-lievitazione-pro"
    version: "1.1"
  min_version: 2024.6.0

esp32:
  board: esp32dev
  framework:
    type: esp-idf
    version: recommended

# ===== WIFI E API =====
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "Cella-Lievitazione-AP"
    password: "12345678"
  fast_connect: true

api:
  encryption:
    key: !secret api_encryption_key
  reboot_timeout: 15min

ota:
  password: !secret ota_password

logger:
  level: INFO
  logs:
    dallas: DEBUG
    scd4x: DEBUG

# ===== WEB SERVER =====
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password

# ===== UART + DISPLAY NEXTION/TJC 5" (touch) =====
# Dettagli e mappa componenti: 20-DISPLAY-NEXTION.md
uart:
  id: uart_nextion
  tx_pin: GPIO14
  rx_pin: GPIO27
  baud_rate: 115200

globals:
  - id: ui_cursor   # campo evidenziato dall'encoder (0-3)
    type: int
    initial_value: "0"
  - id: ui_edit     # true = l'encoder modifica il valore
    type: bool
    initial_value: "false"

display:
  - platform: nextion
    id: nextion_disp
    uart_id: uart_nextion
    update_interval: 1s
    start_up_page: 0
    lambda: |-
      it.set_component_text_printf("t_temp", "%.1f", id(sht45_temp).state);
      it.set_component_text_printf("t_rh",   "%.0f", id(sht45_rh).state);
      it.set_component_text_printf("t_co2",  "%.0f", id(scd41_co2).state);
      it.set_component_text_printf("t_set",  "%.1f", id(climate_heat).target_temperature);
      for (int i = 0; i < 4; i++) {
        it.set_component_background_color(
          ("b_sel" + std::to_string(i)).c_str(),
          (i == id(ui_cursor)) ? (id(ui_edit) ? "RED" : "YELLOW") : "BLACK");
      }

# ===== I2C BUS =====
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
  frequency: 100kHz
  id: bus_a

# ===== 1-WIRE BUS =====
dallas:
  pin: GPIO13
  update_interval: 10s
  id: dallas_bus

# ===== I2C MULTIPLEXER (PCA9548A) =====
tca9548a:
  i2c_id: bus_a
  address: 0x70
  id: pca_multiplexer
  channels:
    - bus_id: mux_ch3
      channel: 3
    - bus_id: mux_ch4
      channel: 4

# ===== SENSORI T/RH/CO2 =====
sensor:
  # SHT45 Canale 3
  - platform: sht4x
    i2c_id: mux_ch3
    address: 0x44
    update_interval: 10s
    
    temperature:
      name: "Temperatura Cella (SHT45)"
      id: sht45_temp
      unit_of_measurement: "°C"
      accuracy_decimals: 1
      filters:
        - offset: 0.0  # Calibrazione
    
    humidity:
      name: "Umidità Relativa (SHT45)"
      id: sht45_rh
      unit_of_measurement: "%"
      accuracy_decimals: 1
      filters:
        - offset: 0.0
  
  # SCD41 Canale 4
  - platform: scd4x
    i2c_id: mux_ch4
    address: 0x62
    update_interval: 10s
    
    co2:
      name: "CO₂ Cella (SCD41)"
      id: scd41_co2
      unit_of_measurement: "ppm"
      accuracy_decimals: 0
    
    temperature:
      name: "Temperatura SCD41 (compensazione)"
      id: scd41_temp
      unit_of_measurement: "°C"
      accuracy_decimals: 1
    
    humidity:
      name: "Umidità SCD41 (compensazione)"
      id: scd41_rh
      unit_of_measurement: "%"
      accuracy_decimals: 1
  
  # DS18B20 Sonde 1-Wire
  - platform: dallas
    dallas_id: dallas_bus
    address: 0x123456789ABCDEF0  # Scoprire al primo run
    name: "Temperatura DS18B20 #1"
    id: ds18b20_1
    unit_of_measurement: "°C"
    accuracy_decimals: 1
  
  - platform: dallas
    dallas_id: dallas_bus
    address: 0xFEDCBA9876543210  # Scoprire al primo run
    name: "Temperatura DS18B20 #2 (Backup)"
    id: ds18b20_2
    unit_of_measurement: "°C"
    accuracy_decimals: 1
  
  # Ventola RPM
  - platform: pulse_counter
    pin: GPIO34
    name: "Ventola RPM"
    id: ventola_rpm
    unit_of_measurement: "RPM"
    accuracy_decimals: 0
    update_interval: 5s
    filters:
      - multiply: 6  # Converti impulsi_5s a RPM (2 impulsi/giro)

  # Encoder rotativo: cursore/valore UI. Versione completa (modifica valori)
  # in 21-ENCODER-PULSANTI.md: sostituire questo blocco con quello.
  - platform: rotary_encoder
    id: ui_encoder
    pin_a: GPIO18
    pin_b: GPIO19
    resolution: 2
    on_clockwise:
      - lambda: |-
          if (!id(ui_edit)) { id(ui_cursor) = (id(ui_cursor) + 1) % 4; }
    on_anticlockwise:
      - lambda: |-
          if (!id(ui_edit)) { id(ui_cursor) = (id(ui_cursor) + 3) % 4; }

# ===== OUTPUT PWM/DIGITALI =====
output:
  # GPIO 32: SSR Riscaldatore (PID output)
  - platform: ledc
    pin: GPIO32
    frequency: 1000Hz
    min_power: 0.0
    max_power: 1.0
    id: heater_ssr
  
  # GPIO 33: LED Strip PWM
  - platform: ledc
    pin: GPIO33
    frequency: 25000Hz
    min_power: 0.0
    max_power: 1.0
    id: led_pwm
  
  # GPIO 25: Ventola PWM
  - platform: ledc
    pin: GPIO25
    frequency: 25000Hz
    min_power: 0.0
    max_power: 1.0
    id: fan_pwm
  
  # GPIO 26: Umidificatore ON/OFF
  - platform: ledc
    pin: GPIO26
    frequency: 1000Hz
    min_power: 0.0
    max_power: 1.0
    id: humid_pwm
  


# ===== CLIMATE CONTROLLER (PID RISCALDAMENTO) =====
climate:
  - platform: pid
    name: "Controllo Temperatura Riscaldamento"
    sensor: sht45_temp
    default_target_temperature: 28
    min_temperature: 15
    max_temperature: 40
    
    heat_output: heater_ssr
    
    control_parameters:
      kp: 0.5      # Proporzionale
      ki: 0.02     # Integrale (lento, no drift)
      kd: 0.1      # Derivativo (damping)
    
    visual:
      - min: 15
        max: 40
    
    id: climate_heat

# ===== VENTOLA (SPEED CONTROL) =====
fan:
  - platform: speed
    output: fan_pwm
    name: "Ventola Ricircolo"
    id: ventola_ricircolo
    min_power: 0.25  # Minimo 25% per mantenere rotazione

# ===== LIGHT (LED STRIP) =====
light:
  - platform: monochromatic
    output: led_pwm
    name: "LED Strip Illuminazione"
    id: led_strip
    restore_mode: RESTORE_DEFAULT_OFF

  # GPIO 16: LED RGB di stato (WS2812, 1 filo dati)
  - platform: esp32_rmt_led_strip
    id: led_status_light
    name: "LED Stato"
    internal: true
    pin: GPIO16
    num_leds: 1
    chipset: WS2812
    rgb_order: GRB
    default_transition_length: 0s

# ===== BINARY SENSORS (PULSANTI E TOUCH) =====

# ===== BINARY SENSORS (PULSANTI E TOUCH) ===== (PULSANTI E TOUCH) =====binary_sensor:
  # Eventi touch Nextion (component_id da verificare nell'editor)
  - platform: nextion
    page_id: 0
    component_id: 10          # b_led
    name: "Touch Luci"
    on_press:
      - light.toggle: led_strip

  - platform: nextion
    page_id: 0
    component_id: 11          # b_hum
    name: "Touch Umidificatore"
    on_press:
      - switch.toggle: switch_humid

  # Pulsante luci LED dedicato (GPIO 23)
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

  # Encoder Switch: entra/esce dalla modifica del campo
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
  
  # Pulsante Multifunzione
  - platform: gpio
    pin: GPIO22
    name: "Pulsante Multifunzione"
    id: btn_multi
    filters:
      - invert:
      - delayed_on: 10ms
      - delayed_off: 10ms
    
    # Pressione breve: annulla modifica e torna alla Home
    on_click:
      min_length: 50ms
      max_length: 1000ms
      then:
        - lambda: |-
            id(ui_edit) = false;
            id(ui_cursor) = 0;
            id(nextion_disp).goto_page("home");
    
    # Pressione lunga: Toggle Umidificatore
    on_multi_click:
      clicks:
        - min_length: 2000ms
          max_length: 5000ms
          then:
            - logger.log:
                level: INFO
                message: "Umidificatore Toggle (long press)"
            - if:
                condition:
                  lambda: return id(humid_pwm).state > 0.01;
                then:
                  - output.turn_off: humid_pwm
                else:
                  - output.turn_on: humid_pwm
  
  # API Connected status
  - platform: api
    name: "API Connected"
    id: api_connected

# ===== SWITCH (ON/OFF CONTROLS) =====
switch:
  # Umidificatore ON/OFF semplice
  - platform: template
    name: "Umidificatore ON/OFF"
    id: switch_humid
    lambda: |-
      return id(humid_pwm).state > 0.01;
    turn_on_action:
      - output.set_level:
          id: humid_pwm
          level: 0.8
    turn_off_action:
      - output.set_level:
          id: humid_pwm
          level: 0.0
  
  # Riscaldatore ON/OFF forzato (testing)
  - platform: template
    name: "Riscaldatore Forzato (DEBUG)"
    id: switch_heater_manual
    lambda: |-
      return id(heater_ssr).state > 0.01;
    turn_on_action:
      - output.set_level:
          id: heater_ssr
          level: 1.0
    turn_off_action:
      - output.set_level:
          id: heater_ssr
          level: 0.0

# (LED di stato: uscita PWM `led_status` su GPIO16, vedi sezione output)

# ===== AUTOMAZIONI =====
automation:
  # Allarme: T > 45°C
  - trigger:
      platform: numeric_state
      entity_id: sensor.sht45_temperature
      above: 45
      for: 5s
    action:
      - logger.log:
          level: ERROR
          message: "ALLARME TEMPERATURA CRITICA > 45°C! Riscaldatore OFF!"
      - output.turn_off: heater_ssr
      - light.turn_on:
          id: led_strip
          brightness: 1.0
  
  # Allarme: RH > 95%
  - trigger:
      platform: numeric_state
      entity_id: sensor.sht45_humidity
      above: 95
      for: 5s
    action:
      - logger.log:
          level: WARNING
          message: "Umidità CRITICA > 95%! Umidificatore OFF!"
      - output.turn_off: humid_pwm
  
  # Allarme: CO2 > 4500 ppm
  - trigger:
      platform: numeric_state
      entity_id: sensor.scd41_co2
      above: 4500
      for: 10s
    action:
      - logger.log:
          level: WARNING
          message: "CO₂ ALTA > 4500 ppm! Ventola al 100%!"
      - fan.turn_on: ventola_ricircolo
      - fan.set_speed:
          id: ventola_ricircolo
          speed: 100
  
  # LED RGB di stato (aggiornato ogni 100 ms)
  - trigger:
      platform: time_interval
      interval: 100ms
    action:
      - lambda: |-
          uint32_t t = millis();
          // blink(periodo_ms): vero nella prima meta' del periodo
          auto blink = [t](uint32_t p) { return (t % p) < (p / 2); };
          float r = 0, g = 0, b = 0;
          float T = id(sht45_temp).state, RH = id(sht45_rh).state, C = id(scd41_co2).state;
          if (std::isnan(T) || std::isnan(RH)) {          // errore sensore: viola
            if (blink(1000)) { r = 1; b = 1; }
          } else if (T > 45) {                            // allarme T: rosso veloce
            if (blink(200)) r = 1;
          } else if (RH > 95) {                           // allarme UR: arancio veloce
            if (blink(330)) { r = 1; g = 0.35; }
          } else if (!std::isnan(C) && C > 4500) {        // allarme CO2: rosso lento
            if (blink(1500)) r = 1;
          } else if (!global_api_server->is_connected()) { // offline: blu lentissimo
            if (blink(4000)) b = 1;
          } else if (id(climate_heat).action == CLIMATE_ACTION_HEATING) {
            r = 1; g = 0.35;                              // riscaldamento: arancio fisso
          } else {                                        // normale: verde lampeggio 1 Hz
            if (blink(1000)) g = 1;
          }
          auto call = id(led_status_light).make_call();
          if (r + g + b > 0) {
            call.set_state(true);
            call.set_rgb(r, g, b);
            call.set_brightness(0.3f);   // 30 %: non abbaglia di notte
          } else {
            call.set_state(false);
          }
          call.perform();

# ===== BUTTON (COMANDI MANUALI) =====
button:
  # Restart ESP32
  - platform: restart
    name: "Restart Sistema"
  
  # Safe mode
  - platform: safe_mode
    name: "Safe Mode"

# ===== TIME (SINCRONIZZAZIONE) =====
time:
  - platform: homeassistant
    id: homeassistant_time

# ===== TEXT SENSOR =====
text_sensor:
  # Versione firmware
  - platform: template
    name: "Versione Firmware"
    lambda: |-
      return {"1.1"};
  
  # Stato ultimo aggiornamento
  - platform: wifi_info
    ip_address:
      name: "IP Address"
    ssid:
      name: "SSID Connesso"
    bssid:
      name: "BSSID"
    scan_results:
      name: "WiFi Scan Results"
```

---

**Versione:** 1.1  
**Data:** Ottobre 2026  
**Status:** ✅ Completo, pronto per ESPHome Web
