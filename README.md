# ⏱️ HASS - FLEXIBLE TIMER AUTOMATION  
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

🌟 **This project demonstrates a fully dynamic, self-disabling timer automation in Home Assistant, configurable via UI (hours/minutes/seconds).**  

---

## 📚 Table of Contents  
- [⏱️ HASS - FLEXIBLE TIMER AUTOMATION](#️-hass---flexible-timer-automation)
  - [📚 Table of Contents](#-table-of-contents)
  - [📦 KEY FEATURES](#-key-features)
  - [🔧 CONFIGURATION](#-configuration)
    - [1. Interval Setting `input_number`](#1-interval-setting-input_number)
    - [2. `input_datetime`: Store Last Execution Time](#2-input_datetime-store-last-execution-time)
    - [3. Timer Logic Automation](#3-timer-logic-automation)
    - [4. Control Scripts](#4-control-scripts)
    - [5. Lovelace UI Example](#5-lovelace-ui-example)
  - [🚀 Usage Examples](#-usage-examples)
  - [💡 Pro Tips](#-pro-tips)
  - [📊 Process Diagram](#-process-diagram)
  - [🎨 Timer Card Visualization](#-timer-card-visualization)
    - [📱 What Does the Card Look Like?](#-what-does-the-card-look-like)
  - [📦 Required Additional Packages](#-required-additional-packages)
  - [🔍 Code Details](#-code-details)
    - [Bubble Card Configuration](#bubble-card-configuration)
    - [Numberbox Styles](#numberbox-styles)
    - [Visibility Conditions](#visibility-conditions)
  - [💡 Customization](#-customization)

---

## 📦 KEY FEATURES  
- 🕒 **UI Configuration** (hours/minutes/seconds)  
- ⚡ **Self-disabling** after execution  
- 🔄 **Reset** without restart  
- 📅 **Stores** last execution time  
- 🛡️ **Prevents** false triggers  

---

## 🔧 CONFIGURATION  

### 1. Interval Setting `input_number`  
```yaml
input_number:
  interval_hours:
    name: "Interval - Hours"
    min: 0
    max: 24
    step: 1
    unit_of_measurement: "h"
    
  interval_minutes:
    name: "Interval - Minutes"
    min: 0
    max: 59
    step: 1
    unit_of_measurement: "min"
    
  interval_seconds:
    name: "Interval - Seconds"
    min: 5  # Minimum 5 sec for stability
    max: 59
    step: 1
    unit_of_measurement: "sec"
```
> 💡 *Defines the interval. Values are summed automatically (1h + 30min = 5400 sec).*

---

### 2. `input_datetime`: Store Last Execution Time  
```yaml
input_datetime:
  last_execution_time:
    name: "Last Execution"
    has_date: true
    has_time: true
```
> ⏳ *This component records when the timer last started. Critical for calculations!*

---

### 3. Timer Logic Automation  
```yaml
automation:
  - alias: "⏱️ TIMER"
    id: auto_flexible_interval
    description: "Executes action after set interval"
    trigger:
      - platform: time_pattern
        seconds: "/1"  # Checks every second
    condition: >
      {% set h = states('input_number.interval_hours') | int %}
      {% set m = states('input_number.interval_minutes') | int %}
      {% set s = states('input_number.interval_seconds') | int %}
      {% set interval = h * 3600 + m * 60 + s %}
      {% set last = states('input_datetime.last_execution_time') %}
      {{ (now() - strptime(last, '%Y-%m-%d %H:%M:%S')).total_seconds() >= interval if last not in ['unknown', 'unavailable'] else false %}
    action:
      - service: notify.all_devices  # 👈 Replace with your service!
        data:
          message: "🔄 Action executed at {{ now().strftime('%H:%M:%S') }}"
          
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          
      - delay: 00:00:02  # Brief pause before disabling
      
      - service: automation.turn_off  # 🔌 SELF-DISABLING
        target:
          entity_id: "{{ this.entity_id }}"
```
> 🔍 **How It Works?**  
> 1️⃣ Checks every second if interval has elapsed  
> 2️⃣ Calculates total time in seconds  
> 3️⃣ Compares with last execution  
> 4️⃣ If condition met → executes action and disables itself  

---

### 4. Control Scripts  
```yaml
script:
  start_interval_timer:
    alias: "▶️ Start Timer"
    sequence:
      - service: input_datetime.set_datetime  # 📌 Sets initial time
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          
      - service: automation.turn_on  # 🚀 Starts automation
        target:
          entity_id: automation.auto_flexible_interval

  reset_interval_timer:
    alias: "🔄 Reset Timer"
    sequence:
      - service: input_datetime.set_datetime  # 🔄 Updates time without stopping
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
```

---

### 5. Lovelace UI Example  
```yaml
type: vertical-stack
cards:
  - type: entities
    title: "⏱️ Control Panel"
    entities:
      - input_number.interval_hours
      - input_number.interval_minutes
      - input_number.interval_seconds
      - input_datetime.last_execution_time
      
  - type: horizontal-stack
    cards:
      - type: button
        name: "▶️ START"
        icon: mdi:play
        tap_action:
          action: call-service
          service: script.start_interval_timer
          
      - type: button
        name: "🔄 RESET"
        icon: mdi:restart
        tap_action:
          action: call-service
          service: script.reset_interval_timer
          
      - type: button
        name: "⏹️ STOP"
        icon: mdi:stop
        tap_action:
          action: call-service
          service: automation.turn_off
          target:
            entity_id: automation.auto_flexible_interval
```
> 🎨 *Customize with `card-mod` for 3D buttons and animations!*

---

## 🚀 Usage Examples  
- **🌡️ AC**: Turn off after 2 hours  
- **💡 Lights**: Night mode (30min)  
- **🔔 Notifications**: Reminder every 15 minutes  

---

## 💡 Pro Tips  
1. **⏱️ Precision**: Use `seconds: "/1"` for second-level accuracy  
2. **🔄 Reset**: Always use `reset_interval_timer` script instead of manual setting  
3. **📊 Visualization**: Add `history_graph` to track executions  

---

## 📊 Process Diagram

![img](/img/diagram.png)

## 🎨 Timer Card Visualization

### 📱 What Does the Card Look Like?
Your timer card will appear as a modern, semi-transparent control panel with three main sections:

1. **⏱️ Interval Settings**  
   - Three columns for hours/minutes/seconds with styled +/- buttons  
   - Shows current set time (e.g., "0h 5min 30s")  
   - Bottom row shows last execution (e.g., "5/20 8:30 PM")

2. **🔼 START Button**  
   - Green accent (typically)  
   - Starts timer via `script.reset_interval_timer`  
   - Visible only when automation is off  

3. **🔴 STOP Button**  
   - Red accent  
   - Stops automation via `script.turn_on`  
   - Visible only when automation is active  

## 📦 Required Additional Packages
To make the card work, install these add-ons:

```yaml
# In HACS (Home Assistant Community Store):
- bubble-card (for pop-up effect)
- numberbox-card (for styled numeric inputs)
- card-mod (for CSS customization)
- button-card (for additional effects)
```

## 🔍 Code Details

### Bubble Card Configuration
```yaml
type: custom:bubble-card
name: TIMER
card_type: pop-up  # Creates "pop-out" effect
hide_backdrop: true  # Hides background
```

### Numberbox Styles
```yaml
type: custom:numberbox-card
entity: input_number.interval_seconds
name: Seconds
unit: s.  # Shows units
card_mod:
  style: |
    ha-card {
      border-radius: 15px;  # Rounded corners
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);  # 3D effect
    }
```

### Visibility Conditions
```yaml
visibility:
  - condition: state
    entity: automation.taimer
    state_not: "on"  # Shows START only when timer is off
```

## 💡 Customization
1. **Colors**: Change `rgba(0, 0, 0, 0.35)` to other colors (e.g., `rgba(25, 25, 112, 0.4)` for midnight blue)
2. **Animations**: Add `transition: all 0.3s ease;` for smooth hover effects
3. **Icons**: Replace `mdi:restart` with other icons from [Material Design Icons](https://materialdesignicons.com/)

Result:  
![Timer Visualization](/img/timer_card.png)

---
> [!TIP]
> If you like this project, find more interesting repositories [HERE](https://github.com/Bacard1?tab=repositories).  
> For questions or issues, don't hesitate to contact me.