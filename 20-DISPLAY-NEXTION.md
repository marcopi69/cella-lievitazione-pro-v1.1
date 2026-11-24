# DISPLAY NEXTION / TJC 5" (UART) v1.1

> Sostituisce il display SPI ILI9341 + XPT2046 + LVGL delle versioni precedenti.
> **Costo in GPIO: solo 2 (UART2 TX/RX).** Il touch è interno al display e viaggia sulla stessa UART.

## 1. Scelta Display

| Parametro | Valore | Note |
|---|---|---|
| **Famiglia** | Nextion Basic/Enhanced oppure TJC (compatibile) | Scegliere 5" (800×480) |
| **Interfaccia verso ESP32** | UART 3.3 V TTL, 115200 baud | 2 fili + alimentazione |
| **Touch** | Resistivo (Basic) o capacitivo (Enhanced/Intelligent) | **Nessuna differenza di cablaggio e firmware** |
| **Alimentazione** | +5 V | ~250 mA tipici, ~500-600 mA picco (retroilluminazione 100 %) |
| **Controller / GUI** | Nel display (CPU, RAM e flash proprie) | Zero framebuffer sull'ESP32 |
| **Editor** | Nextion Editor (Nextion) / USART HMI (TJC) | Il file `.tft` si progetta fuori da ESPHome |
| **Prezzo indicativo** | 35-70 EUR | Capacitivo/Enhanced più caro |

**Resistivo vs capacitivo:** cambia solo la sensazione al tatto (il capacitivo è più sensibile e funziona con un tocco leggero; il resistivo richiede pressione e funziona con guanti o dita infarinate). Hardware e software sono identici.

**Compatibilità ESP32:** ESP32-DevKitC V4 (WROOM-32E, 4 MB) è più che sufficiente. Nessuna PSRAM necessaria. Il display non occupa RAM dell'ESP32, a differenza di LVGL.

---

## 2. Collegamento

```
ESP32 GPIO 14 (UART2 TX) ──── R 1kΩ ──── Nextion RX
ESP32 GPIO 27 (UART2 RX) ──── Nextion TX
+5 V (F2) ─────────────────── Nextion +5V   (cavo ≥ 0.5 mm², max 50 cm)
GND ───────────────────────── Nextion GND   (stella di massa comune)
```

| Segnale | ESP32 | Nextion | Note |
|---|---|---|---|
| TX → RX | GPIO 14 | RX (giallo) | Resistenza serie 1 kΩ consigliata |
| RX ← TX | GPIO 27 | TX (blu) | **Verificare con tester il livello di TX**: se è 5 V usare partitore 1 kΩ / 2.2 kΩ verso GND |
| +5 V | linea +5V_SYS | 5V (rosso) | Decoupling 100 µF + 100 nF vicino al connettore |
| GND | GND | GND (nero) | Massa comune con ESP32 |

- GPIO 14 emette un breve impulso PWM durante il boot dell'ESP32: il display lo ignora.
- UART0 (GPIO 1/3) resta riservata al debug via USB.
- Connettore sulla PCB: **J_DISP**, JST/Molex 4 pin (+5V, GND, TX, RX).

---

## 3. Struttura Pagine Nextion (da progettare nell'editor)

| Pagina | Nome | Contenuto |
|---|---|---|
| 0 | `home` | T, RH, CO₂ grandi; setpoint; stato riscaldatore / ventola / umidificatore / luci; icona Wi-Fi |
| 1 | `setpoint` | Setpoint T (15-40 °C), setpoint RH, tasti +/- |
| 2 | `luci_vent` | Slider luminosità LED, slider ventola, switch umidificatore |
| 3 | `setup` | Allarmi, offset sensori, IP, versione firmware |

Convenzioni di nomenclatura dei componenti (devono combaciare con il YAML):

| Componente | Nome | Tipo |
|---|---|---|
| Temperatura | `t_temp` | Testo |
| Umidità | `t_rh` | Testo |
| CO₂ | `t_co2` | Testo |
| Setpoint | `t_set` | Testo |
| Tasto luci | `b_led` | Pulsante (send component id: attivo) |
| Tasto umidificatore | `b_hum` | Pulsante |
| Slider ventola | `h_fan` | Slider |
| Tasti setpoint | `b_up`, `b_dn` | Pulsante |
| Campo selezionato (cursore) | `b_sel0 … b_sel3` | Pulsante/cornice |

---

## 4. Uso Combinato Touch + Encoder

| Azione | Touch | Encoder / pulsanti |
|---|---|---|
| Modifica setpoint | Tasti +/- | Ruota = cursore o valore |
| Cambio campo | Tocco sul campo | Click encoder = entra/esce in modifica |
| Luci LED | Tasto `b_led` | **Pulsante luci dedicato (GPIO 23)** |
| Torna a Home | Tasto indietro | Pulsante MF (pressione breve) |
| Umidificatore | Tasto `b_hum` | Pulsante MF (pressione lunga) |

Logica del cursore (nessun hardware aggiuntivo):

1. Fuori modifica: la rotazione sposta l'evidenziazione tra i campi (`ui_cursor` = 0…3).
2. Click encoder: entra in modifica del campo selezionato (`ui_edit = true`).
3. In modifica: la rotazione cambia il valore (setpoint ±0.5 °C, ventola ±5 %, luci ±5 %).
4. Click di nuovo: conferma ed esce. Pulsante MF breve: annulla e torna alla Home.

---

## 5. Configurazione ESPHome

```yaml
uart:
  id: uart_nextion
  tx_pin: GPIO14
  rx_pin: GPIO27
  baud_rate: 115200

display:
  - platform: nextion
    id: nextion_disp
    uart_id: uart_nextion
    update_interval: 1s
    start_up_page: 0
    # Aggiornamento .tft via OTA (opzionale)
    # tft_url: "http://<server>/cella.tft"
    lambda: |-
      it.set_component_text_printf("t_temp", "%.1f", id(sht45_temp).state);
      it.set_component_text_printf("t_rh",   "%.0f", id(sht45_rh).state);
      it.set_component_text_printf("t_co2",  "%.0f", id(scd41_co2).state);
      it.set_component_text_printf("t_set",  "%.1f", id(climate_heat).target_temperature);
      // evidenziazione cursore
      for (int i = 0; i < 4; i++) {
        it.set_component_background_color(
          ("b_sel" + std::to_string(i)).c_str(),
          (i == id(ui_cursor)) ? (id(ui_edit) ? "RED" : "YELLOW") : "BLACK");
      }

binary_sensor:
  # Eventi touch dal display
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

  - platform: nextion
    page_id: 1
    component_id: 3           # b_up
    name: "Touch Setpoint +"
    on_press:
      - lambda: |-
          auto t = id(climate_heat).target_temperature;
          auto call = id(climate_heat).make_call();
          call.set_target_temperature(std::min(t + 0.5f, 40.0f));
          call.perform();

  - platform: nextion
    page_id: 1
    component_id: 4           # b_dn
    name: "Touch Setpoint -"
    on_press:
      - lambda: |-
          auto t = id(climate_heat).target_temperature;
          auto call = id(climate_heat).make_call();
          call.set_target_temperature(std::max(t - 0.5f, 15.0f));
          call.perform();

globals:
  - id: ui_cursor
    type: int
    initial_value: "0"
  - id: ui_edit
    type: bool
    initial_value: "false"
```

I `component_id` vanno verificati nell'editor Nextion (proprietà *id* del componente) e devono avere *Send Component ID* attivo su *Touch Press Event*.

---

## 6. Caricamento del file .tft

| Metodo | Procedura | Note |
|---|---|---|
| **microSD** | Copia `.tft` su SD FAT32, inseriscila a display spento, accendi | Il più affidabile |
| **Via UART da ESPHome** | `tft_url` + pulsante `nextion.upload_tft` | Richiede HTTP (non HTTPS), baud ≤ 115200 consigliato |

Avvertenze:
- Il display va alimentato dall'alimentatore +5 V, non dal solo USB dell'ESP32.
- Tenere una copia del progetto `.HMI` nel repository (cartella `NEXTION/`, da aggiungere).

---

## 7. Consumo e Alimentazione

| Componente | Tipico | Picco |
|---|---|---|
| Nextion 5" | ~250 mA @ 5 V | ~600 mA @ 5 V (backlight 100 %) |

Linea +5 V rivista: totale ~**1.25 A** di picco contro il fusibile F2 da 2 A SB (margine 1.6×). Ridurre la luminosità (`dim=60`) in uso normale abbassa il consumo di circa 150 mA.

---

## 8. Test di Messa in Servizio

- [ ] Display acceso con +5 V e GND comune
- [ ] `it.set_component_text` aggiorna T / RH / CO₂
- [ ] Eventi touch visibili nel log ESPHome
- [ ] Encoder sposta il cursore, click entra/esce in modifica
- [ ] Pulsante luci (GPIO 23) accende/spegne la striscia LED
- [ ] Nessun reset durante l'avvio del riscaldatore (verifica caduta +5 V)

---

**Versione:** 1.1 (rev. display Nextion UART)  
**Data:** Novembre 2026  
**Status:** ✅ Documentazione aggiornata
