# 💡 Beleuchtung Hybrid & Adaptive

Bewegungsgesteuerte Lichtsteuerung mit drei automatischen Helligkeitsmodi – abhängig von der Tageszeit. Einsetzbar in beliebigen Räumen: Schlafzimmer, Flur, Badezimmer, Büro, Wohnzimmer, ...

Integriert Adaptive Lighting für natürliche Farbtemperatur und einen Nachtmodus-Schalter der das Licht komplett unterdrückt. Zeiten sind flexibel per Festwert oder über Helfer / Sonnen-Template-Sensoren steuerbar.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/harwin63/ha_bp/main/beleuchtung-hybrid-adaptive/blueprint_beleuchtung_hybrid-steuerung.yaml)

---

## ✅ Voraussetzungen

- Bewegungsmelder oder Präsenzsensor (`binary_sensor` mit device_class `motion` oder `occupancy`)
- Smartes Licht (kompatibel mit `light`-Domain)
- [Adaptive Lighting Integration](https://github.com/basnijholt/adaptive-lighting) installiert  
  → Über HACS: **HACS → Integrationen → Adaptive Lighting**
- Nachtmodus-Toggle (`input_boolean` oder `switch`)

---

## 🔧 Einmalige Vorbereitung

Pro Raum werden folgende Helfer empfohlen. Die Namen sind frei wählbar – hier am Beispiel „Schlafzimmer". Anlegen unter:  
**Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen**

Eine fertige Referenz-Datei für alle Helfer liegt unter [`helfer.yaml`](helfer.yaml).

### Übersicht der Helfer (Beispiel Schlafzimmer)

| Typ | Name | Beschreibung |
|---|---|---|
| `input_boolean` | `Nachtmodus` | Schalter – wenn AN bleibt das Licht aus |
| `input_select` | `Schlafzimmer Zeitsteuerung Modus` | Umschalten zwischen `Uhrzeit` und `Sonnen-Event` |
| `input_datetime` | `Schlafzimmer Morgen Ende (Uhrzeit)` | Standard: 07:30 |
| `input_datetime` | `Schlafzimmer Abend Start (Uhrzeit)` | Standard: 21:00 |
| `input_number` | `Schlafzimmer Morgen Ende Offset` | Offset auf Sonnenaufgang in Minuten (Standard: +60 min) |
| `input_number` | `Schlafzimmer Abend Start Offset` | Offset auf Sonnenuntergang in Minuten (Standard: 0 min) |

> 💡 Der `Nachtmodus` Toggle kann raumübergreifend genutzt werden – einmal anlegen, in mehreren Blueprint-Instanzen verwenden. Oder pro Raum einen eigenen anlegen, z.B. `Nachtmodus Schlafzimmer`.

> 💡 Der gleiche `Nachtmodus` Toggle wird auch vom Blueprint **Nachtmodus Auto-Ausschaltung** genutzt.

---

## ▶️ Blueprint einrichten

1. Blueprint importieren (Badge oben oder Raw-URL manuell eingeben)
2. **Einstellungen → Automationen → Blueprints → Blueprint verwenden**
3. Felder ausfüllen:

| Feld | Beschreibung |
|---|---|
| Bewegungsmelder | Bewegungs- oder Präsenzsensor des Raums |
| Licht | Die zu steuernde Lichtentität |
| Adaptive Lighting Sleep Mode Schalter | `switch.adaptive_lighting_sleep_mode_...` |
| Nachtmodus-Schalter | `input_boolean` oder `switch` |
| Morgen Ende (Festwert) | Uhrzeit ab der Tagesmodus gilt (Standard: 09:00) |
| Morgen Ende (Helfer-Option) | Optional: `input_datetime` oder Sonnen-Template-Sensor |
| Abend Start (Festwert) | Uhrzeit ab der Abendmodus gilt (Standard: 20:00) |
| Abend Start (Helfer-Option) | Optional: `input_datetime` oder Sonnen-Template-Sensor |
| Wartezeit (Festwert) | Sekunden nach letzter Bewegung bis Licht ausgeht (Standard: 120s) |
| Wartezeit (Helfer-Option) | Optional: `input_number` für dynamische Wartezeit |

> 💡 **Festwert oder Helfer?** Wenn ein Helfer ausgewählt ist, hat dieser immer Vorrang vor dem Festwert. Ohne Helfer-Auswahl gilt der eingetragene Festwert – so funktioniert der Blueprint auch ohne zusätzliche Helfer sofort.

---

## ⚙️ Wie es funktioniert

```
Bewegung erkannt
  → Ist Nachtmodus AN?
      Ja  →  Licht bleibt aus

      Nein → Welche Tageszeit ist es?

          Morgen   (vor Morgen-Ende-Zeit)  →  Adaptive Lighting Sleep AN  → Licht gedimmt & warm
          Tagsüber (zwischen den Zeiten)   →  Adaptive Lighting Sleep AUS → Licht hell & tageslichtweiß
          Abend    (nach Abend-Start-Zeit) →  Adaptive Lighting Sleep AN  → Licht gedimmt & warm

  → Warten bis keine Bewegung mehr (konfigurierbare Wartezeit)
  → Licht aus (sanfter Übergang 3s)
```

---

## 🔗 Raw-URL für manuellen Import

```
https://raw.githubusercontent.com/harwin63/ha_bp/main/beleuchtung-hybrid-adaptive/blueprint_beleuchtung_hybrid-steuerung.yaml
```
