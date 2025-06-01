# ⏱️ HASS - FLEXIBLE TIMER-AUTOMATISIERUNG  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?color=ff00d8)](https://opensource.org/licenses/MIT)
![GitHub last commit](https://img.shields.io/github/last-commit/Bacard1/HASS-Flexible-Timer.svg?color=ff00d8)
[![hacs_badge](https://img.shields.io/badge/HACS-2025.5.3-orange.svg?color=ff00d8)](https://github.com/hacs/integration)

[![Home Assistant](https://img.shields.io/badge/.-HOME_ASSISTANT-blue?logo=homeassistant)](https://www.home-assistant.io/) 
[![Donate via PayPal](https://img.shields.io/badge/PayPal-DONATE-blue?logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=AAWFZVF2XCP5A)
![Script](https://img.shields.io/badge/Script-YAML-blue?logo=yaml)

[![Български](https://img.shields.io/badge/BG-ЕЗИК-gree?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg
)](BG.md)
[![Deutch](https://img.shields.io/badge/DE-SPRACHE-gree?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg
)](DE.md)
[![English](https://img.shields.io/badge/EN-LANGUAGE-gree?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg)](README.md)


🌟 **Dieses Projekt demonstriert eine vollständig dynamische, selbstausschaltende Timer-Automatisierung in Home Assistant, konfigurierbar über die Benutzeroberfläche (Stunden/Minuten/Sekunden).**  

---

## 📚 Inhaltsverzeichnis  
- [⏱️ HASS - FLEXIBLE TIMER-AUTOMATISIERUNG](#️-hass---flexible-timer-automatisierung)
  - [📚 Inhaltsverzeichnis](#-inhaltsverzeichnis)
  - [📦 KERN-FUNKTIONEN](#-kern-funktionen)
  - [🔧 KONFIGURATION](#-konfiguration)
    - [1. Intervall einstellen `input_number`](#1-intervall-einstellen-input_number)
    - [2. `input_datetime`: Speichert die Zeit der letzten Ausführung](#2-input_datetime-speichert-die-zeit-der-letzten-ausführung)
    - [3. Automatisierung mit Timer-Logik](#3-automatisierung-mit-timer-logik)
    - [4. Skripte zur Steuerung](#4-skripte-zur-steuerung)
    - [5. Lovelace UI Beispiel](#5-lovelace-ui-beispiel)
  - [🚀 Anwendungsbeispiele](#-anwendungsbeispiele)
  - [💡 Professionelle Tipps](#-professionelle-tipps)
  - [📊 Prozessdiagramm](#-prozessdiagramm)
  - [🎨 Timer-Kartenvisualisierung](#-timer-kartenvisualisierung)
    - [📱 Wie sieht die Karte aus?](#-wie-sieht-die-karte-aus)
  - [📦 Erforderliche Zusatzpakete](#-erforderliche-zusatzpakete)
  - [🔍 Code-Details](#-code-details)
    - [Bubble Card Konfiguration](#bubble-card-konfiguration)
    - [Numberbox Stile](#numberbox-stile)
    - [Sichtbarkeitsbedingungen](#sichtbarkeitsbedingungen)
  - [💡 Anpassung](#-anpassung)

---

## 📦 KERN-FUNKTIONEN  
- 🕒 **UI-Konfiguration** (Stunden/Minuten/Sekunden)  
- ⚡ **Selbstausschaltung** nach Ausführung  
- 🔄 **Zurücksetzen** ohne Neustart  
- 📅 **Speichert** letzte Ausführung  
- 🛡️ **Verhindert** fehlerhafte Auslösungen  

---

## 🔧 KONFIGURATION  

### 1. Intervall einstellen `input_number`  
```yaml
input_number:
  interval_hours:
    name: "Intervall - Stunden"
    min: 0
    max: 24
    step: 1
    unit_of_measurement: "h"
    
  interval_minutes:
    name: "Intervall - Minuten"
    min: 0
    max: 59
    step: 1
    unit_of_measurement: "min"
    
  interval_seconds:
    name: "Intervall - Sekunden"
    min: 5  # Minimum 5 Sekunden für Stabilität
    max: 59
    step: 1
    unit_of_measurement: "sek"
```
> 💡 *Hier definieren Sie das Intervall. Die Werte werden automatisch summiert (1h + 30min = 5400 sek).*

---

### 2. `input_datetime`: Speichert die Zeit der letzten Ausführung  
```yaml
input_datetime:
  last_execution_time:
    name: "Letzte Ausführung"
    has_date: true
    has_time: true
```
> ⏳ *Diese Komponente zeichnet auf, wann der Timer zuletzt gestartet wurde. Kritisch für die Berechnungen!*

---

### 3. Automatisierung mit Timer-Logik  
```yaml
automation:
  - alias: "⏱️ TIMER"
    id: auto_flexible_interval
    description: "Führt eine Aktion nach einem festgelegten Intervall aus"
    trigger:
      - platform: time_pattern
        seconds: "/1"  # Überprüft jede Sekunde
    condition: >
      {% set h = states('input_number.interval_hours') | int %}
      {% set m = states('input_number.interval_minutes') | int %}
      {% set s = states('input_number.interval_seconds') | int %}
      {% set interval = h * 3600 + m * 60 + s %}
      {% set last = states('input_datetime.last_execution_time') %}
      {{ (now() - strptime(last, '%Y-%m-%d %H:%M:%S')).total_seconds() >= interval if last not in ['unknown', 'unavailable'] else false %}
    action:
      - service: notify.all_devices  # 👈 Ändern Sie dies zu Ihrem Service!
        data:
          message: "🔄 Aktion ausgeführt um {{ now().strftime('%H:%M:%S') }}"
          
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          
      - delay: 00:00:02  # Kurze Pause vor dem Ausschalten
      
      - service: automation.turn_off  # 🔌 SELBSTAUSSCHALTUNG
        target:
          entity_id: "{{ this.entity_id }}"
```
> 🔍 **Wie funktioniert es?**  
> 1️⃣ Überprüft jede Sekunde, ob das Intervall abgelaufen ist  
> 2️⃣ Berechnet die Gesamtzeit in Sekunden  
> 3️⃣ Vergleicht mit der letzten Ausführung  
> 4️⃣ Wenn die Bedingung erfüllt ist → führt die Aktion aus und schaltet sich aus  

---

### 4. Skripte zur Steuerung  
```yaml
script:
  start_interval_timer:
    alias: "▶️ Timer starten"
    sequence:
      - service: input_datetime.set_datetime  # 📌 Setzt die Startzeit
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          
      - service: automation.turn_on  # 🚀 Startet die Automatisierung
        target:
          entity_id: automation.auto_flexible_interval

  reset_interval_timer:
    alias: "🔄 Timer zurücksetzen"
    sequence:
      - service: input_datetime.set_datetime  # 🔄 Aktualisiert die Zeit ohne Stopp
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
```

---

### 5. Lovelace UI Beispiel  
```yaml
type: vertical-stack
cards:
  - type: entities
    title: "⏱️ Kontrollpanel"
    entities:
      - input_number.interval_hours
      - input_number.interval_minutes
      - input_number.interval_seconds
      - input_datetime.last_execution_time
      
  - type: horizontal-stack
    cards:
      - type: button
        name: "▶️ Start"
        icon: mdi:play
        tap_action:
          action: call-service
          service: script.start_interval_timer
          
      - type: button
        name: "🔄 Zurücksetzen"
        icon: mdi:restart
        tap_action:
          action: call-service
          service: script.reset_interval_timer
          
      - type: button
        name: "⏹️ Stopp"
        icon: mdi:stop
        tap_action:
          action: call-service
          service: automation.turn_off
          target:
            entity_id: automation.auto_flexible_interval
```
> 🎨 *Passen Sie es mit Kartenmodulen (`card-mod`) für 3D-Buttons und Animationen an!*

---

## 🚀 Anwendungsbeispiele  
- **🌡️ Klimaanlage**: Ausschalten nach 2 Stunden Betrieb  
- **💡 Beleuchtung**: Nachtmodus (30min)  
- **🔔 Benachrichtigungen**: Erinnerung alle 15 Minuten  

---

## 💡 Professionelle Tipps  
1. **⏱️ Präzision**: Verwenden Sie `seconds: "/1"` für sekundengenaue Präzision  
2. **🔄 Zurücksetzen**: Nutzen Sie immer das Skript `reset_interval_timer` statt manueller Einstellung  
3. **📊 Visualisierung**: Fügen Sie `history_graph` hinzu, um Ausführungen zu verfolgen  

---

## 📊 Prozessdiagramm

![img](/img/diagram.png)

## 🎨 Timer-Kartenvisualisierung

### 📱 Wie sieht die Karte aus?
Ihre Timer-Karte wird wie ein modernes, halbtransparentes Kontrollpanel mit drei Hauptabschnitten aussehen:

1. **⏱️ Intervall-Einstellungen**  
   - Drei Spalten für Stunden, Minuten und Sekunden mit stilisierten +/- Buttons  
   - Zeigt die aktuell eingestellte Zeit an (z.B. "0h 5min 30s")  
   - Letzte Zeile zeigt die letzte Ausführung (z.B. "20.05. 20:30")

2. **🔼 START-Button**  
   - Grüner Akzent (standardmäßig)  
   - Startet den Timer über `script.reset_interval_timer`  
   - Nur sichtbar, wenn die Automatisierung ausgeschaltet ist  

3. **🔴 STOP-Button**  
   - Roter Akzent  
   - Stoppt die Automatisierung über `script.turn_on`  
   - Nur sichtbar, wenn die Automatisierung aktiv ist  

## 📦 Erforderliche Zusatzpakete
Für die Funktionalität der Karte müssen folgende Add-ons installiert sein:

```yaml
# In HACS (Home Assistant Community Store) benötigen Sie:
- bubble-card (für Pop-up-Effekte)
- numberbox-card (für stilisierte numerische Felder)
- card-mod (für CSS-Anpassungen)
- button-card (für zusätzliche Effekte)
```

## 🔍 Code-Details

### Bubble Card Konfiguration
```yaml
type: custom:bubble-card
name: TIMER
card_type: pop-up  # Erzeugt einen "Pop-up"-Effekt
hide_backdrop: true  # Blendet den Hintergrund aus
```

### Numberbox Stile
```yaml
type: custom:numberbox-card
entity: input_number.interval_seconds
name: Sekunden
unit: s.  # Zeigt die Einheiten an
card_mod:
  style: |
    ha-card {
      border-radius: 15px;  # Abgerundete Ecken
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);  # 3D-Effekt
    }
```

### Sichtbarkeitsbedingungen
```yaml
visibility:
  - condition: state
    entity: automation.taimer
    state_not: "on"  # Zeigt START nur an, wenn der Timer ausgeschaltet ist
```

## 💡 Anpassung
1. **Farben**: Ändern Sie `rgba(0, 0, 0, 0.35)` zu einer anderen Farbe (z.B. `rgba(25, 25, 112, 0.4)` für Midnight Blue)
2. **Animationen**: Fügen Sie `transition: all 0.3s ease;` für sanfte Hover-Effekte hinzu
3. **Icons**: Ersetzen Sie `mdi:restart` mit anderen Icons von [Material Design Icons](https://materialdesignicons.com/)

Ergebnis:  
![Timer-Visualisierung](/img/timer_card.png)

---
---
> [!TIP]
> Wenn Ihnen dieses Projekt gefallen hat, finden Sie [HIER](https://github.com/Bacard1?tab=repositories) weitere interessante Repositories von mir.<br>
> Wenn Sie Schwierigkeiten haben oder Fragen haben, zögern Sie nicht, mich zu kontaktieren.
[file content end]