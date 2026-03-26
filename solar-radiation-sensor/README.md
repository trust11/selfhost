# Solarstrahlungs-Sensor fuer Home Assistant

Lokale Messung der Sonneneinstrahlung mit ESP32 + BH1750, optimiert fuer
Bergstandorte mit Horizontverschattung.

## Problem

- Wetterprognosen gehen von freiem Horizont aus
- Am Bergstandort: spaete Morgensonne, frueher Nachmittagsschatten
- PV-Ertragsprognosen sind dadurch ungenau

## Loesung

Ein guenstiger Lux-Sensor (BH1750) am ESP32 misst die **tatsaechliche
Einstrahlung vor Ort** und meldet sie via ESPHome an Home Assistant.

## Hardware (ca. 10-15 EUR)

| Bauteil           | Preis   |
|--------------------|---------|
| ESP32 DevKit       | ~5 EUR  |
| BH1750 Breakout    | ~3 EUR  |
| Gehaeuse (IP65)    | ~5 EUR  |
| Dupont-Kabel       | ~1 EUR  |

### Verkabelung

```
ESP32 GPIO21 (SDA) --> BH1750 SDA
ESP32 GPIO22 (SCL) --> BH1750 SCL
ESP32 3.3V         --> BH1750 VCC
ESP32 GND          --> BH1750 GND
```

## Sensoren in Home Assistant

| Sensor                        | Einheit | Beschreibung                          |
|-------------------------------|---------|---------------------------------------|
| Sonneneinstrahlung Lux        | lx      | Roher Lux-Wert                        |
| Sonneneinstrahlung W/m²       | W/m²    | Umgerechnete Bestrahlungsstaerke      |
| Geschaetzte PV-Leistung       | W       | Momentane PV-Leistung (konfigurierbar)|
| Geschaetzter Tagesertrag      | kWh     | Kumulierter Tagesertrag               |
| Sonne aktiv                   | on/off  | Scheint die Sonne gerade?             |
| Einstrahlungs-Kategorie       | Text    | Nacht/Bewoelkt/Volle Sonne           |

## Installation

1. ESPHome Add-on in Home Assistant installieren
2. `esphome-solar-sensor.yaml` als neues Geraet hinzufuegen
3. WiFi-Credentials in `secrets.yaml` eintragen
4. ESP32 flashen
5. Sensor montieren (gleiche Ausrichtung wie PV-Module)
6. Optional: Automationen aus `home-assistant-automations.yaml` uebernehmen

## Anpassung an deine PV-Anlage

In `esphome-solar-sensor.yaml` die folgenden Werte anpassen:

```yaml
float panel_area_m2 = 20.0;       // Gesamtflaeche deiner Module in m²
float module_efficiency = 0.18;    // Wirkungsgrad (Datenblatt)
float system_losses = 0.85;        // Systemverluste
```

## Alternativen

| Sensor        | Preis     | Genauigkeit | Bemerkung                    |
|---------------|-----------|-------------|-------------------------------|
| BH1750        | ~3 EUR    | Gut         | Lux-basiert, guenstig         |
| VEML7700      | ~5 EUR    | Besser      | Breiteres Spektrum + UV       |
| Apogee SP-110 | ~150 EUR  | Sehr gut    | Echtes Pyranometer, W/m²      |
| TSL2591       | ~5 EUR    | Gut         | Hohe Dynamik, IR + sichtbar   |

Fuer die meisten Heimanwendungen reicht der BH1750 voellig aus.
