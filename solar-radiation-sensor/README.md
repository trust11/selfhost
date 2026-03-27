# Autarker Solarstrahlungs-Sensor fuer Home Assistant

Komplett solarbetriebener Sensor - kein Netzanschluss, kein Kabel.
Misst die tatsaechliche Sonneneinstrahlung am Standort und schaetzt
den PV-Ertrag der Dachanlage ab, um EV-Ladung zu optimieren.

## Problem

- PV-Anlage auf dem Dach, aber kein Zugang zum Wechselrichter
- Wetterprognosen gehen von freiem Horizont aus
- Bergstandort: spaete Morgensonne, frueher Nachmittagsschatten
- Wissen wann genug Solarstrom da ist, um das Auto zu laden

## Loesung

Autarker Sensor mit eigenem Solarpanel + Akku, der die echte Einstrahlung
misst und per WiFi an Home Assistant meldet. Kann ueberall aufgehaengt werden.

## Einkaufsliste (ca. 15-25 EUR)

| Bauteil                        | Preis    | Bemerkung                              |
|--------------------------------|----------|----------------------------------------|
| ESP32 DevKit V1 (WROOM-32)    | ~5-8 EUR | AliExpress, Bastelgarage.ch, Amazon    |
| BH1750 Breakout (GY-302)      | ~2-4 EUR | Lux-Sensor                             |
| Mini-Solarpanel 5V/6V, 1-2W   | ~3-5 EUR | ca. 80x55mm genuegt                    |
| TP4056 Lademodul (mit Schutz!) | ~1-2 EUR | Micro-USB Variante mit DW01 Schutz-IC  |
| 18650 Li-Ion Akku              | ~3-5 EUR | z.B. aus altem Laptop, oder neu kaufen  |
| 18650 Halter                   | ~1 EUR   | Zum Einloeten oder Clippen              |
| IP65 Gehaeuse                  | ~3-5 EUR | Wetterfest                             |
| Dupont-Kabel (4 Stueck)       | ~1 EUR   | Oft beim ESP32 dabei                   |
| 2x 100kOhm Widerstaende       | ~0.10 EUR| Fuer Batteriespannungsmessung (optional)|

**Optional:** 2x 100kOhm Widerstaende als Spannungsteiler, um die
Batteriespannung zu ueberwachen. Sehr empfohlen damit du siehst,
ob der Akku voll ist oder leer wird.

## Verkabelung

```
SOLARPANEL               TP4056 LADEMODUL            ESP32
  (+) ──────────────────> IN+
  (-) ──────────────────> IN-
                          OUT+ ───────────────────> VIN (oder 5V)
                          OUT- ───────────────────> GND
                          BAT+ ──> 18650 (+)
                          BAT- ──> 18650 (-)

ESP32                    BH1750
  GPIO21 (SDA) ─────────> SDA
  GPIO22 (SCL) ─────────> SCL
  3.3V ─────────────────> VCC
  GND ──────────────────> GND

Batteriespannung messen (optional):
  BAT+ ──> 100kOhm ──┬──> ESP32 GPIO35
                      │
                     100kOhm
                      │
  GND ────────────────┘
```

## So funktioniert's

1. ESP32 wacht alle **2 Minuten** aus dem Deep-Sleep auf
2. Misst Lux-Wert mit BH1750
3. Rechnet in W/m² um und schaetzt PV-Leistung
4. Sendet alles an Home Assistant
5. Geht wieder schlafen (~10uA Verbrauch)

Der 18650 Akku haelt damit **mehrere Tage ohne Sonne**.
Mit Solarpanel laedt er sich tagsueglich automatisch nach.

## Sensoren in Home Assistant

| Sensor                        | Einheit | Beschreibung                              |
|-------------------------------|---------|-------------------------------------------|
| Sonneneinstrahlung Lux        | lx      | Roher Lux-Wert                            |
| Sonneneinstrahlung             | W/m²    | Umgerechnete Bestrahlungsstaerke          |
| Geschaetzte PV-Leistung       | W       | Momentane PV-Leistung der Dachanlage      |
| Verfuegbar fuer EV-Ladung     | W       | Nach Abzug Grundlast Haushalt             |
| EV Lade-Ampere moeglich       | A       | Moegliche Ampere fuer Wallbox             |
| EV Laden moeglich              | on/off  | Genug Strom fuer Wallbox (>= 6A)?        |
| Ladeempfehlung                 | Text    | Klartext-Empfehlung                       |
| Batteriespannung               | V       | Spannung des 18650 Akkus                  |
| Batteriestand                  | %       | Ladestand in Prozent                      |
| Batterie Status                | Text    | Voll/OK/Niedrig/Kritisch                  |
| Sonne aktiv                   | on/off  | Scheint die Sonne gerade?                 |
| Einstrahlungs-Kategorie       | Text    | Nacht/Bewoelkt/Teilweise/Volle Sonne     |

## Installation

1. ESPHome Add-on in Home Assistant installieren
2. `esphome-solar-sensor.yaml` als neues Geraet hinzufuegen
3. WiFi-Credentials in `secrets.yaml` eintragen
4. ESP32 per USB flashen (einmalig)
5. Alles zusammenbauen und draussen montieren
6. Optional: Automationen aus `home-assistant-automations.yaml` uebernehmen

## Anpassung an deine Dachanlage

In `esphome-solar-sensor.yaml` diese Werte anpassen:

```yaml
float peak_power_wp = 10000.0;  // Nennleistung deiner Dachanlage in Wp
float system_losses = 0.80;     // Systemverluste
float household_base_load = 400.0;  // Grundlast deines Haushalts in W
```

## OTA-Updates (kabellos)

Da der Sensor im Deep-Sleep ist, muss fuer Updates der **OTA Modus**
Schalter in Home Assistant eingeschaltet werden. Dann bleibt der ESP32
wach und kann per WiFi geflasht werden.

## Tipps

- **Solarpanel-Ausrichtung:** Gleich wie deine Dach-PV-Module ausrichten,
  dann misst der Sensor genau das, was die Dachanlage auch "sieht"
- **WiFi-Reichweite:** ESP32 hat ca. 30-50m Reichweite. Falls zu weit weg,
  gibt es ESP32 mit externer Antenne
- **Kalibrierung:** Nach ein paar Tagen Betrieb kannst du die Werte mit
  deiner Stromrechnung vergleichen und den `system_losses` Faktor anpassen
