# Tasmota + MIEL_HVAC (Mitsubishi CN105) — compilación fiable sin tasmocompiler

## Por qué falla tasmocompiler
El driver MiElHVAC no viene en ninguna compilación estándar de Tasmota:
hay que activarlo con `#define USE_MIEL_HVAC` en `user_config_override.h`
antes de compilar. En tasmocompiler esto se hace en el paso "Custom
parameters", pero si la línea no llega al fichero (bloques #ifndef mal
cerrados, versión antigua, o el campo no se guarda) el bin sale sin el
driver y los comandos `HVAC*` dan "Unknown command".

## Método recomendado: GitHub Actions (5 min, sin instalar nada)
1. Crea un repositorio nuevo en tu GitHub (privado vale).
2. En él, crea el fichero `.github/workflows/build-tasmota-miel.yml`
   con el contenido de `build-tasmota-miel.yml` (incluido aquí al lado).
3. Pestaña **Actions → Build Tasmota with MIEL_HVAC → Run workflow**.
4. En ~5 minutos descarga el artifact `tasmota-miel-hvac` con
   `tasmota.bin` (ESP8266) y `tasmota32.bin` (ESP32).

## Alternativa local: PlatformIO
```
git clone --depth 1 --branch v16.0.0 https://github.com/arendst/Tasmota
cd Tasmota
# crea tasmota/user_config_override.h con:
#   #ifndef USER_CONFIG_OVERRIDE_H
#   #define USER_CONFIG_OVERRIDE_H
#   #define USE_MIEL_HVAC
#   #endif
pip install platformio
pio run -e tasmota
```
El bin queda en `build_output/firmware/tasmota.bin`.

## Configuración tras flashear
1. Flashea con [Tasmota WebInstaller](https://tasmota.github.io/install/) o esptool.
2. Configura WiFi (wifiX no hace falta, es la build normal).
3. En *Configuration → Configure Module*:
   - GPIO1 → `MiEl HVAC Tx`
   - GPIO3 → `MiEl HVAC Rx`
4. Verifica en consola: `HVACSetTemp 21` debe responder y el split debe
   reaccionar. Si dice "Unknown command", el bin no lleva el driver.
5. Conexión física al CN105: 5V→Vin, GND→GND, TX(AC)→GPIO3(RX),
   RX(AC)→GPIO1(TX). Recuerda la adaptación de nivel 5V→3,3V en RX del ESP.

## Comandos principales del driver
- `POWER 1|0` — encender / apagar
- `HVACSetHAMode auto|heat|dry|cool|fan_only|off` — modo estilo Home Assistant
- `HVACSetTemp 16..31` — consigna
- `HVACSetFan auto|quiet|1..4` — velocidad ventilador
- `HVACSetVane 1..5|auto|swing` — vano vertical
- `HVACSetWideVane LL|L|C|R|RR|split|swing` — vano horizontal

El estado sale por MQTT en `tele/<topic>/SENSOR` (temperatura de sala) y
`tele/<topic>/HVACSETTINGS`. Para Home Assistant existe la integración
dedicada [ha-tasmota-mielhvac](https://github.com/aldweb/ha-tasmota-mielhvac).
