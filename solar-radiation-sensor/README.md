# Autarker Solarstrahlungs-Sensor fuer Home Assistant

Komplett solarbetriebener Sensor mit Supercap-Puffer - kein Akku, kein
Netzanschluss, kein Kabel. Misst die tatsaechliche Sonneneinstrahlung
am Standort und schaetzt den PV-Ertrag der Dachanlage ab, um
EV-Ladung zu optimieren.

## Problem

- PV-Anlage auf dem Dach, aber kein Zugang zum Wechselrichter
- Wetterprognosen gehen von freiem Horizont aus
- Bergstandort: spaete Morgensonne, frueher Nachmittagsschatten
- Wissen wann genug Solarstrom da ist, um das Auto zu laden

## Loesung

Autarker Sensor mit eigenem Solarpanel + Supercaps als Puffer.
Solange Sonne da ist, laeuft der Sensor und misst. Bei Wolken
ueberbruecken die Supercaps ca. 30-60 Minuten. Wenn die Sonne
dann wirklich weg ist, geht der Sensor aus - und das ist ok,
denn dann gibt es auch keinen PV-Strom zum Messen.

## Einkaufsliste (ca. 15-25 EUR)

| Bauteil                        | Preis    | Bemerkung                              |
|--------------------------------|----------|----------------------------------------|
| ESP32 DevKit V1 (WROOM-32)    | ~5-8 EUR | AliExpress, Bastelgarage.ch, Amazon    |
| BH1750 Breakout (GY-302)      | ~2-4 EUR | Lux-Sensor                             |
| Mini-Solarpanel 5V/6V, 1-2W   | ~3-5 EUR | ca. 80x55mm genuegt                    |
| Supercap-Modul 10F 5.5V       | ~3-5 EUR | ODER 2x 25F 2.7V in Reihe             |
| Schottky-Diode (1N5817)       | ~0.20 EUR| Verhindert Rueckfluss ins Panel        |
| IP65 Gehaeuse                  | ~3-5 EUR | Wetterfest                             |
| Dupont-Kabel (4 Stueck)       | ~1 EUR   | Oft beim ESP32 dabei                   |
| 2x 100kOhm Widerstaende       | ~0.10 EUR| Fuer Supercap-Spannungsmessung         |

**Kein Akku, kein Lademodul, kein TP4056 noetig!**
Supercaps sind wartungsfrei, halten quasi ewig (>500'000 Ladezyklen)
und funktionieren auch bei Kaelte im Winter problemlos.

## Verkabelung

```
SOLARPANEL                                  ESP32
  (+) ──> Schottky-Diode ──┬──────────────> VIN (oder 5V)
  (-) ──────────────────┬──┴──────────────> GND
                        │
   Supercap (+) ────────┤ (nach der Diode!)
   Supercap (-) ────────┘ (an GND)

   Die Diode verhindert, dass die Supercaps sich
   ueber das Panel entladen wenn keine Sonne da ist.

ESP32                    BH1750
  GPIO21 (SDA) ─────────> SDA
  GPIO22 (SCL) ─────────> SCL
  3.3V ─────────────────> VCC
  GND ──────────────────> GND

Supercap-Spannung messen (empfohlen):
  VIN ───> 100kOhm ──┬──> ESP32 GPIO35
                      │
                     100kOhm
                      │
  GND ────────────────┘
```

## So funktioniert's

1. Sonne scheint -> Solarpanel liefert Strom + laedt Supercaps
2. ESP32 laeuft, misst Lux mit BH1750, sendet an Home Assistant
3. Wolke kommt -> Supercaps ueberbruecken 30-60 Minuten
4. Sonne bleibt weg -> ESP32 geht in Deep-Sleep (spart Strom)
5. Supercaps leer -> Sensor geht aus (kein Problem, keine Sonne = kein PV-Strom)
6. Naechster Morgen -> Sonne laedt Supercaps, Sensor startet automatisch

**Smarte Deep-Sleep Steuerung:**
- Sonne da (> 30 W/m²) -> ESP32 bleibt wach, misst alle 30 Sekunden
- Sonne weg -> Deep-Sleep, wacht alle 5 Min kurz auf zum Pruefen
- So halten die Supercaps deutlich laenger

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
| Supercap Spannung              | V       | Aktuelle Spannung der Supercaps           |
| Supercap Ladestand             | %       | Ladestand der Supercaps                   |
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

Der OTA Modus Schalter in Home Assistant verhindert Deep-Sleep,
damit der ESP32 wach bleibt und per WiFi geflasht werden kann.
Nur bei Sonnenschein moeglich (Sensor muss laufen).

## Tipps

- **Solarpanel-Ausrichtung:** Gleich wie deine Dach-PV-Module ausrichten,
  dann misst der Sensor genau das, was die Dachanlage auch "sieht"
- **WiFi-Reichweite:** ESP32 hat ca. 30-50m Reichweite. Falls zu weit weg,
  gibt es ESP32 mit externer Antenne
- **Kalibrierung:** Nach ein paar Tagen Betrieb kannst du die Werte mit
  deiner Stromrechnung vergleichen und den `system_losses` Faktor anpassen
- **Supercap-Groesse:** 10F reicht fuer ~15-30 Min Puffer,
  2x 25F in Reihe fuer ~30-60 Min. Bei Deep-Sleep noch deutlich laenger.
