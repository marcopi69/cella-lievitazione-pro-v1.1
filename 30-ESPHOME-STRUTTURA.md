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

# ===== SENSORI T/RH/CO2 =====
sensor:
  # SHT45 Canale 3
  - platform: sht4x
    i2c_id: pca_multiplexer
    i2c_address: 0x44
    channel: 3
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
    i2c_id: pca_multiplexer
    i2c_address: 0x62
    channel: 4
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
  
  # GPIO 16: LED di stato
  - platform: ledc
    pin: GPIO16
    frequency: 1000Hz
    min_power: 0.0
    max_power: 1.0
    id: led_status

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

# ===== BINARY SENSORS (PULSANTI) =====
binary_sensor:
  # Encoder CLK
  - platform: gpio
    pin: GPIO18
    name: "Encoder CLK"
    id: enc_clk
    internal: true
  
  # Encoder DT
  - platform: gpio
    pin: GPIO19
    name: "Encoder DT"
    id: enc_dt
    internal: true
  
  # Encoder Switch
  - platform: gpio
    pin: GPIO21
    name: "Pulsante Encoder"
    id: btn_encoder
    filters:
      - delayed_on: 10ms
      - delayed_off: 10ms
    on_press:
      lambda: |-
        ESP_LOGD("UI", "Encoder button pressed");
  
  # Pulsante Multifunzione
  - platform: gpio
    pin: GPIO22
    name: "Pulsante Multifunzione"
    id: btn_multi
    filters:
      - invert:
      - delayed_on: 10ms
      - delayed_off: 10ms
    
    # Pressione breve: Toggle LED
    on_click:
      min_length: 50ms
      max_length: 1000ms
      then:
        - logger.log:
            level: INFO
            message: "LED Toggle (short press)"
        - if:
            condition:
              lambda: return id(led_pwm).state > 0.01;
            then:
              - output.turn_off: led_pwm
            else:
              - output.turn_on: led_pwm
    
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

# ===== STATUS LED =====
status_led:
  pin:
    number: GPIO16
    inverted: false

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
  
  # Lampeggio LED operativo (1 Hz)
  - trigger:
      platform: time_interval
      interval: 1s
    action:
      - if:
          condition: api.connected
          then:
            - output.set_level:
                id: led_status
                level: 0.5
            - delay: 500ms
            - output.set_level:
                id: led_status
                level: 0.0

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
