# 🤖 Dreame Vacuum Multi-Floor Button Control 

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/your-repo/vacuum-control-blueprint)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.4+-green.svg)](https://www.home-assistant.io/)
[![License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)

> **Automation der Reinigungsvorbereitung bei mehreren Stockwerken über phyaische Schalter. Wechselt die Karte und startet in Abhängigkeit der Kartenauswahl verschieden Vorbereitungsprogramme. Der Roboter wird so je nach Auswahl zum manuellen umsetzen in einen anderen Wohnbereich oder Stockwerk vorbereitet. Die Reinigung startet automatisch nach dem umsetzen. Der Roboter waret nach der Reinigung und wechselt die Karte beim erneuten umsetzen automatisch zurück auf die Karte mit Reinigungsstation Standort.   

---

## 📋 Inhaltsverzeichnis

- [Features](#-features)
- [Voraussetzungen](#-voraussetzungen)
- [Installation](#-installation)
- [Konfiguration](#%EF%B8%8F-konfiguration)
  - [Geräte & Entitäten](#1%EF%B8%8F⃣-geräte--entitäten)
  - [Karten-Konfiguration](#2%EF%B8%8F⃣-karten-konfiguration)
  - [Button-Zuweisungen](#3%EF%B8%8F⃣-button-zuweisungen)
  - [Reinigungs-Modi](#4%EF%B8%8F⃣-reinigungs-modi)
  - [Status & Timeouts](#5%EF%B8%8F⃣-status--timeouts)
  - [Erweiterte Optionen](#6%EF%B8%8F⃣-erweiterte-optionen)
- [Funktionsweise](#-funktionsweise)
- [Troubleshooting](#-troubleshooting)
- [FAQ](#-faq)
- [Changelog](#-changelog)

---

## ✨ Features

### Kernfunktionen
- ✅ **6 frei konfigurierbare Buttons** für MQTT-Schalter (Zigbee2MQTT)
- ✅ **Intelligente Start/Pause/Resume-Logik** mit Statuserkennung
- ✅ **Automatisches Pausieren** nach Pad-Befeuchtung (Transport zu anderen Stockwerken)
- ✅ **Kartenbasierte Reinigung** mit konfigurierbaren Wiederholungen
- ✅ **Basisstations-Map-Erkennung** (Start ohne Pause)
- ✅ **Segment-Service-Unterstützung** für präzise Raum-Wiederholungen
- ✅ **Dynamische Timeout-Konfiguration**
- ✅ **Debug-Modus** mit Persistent Notifications

### Design-Prinzipien
- 🎯 **Vollständig GUI-editierbar** - Keine YAML-Kenntnisse erforderlich
- 🧩 **Modular & Erweiterbar** - Einfaches Hinzufügen weiterer Buttons
- 🔒 **Robust & Fehlertolerant** - Queued Mode mit Timeout-Absicherung
- 📱 **Responsive** - Funktioniert mit allen MQTT-fähigen Schaltern
- 🌐 **Mehrsprachig vorbereitet** - Deutsche Beschreibungen, leicht übersetzbar

---

## 🔧 Voraussetzungen

### Hardware
- ✅ Dreame Saugroboter (kompatibel mit [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum))
- ✅ MQTT-fähiger Schalter (z.B. Zigbee2MQTT 6-Fach-Taster)

### Software
- ✅ **Home Assistant** ≥ 2024.4.0
- ✅ **Dreame Vacuum Custom Integration** installiert & konfiguriert
- ✅ **MQTT Broker** (z.B. Mosquitto)
- ✅ **Zigbee2MQTT** (wenn Zigbee-Schalter verwendet wird)

### Erforderliche Entitäten
Der Roboter muss folgende Entitäten bereitstellen:
- `vacuum.{name}` - Hauptentität
- `sensor.{name}_status` - Status-Sensor
- `select.{name}_cleaning_mode` - Reinigungsmodus
- `select.{name}_selected_map` - Kartenauswahl
- `camera.{name}_map` - Live-Map (für Segment-IDs)

---

## 📥 Installation

### Methode 1: UI (Empfohlen)

1. **Blueprint importieren:**
   - Settings → Automations & Scenes → Blueprints
   - Klick auf "Import Blueprint"
   - URL einfügen:`[https://github.com/errormastern/Dreame_Multifloor_Button_Control/raw/main/vacuum_control.yaml`

2. **Automation erstellen:**
   - Klick auf "Create Automation"
   - Blueprint auswählen: "Dreame Vacuum Multi-Button Control"
   - Konfigurieren (siehe unten)

### Methode 2: Manuell

1. Datei speichern als: `config/blueprints/automation/dreame_vacuum/vacuum_control.yaml`

2. In `automations.yaml`:
```yaml
automation:
  - use_blueprint:
      path: dreame_vacuum/vacuum_control.yaml
      input:
        mqtt_switch_topic: "zigbee2mqtt/Schalter Staubsaugroboter"
        vacuum_entity: vacuum.helene_wischer
        # ... weitere Konfiguration
```

3. Automations neu laden:
   - Developer Tools → YAML → Automations neu laden

---

## ⚙️ Konfiguration

### 1️⃣ Geräte & Entitäten

#### MQTT Topic des Schalters
```yaml
mqtt_switch_topic: "zigbee2mqtt/Schalter Staubsaugroboter"
```

**💡 So finden:**
- **Option A:** Zigbee2MQTT UI → Gerät auswählen → "Exposes" → Topic kopieren
- **Option B:** MQTT Explorer → Topic mit `/action` Suffix suchen
- **Option C:** Developer Tools → Events → `mqtt` Event type → Button drücken

**Beispiel Payload:**
```json
{
  "action": "1_single",
  "battery": 100,
  "linkquality": 255
}
```

#### Vacuum-Entität
```yaml
vacuum_entity: vacuum.helene_wischer
```
**Filter:** Nur `vacuum.*` Entitäten werden angezeigt

#### Status-Sensor
```yaml
status_sensor: sensor.helene_wischer_status
```
**Typische Werte:**
- `idle` - Bereit
- `cleaning` - Reinigt
- `paused` - Pausiert
- `returning` - Kehrt zurück
- `charging` - Lädt
- `docked` - An Station

#### Cleaning Mode Select
```yaml
cleaning_mode_select: select.helene_wischer_cleaning_mode
```
**Optionen (je nach Modell):**
- `sweeping` - Nur Saugen
- `mopping` - Nur Wischen
- `sweeping_and_mopping` - Saugen & Wischen

#### Ausgewählte Karte Select
```yaml
selected_map_select: select.helene_wischer_selected_map
```
**Hinweis:** Zeigt den **Anzeigenamen** der Map (nicht die ID!)

#### Map Camera Entity
```yaml
map_camera: camera.helene_wischer_map
```
**Verwendet für:** Extrahieren der Segment-IDs aus den Attributen

---

### 2️⃣ Karten-Konfiguration

#### Basisstations-Karte
```yaml
base_station_map_name: "Wohnzimmer"
```

**❓ Was ist das?**  
Die Karte, auf der sich die Ladestation befindet. Reinigungen auf dieser Karte werden **NICHT pausiert**, da der Roboter nicht transportiert werden muss.

**⚠️ Wichtig:**
- Name **EXAKT** wie in `select.helene_wischer_selected_map` angezeigt
- Groß-/Kleinschreibung beachten!
- Leerzeichen zählen!

**💡 So prüfen:**
1. Developer Tools → States
2. `select.helene_wischer_selected_map` suchen
3. "State" kopieren

#### Kartennamen (Map 1-3)
```yaml
map_1_name: "Wohnzimmer"
map_2_name: "Esszimmer"
map_3_name: "Obergeschoss"
```

**Zweck:** Zuordnung der Buttons 4-6 zu den Maps

**Beispiel-Szenarien:**

**Szenario A: Einfamilienhaus (2 Etagen)**
```yaml
map_1_name: "Erdgeschoss"      # Basisstation hier
map_2_name: "Obergeschoss"     # Manuell hochtragen
map_3_name: "Keller"           # Manuell runtertragen
base_station_map_name: "Erdgeschoss"
```

**Szenario B: Wohnung (1 Etage, 3 Bereiche)**
```yaml
map_1_name: "Wohnbereich"      # Basisstation hier
map_2_name: "Schlafbereich"
map_3_name: "Büro"
base_station_map_name: "Wohnbereich"
```

---

### 3️⃣ Button-Zuweisungen

Konfiguriere die MQTT Payload-Werte für jeden Button:

```yaml
button_1_action: "1_single"  # Modus: Nur Saugen
button_2_action: "2_single"  # Modus: Saugen + Wischen
button_3_action: "3_single"  # START/PAUSE/RESUME
button_4_action: "4_single"  # Karte 1
button_5_action: "5_single"  # Karte 2
button_6_action: "6_single"  # Karte 3
```

**💡 Anpassbar:** Wenn dein Schalter andere Werte sendet (z.B. `button_1_click`), einfach hier anpassen!

**📋 Typische MQTT Action-Formate:**
- IKEA Tradfri: `toggle`, `brightness_up_click`, `brightness_down_click`
- Aqara Opple: `button_1_single`, `button_1_double`, `button_1_hold`
- Tuya/Moes: `1_single`, `2_single`, `3_single`, etc.

---

### 4️⃣ Reinigungs-Modi

#### Nur Saugen - Option
```yaml
sweeping_mode_value: "sweeping"
```
**💡 So herausfinden:**
1. Developer Tools → States
2. `select.helene_wischer_cleaning_mode` suchen
3. Manuell über App/UI auf "Nur Saugen" stellen
4. Wert aus "State" kopieren

#### Saugen + Wischen - Option
```yaml
sweeping_mopping_mode_value: "sweeping_and_mopping"
```

#### Reinigungsdurchgänge
```yaml
cleaning_repeats: 2
```
**Bereich:** 1-3 Durchgänge  
**Empfehlung:** 2x für gründliche Reinigung

**⚠️ Wichtig:** Gilt für **alle Maps außer Basisstation**!

---

### 5️⃣ Status & Timeouts

#### Aktive Reinigungs-Status
```yaml
cleaning_states: "cleaning,returning,zone_cleaning,room_cleaning"
```
**Zweck:** Liste aller Status, die bedeuten "Reinigung läuft"

**Format:** Komma-getrennt, **KEINE Leerzeichen**

**Typische Werte:**
- `cleaning` - Aktive Reinigung
- `returning` - Rückkehr zur Station
- `zone_cleaning` - Zonenreinigung
- `room_cleaning` - Raumreinigung
- `segment_cleaning` - Segmentreinigung

#### Pausiert-Status
```yaml
paused_state: "paused"
```

#### Befeuchtungs-Status (Optional!)
```yaml
moistening_status: ""  # Leer lassen wenn unbekannt
```

**❓ Wofür?**  
Beim Wischmodus befeuchtet der Roboter erst die Pads, bevor er losfährt. Während dieser Zeit soll **noch nicht pausiert** werden.

**📋 So herausfinden:**

1. **Vorbereitung:**
   - Wischmodus aktivieren (Button 2)
   - Map 2 oder 3 wählen (Button 5/6)
   - Debug-Modus aktivieren (siehe unten)

2. **Test-Reinigung starten:**
   - Button 3 drücken
   - Developer Tools → States öffnen
   - `sensor.helene_wischer_status` beobachten

3. **Während Befeuchtung:**
   - Status kopieren (z.B. `washing_mop`, `moistening`, `self_cleaning`)
   - Hier eintragen

**Typische Werte:**
- `washing_mop` (L10s Ultra, X40)
- `moistening` (L20 Ultra)
- `self_cleaning` (X30)
- `` (Leer = Sofort pausieren)

**⚠️ Wenn unsicher:** Einfach **leer lassen** → Roboter wird direkt nach Start pausiert (funktioniert auch, nur weniger elegant)

#### Start-Timeout
```yaml
start_timeout: 120  # Sekunden
```
**Zweck:** Max. Wartezeit bis Status von `idle` → `cleaning` wechselt

**Empfehlung:**
- 60s bei schnellen Robotern (ohne Befeuchtung)
- 120s bei Robotern mit Selbstreinigung
- 180s bei langsamen Stationen

#### Befeuchtungs-Timeout
```yaml
moistening_timeout: 60  # Sekunden
```
**Zweck:** Max. Wartezeit für Pad-Befeuchtung

**Typische Werte:**
- L10s Ultra: 30-45s
- L20 Ultra: 45-60s
- X40 Ultra: 60-90s

---

### 6️⃣ Erweiterte Optionen

#### Segment-Service nutzen
```yaml
use_segment_service: true
```

**Unterschied:**

| Methode | Service | Repeats | Vorteil |
|---------|---------|---------|---------|
| **Standard** | `vacuum.start` | Nicht möglich | Einfach, funktioniert immer |
| **Segment** | `dreame_vacuum.vacuum_clean_segment` | Pro Raum konfigurierbar | Präzise Wiederholungen |

**❓ Wann deaktivieren?**
- Roboter unterstützt `vacuum_clean_segment` nicht
- Fehler: "segments not found"
- Einfachheit gewünscht

**✅ Empfohlen:** Aktiviert lassen für beste Ergebnisse!

#### Debug-Modus
```yaml
debug_mode: true
```

**Aktiviert:**
- ✅ Persistent Notifications bei jedem Schritt
- ✅ Anzeige der Variablen-Werte
- ✅ Fehlersuche bei Problemen

**Notifications erscheinen bei:**
- Button-Druck (welche Action empfangen)
- Moduswechsel
- Kartenwechsel
- Reinigungsstart
- Unbekannte Button-Actions

**📌 Nach Setup:** Debug-Modus **deaktivieren** für saubere UI

---

## 🔄 Funktionsweise

### Button 1 & 2: Moduswechsel

```
┌─────────────┐
│  Button 1   │ → select.cleaning_mode = "sweeping"
└─────────────┘

┌─────────────┐
│  Button 2   │ → select.cleaning_mode = "sweeping_and_mopping"
└─────────────┘
```

**Einfach:** Setzt nur den Modus, startet KEINE Reinigung.

---

### Button 3: Intelligente START/PAUSE/RESUME

```
                    ┌──────────────────┐
                    │  Button 3 Press  │
                    └────────┬─────────┘
                             │
                    ┌────────▼──────────┐
                    │ Status prüfen     │
                    └────────┬──────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐      ┌────▼─────┐      ┌────▼─────┐
    │ Cleaning  │      │  Paused  │      │  Idle    │
    └─────┬─────┘      └────┬─────┘      └────┬─────┘
          │                  │                  │
     ┌────▼────┐        ┌───▼────┐        ┌────▼─────────┐
     │  PAUSE  │        │ RESUME │        │ START + LOGIC│
     └─────────┘        └────────┘        └──────────────┘
```

#### START-Logik (Wenn Status = Idle):

**Basisstations-Map:**
```
┌─────────────────────┐
│ Ist Basisstation?   │ = JA
└──────────┬──────────┘
           │
    ┌──────▼───────┐
    │ START SOFORT │  (Kein Pause nötig)
    └──────────────┘
```

**Andere Maps:**
```
┌─────────────────────┐
│ Ist Basisstation?   │ = NEIN
└──────────┬──────────┘
           │
    ┌──────▼────────────┐
    │ 1. START          │
    └──────┬────────────┘
           │
    ┌──────▼────────────────────┐
    │ 2. Warte auf "cleaning"   │ (bis zu 120s)
    └──────┬────────────────────┘
           │
    ┌──────▼──────────────────────┐
    │ 3. Wischmodus?              │
    │    → Warte auf Befeuchtung  │ (bis zu 60s)
    └──────┬──────────────────────┘
           │
    ┌──────▼────────────┐
    │ 4. PAUSE          │ ⏸️
    └───────────────────┘
           │
    ✅ Bereit zum Hochtragen!
```

---

### Button 4-6: Kartenwechsel

```
┌───────┐     ┌───────┐     ┌───────┐
│ Btn 4 │ →   │ Btn 5 │ →   │ Btn 6 │ →
└───┬───┘     └───┬───┘     └───┬───┘
    │             │             │
    ▼             ▼             ▼
  Map 1         Map 2         Map 3
```

**Einfach:** Wechselt nur die Karte, startet KEINE Reinigung.

**⚠️ Hinweis:** Roboter beendet aktive Reinigung beim Kartenwechsel!

---

## 🛠️ Troubleshooting

### Problem: Button funktioniert nicht

**1. MQTT-Topic prüfen:**
```bash
# In MQTT Explorer
Topic: zigbee2mqtt/Schalter Staubsaugroboter
Payload: {"action": "1_single", ...}
```

**2. Action-Wert prüfen:**
```yaml
button_1_action: "1_single"  # Muss EXAKT übereinstimmen
```

**3. Debug-Modus aktivieren:**
```yaml
debug_mode: true
```
→ Notification zeigt empfangene Action

---

### Problem: Reinigung pausiert sofort

**Ursache:** `moistening_status` ist leer oder falsch

**Lösung:**
1. `moistening_status` leer lassen
2. ODER: Korrekten Status ermitteln (siehe [Befeuchtungs-Status](#befeuchtungs-status-optional))

---

### Problem: "Segments not found" Fehler

**Ursache:** `use_segment_service: true`, aber keine Segment-IDs verfügbar

**Lösung A:** Segment-Service deaktivieren
```yaml
use_segment_service: false
```

**Lösung B:** Map-Camera prüfen
```bash
# Developer Tools → States
camera.helene_wischer_map

# Attribute "segments" muss existieren:
segments:
  1: "Wohnzimmer"
  2: "Küche"
  3: "Flur"
```

---

### Problem: Timeout beim Start

**Symptom:** "Timeout nach 120s"

**Ursachen:**
1. Roboter startet nicht (Fehler, leerer Akku)
2. Status-Werte falsch konfiguriert
3. Timeout zu kurz

**Lösung:**
```yaml
# Timeout erhöhen
start_timeout: 180

# Status-Werte prüfen
cleaning_states: "cleaning,returning,zone_cleaning,room_cleaning"
```

---

### Problem: Karte wechselt nicht

**Prüfung:**
```yaml
# Exakte Schreibweise?
map_1_name: "Wohnzimmer"  # ✅
map_1_name: "wohnzimmer"  # ❌ (Kleinschreibung)
map_1_name: "Wohnzimmer " # ❌ (Leerzeichen am Ende)
```

**Hilfsmittel:** Debug-Modus zeigt aktuellen Map-Namen

---

## ❓ FAQ

<details>
<summary><strong>Kann ich mehr als 6 Buttons verwenden?</strong></summary>

Ja! Blueprint ist erweiterbar:

1. Blueprint YAML öffnen
2. Neue Input-Felder hinzufügen:
```yaml
button_7_action:
  name: "Button 7 - Action Value"
  selector:
    text:
  default: "7_single"
```
3. In `variables` hinzufügen:
```yaml
btn_7: !input button_7_action
```
4. Neue `choose`-Bedingung hinzufügen:
```yaml
- conditions:
    - condition: template
      value_template: "{{ button_action == btn_7 }}"
  sequence:
    # Deine Aktion hier
```

</details>

<details>
<summary><strong>Funktioniert es mit anderen Robotern?</strong></summary>

**Prinzipiell JA**, wenn:
- ✅ Integration ähnliche Entitäten bereitstellt
- ✅ Services kompatibel sind

**Getestet mit:**
- Dreame L10s Ultra
- Dreame L20 Ultra
- Dreame X40 Ultra

**Anpassungen nötig:**
- Status-Werte können abweichen
- Service-Namen ggf. anders

</details>

<details>
<summary><strong>Kann ich andere MQTT-Geräte nutzen?</strong></summary>

Ja! Jedes MQTT-Gerät funktioniert, das JSON mit `"action"` Key sendet:

```json
{
  "action": "button_press",
  "button": 1
}
```

Dann `button_1_action: "button_press"` setzen.

</details>

<details>
<summary><strong>Warum pausiert auf Basisstation nicht?</strong></summary>

**Use Case:** Multi-Etagen ohne Lift

**Szenario:**
- Erdgeschoss: Basisstation hier → Sofort starten ✅
- Obergeschoss: Manuell hochtragen → Pause für Transport ⏸️

**Vorteil:** Ein Blueprint für alle Stockwerke!

</details>

<details>
<summary><strong>Wie viele Durchgänge sind sinnvoll?</strong></summary>

**Empfehlung nach Map:**

| Map | Durchgänge | Grund |
|-----|-----------|-------|
| **Basisstation** | 2-3x | Gründlich, da keine Transport-Zeit |
| **Andere Maps** | 1-2x | Zeitersparnis (Transport + Ladezeit) |

**Tipp:** `cleaning_repeats: 2` ist guter Mittelwert.

</details>

---

## 📝 Changelog

### Version 1.0.0 (2025-01-XX)

**Initial Release**

**Features:**
- ✅ 6-Button MQTT-Steuerung
- ✅ Intelligente Start/Pause/Resume-Logik
- ✅ Kartenbasierte Reinigung mit Repeats
- ✅ Basisstations-Erkennung
- ✅ Segment-Service-Unterstützung
- ✅ Debug-Modus
- ✅ Vollständig GUI-konfigurierbar

**Kompatibilität:**
- Home Assistant ≥ 2024.4.0
- Dreame Vacuum Integration ≥ 2.0.0
- Zigbee2MQTT (empfohlen)

---

## 🙏 Credits & Inspiration

**Entwickelt von:** Stefan  
**Inspiriert von:**
- [Blackymas](https://github.com/Blackymas) - Blueprint Best Practices
- [EPMatt](https://github.com/EPMatt) - Controller Blueprints
- [Tasshack](https://github.com/Tasshack) - Dreame Vacuum Integration

**Community:**
- Home Assistant Community Forum
- r/homeassistant
- Deutsche Home Assistant Community

---

## 📄 Lizenz

MIT License - Siehe [LICENSE](LICENSE) für Details

---

## 🔗 Links

- [GitHub Repository](#)
- [Home Assistant Community Forum](#)
- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum)
- [Zigbee2MQTT Dokumentation](https://www.zigbee2mqtt.io/)

---

## 🐛 Fehler melden / Feature-Request

**Issues:** [GitHub Issues](#)  
**Discussions:** [GitHub Discussions](#)  
**Support:** [Community Forum](#)

---

**Viel Erfolg mit deinem smarten Saugroboter! 🤖✨**
