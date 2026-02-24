# ⚡ Energie-Ladetracker

Misst automatisch Verbrauch und Kosten eines Ladevorgangs über einen Smartplug mit Energiemessung. Nach dem Ladeende werden kWh und Kosten gespeichert – beim nächsten Laden startet der Zähler automatisch neu.

Einsetzbar für: Fahrradakku, E-Scooter, E-Bike, Laptop, Werkzeugakku, ...

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/harwin63/ha_bp/main/energie-ladetracker/blueprint_ladetracker_vollstaendig.yaml)

---

## ✅ Voraussetzungen

- Smartplug mit **Leistungsmessung (Watt)** und **Energiemessung (kWh)**  
  Beispiele: Shelly Plug S, TP-Link Kasa, Nous A1T, IKEA Tradfri Outlet

---

## 🔧 Einmalige Vorbereitung

### Schritt 1 – Strompreis-Helfer (einmalig global, nur einmal für alle Geräte nötig)

**Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → Zahl**

| Feld | Wert |
|---|---|
| Name | `Strompreis pro kWh` |
| Einheit | `€/kWh` |
| Min | `0` |
| Max | `1` |
| Schritt | `0.01` |
| Startwert | z.B. `0.32` |

### Schritt 2 – Drei Helfer pro Gerät anlegen

**Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → Zahl**

| # | Name (Beispiel) | Einheit | Min | Max | Schritt |
|---|---|---|---|---|---|
| 1 | `Fahrradakku Ladestart intern` | `kWh` | `0` | `999999` | `0.001` |
| 2 | `Fahrradakku Verbrauch kWh` | `kWh` | `0` | `999` | `0.001` |
| 3 | `Fahrradakku Kosten EUR` | `€` | `0` | `999` | `0.001` |

> 💡 Für jedes weitere Gerät (E-Scooter, Laptop, ...) einfach drei neue Helfer mit passendem Namen anlegen und eine weitere Blueprint-Instanz erstellen.

### Schritt 3 – Template-Helfer für Live-Anzeige (pro Gerät)

Diese Helfer berechnen während des Ladens in Echtzeit Verbrauch und Kosten und werden von der Lovelace-Karte benötigt. Anlegen über die UI – kein Bearbeiten der `configuration.yaml` nötig.

**Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → Template → Template für einen Sensor**

**Template-Helfer 1 – Aktueller Verbrauch (kWh)**

| Feld | Wert |
|---|---|
| Name | `Fahrradakku Aktueller Verbrauch kWh` |
| Einheit | `kWh` |
| Gerätetyp | `Energie` |
| Zustandsklasse | `Summenwert` |
| Icon | `mdi:lightning-bolt` |
| Zustandsvorlage | siehe unten |

```jinja
{% set aktuell = states('sensor.DEIN_ENERGIEZAEHLER_SENSOR') | float(0) %}
{% set start   = states('input_number.fahrradakku_ladestart_intern') | float(0) %}
{% if start > 0 %}
  {{ [aktuell - start, 0] | max | round(3) }}
{% else %}
  0
{% endif %}
```

**Template-Helfer 2 – Aktuelle Kosten (€)**

| Feld | Wert |
|---|---|
| Name | `Fahrradakku Aktueller Verbrauch EUR` |
| Einheit | `€` |
| Gerätetyp | `Monetär` |
| Zustandsklasse | `Summenwert` |
| Icon | `mdi:currency-eur` |
| Zustandsvorlage | siehe unten |

```jinja
{% set kwh   = states('sensor.fahrradakku_aktueller_verbrauch_kwh') | float(0) %}
{% set preis = states('input_number.strompreis_pro_kwh') | float(0.32) %}
{{ (kwh * preis) | round(3) }}
```

> 💡 Die Entitäts-IDs `sensor.DEIN_ENERGIEZAEHLER_SENSOR` und `input_number.fahrradakku_ladestart_intern` an dein Gerät anpassen. Der zweite Template-Helfer referenziert den ersten – daher zuerst Helfer 1 anlegen!

---

## ▶️ Blueprint einrichten

1. Blueprint importieren (Badge oben oder Raw-URL manuell eingeben)
2. **Einstellungen → Automationen → Blueprints → Blueprint verwenden**
3. Felder ausfüllen:

| Feld | Beschreibung |
|---|---|
| 📛 Gerätename | Frei wählbar, erscheint in Benachrichtigungen |
| ⚡ Leistungssensor | Watt-Sensor des Smartplugs |
| 🔋 Energiezähler | kWh-Sensor des Smartplugs |
| 🔧 Helfer Startwert | Helfer 1 (intern, z.B. `Fahrradakku Ladestart intern`) |
| 🔧 Helfer Verbrauch | Helfer 2 (z.B. `Fahrradakku Verbrauch kWh`) |
| 🔧 Helfer Kosten | Helfer 3 (z.B. `Fahrradakku Kosten EUR`) |
| 💰 Strompreis-Helfer | Globaler Strompreis-Helfer |
| ▶️ Einschaltschwelle | Ab wieviel Watt gilt Laden als gestartet (Standard: 2W) |
| ⏹️ Ausschaltschwelle | Unter wieviel Watt gilt Laden als beendet (Standard: 5W) |
| ⏱️ Wartezeit Ladeende | Minuten unter Schwelle bis "Laden beendet" ausgelöst wird |
| 📱 Benachrichtigung | `notify.persistent_notification` oder `notify.mobile_app_...` |

---

## 📊 Lovelace-Karte

Eine fertige Beispielkarte liegt unter [`lovelace_beispiel.yaml`](lovelace_beispiel.yaml).

Einfügen über: **Dashboard → Bearbeiten → Karte hinzufügen → YAML manuell eingeben**

Die Entitätsnamen in der Karte an deine Helfer-Namen anpassen (alle Stellen mit `DEIN_GERAET` ersetzen).

---

## ⚙️ Wie es funktioniert

```
Smartplug > 2W  →  Ladestart erkannt
                    Zählerstand wird als Referenz gespeichert
                    
Smartplug < 5W  →  Ladeende erkannt (nach 2 Min.)
  (2 Min. lang)     Verbrauch  = aktueller Zähler − Referenz
                    Kosten     = Verbrauch × Strompreis
                    → Ergebnis in Helfer speichern
                    → Benachrichtigung senden
                    → Referenz zurücksetzen (bereit für nächsten Ladevorgang)
```

---

## 🔗 Raw-URL für manuellen Import

```
https://raw.githubusercontent.com/harwin63/ha_bp/main/energie-ladetracker/blueprint_ladetracker_vollstaendig.yaml
```
