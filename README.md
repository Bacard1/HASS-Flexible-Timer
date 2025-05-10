![BANNER](/img/banner.png)
[![PayPal Donation](https://img.shields.io/badge/PayPal-Дари-синьо?logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=AAWFZVF2XCP5A)
![Script](https://img.shields.io/badge/logo-yaml-green?logo=yaml)
[![БЪЛГАРСКИ](https://img.shields.io/badge/БЪЛГАРСКИ-език-green?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/en)](BG.md)

# ⏱️ Home Assistant Flexible Timer Automation

> This project demonstrates how to create a fully dynamic, self-disabling timer-based automation in Home Assistant, configurable in hours, minutes, and seconds from the UI.

---

## 📚 Table of Contents

- [⏱️ Home Assistant Flexible Timer Automation](#️-home-assistant-flexible-timer-automation)
  - [📚 Table of Contents](#-table-of-contents)
  - [📦 Features](#-features)
  - [🔧 Configuration](#-configuration)
    - [1. `input_number`: User-defined time interval](#1-input_number-user-defined-time-interval)
    - [2. `input_datetime`: Stores last execution time](#2-input_datetime-stores-last-execution-time)
    - [3. Automation: Timer logic with self-disable](#3-automation-timer-logic-with-self-disable)
    - [4. Scripts: Start and Reset helpers](#4-scripts-start-and-reset-helpers)
    - [5. Lovelace UI Example](#5-lovelace-ui-example)
  - [✅ Example Use Cases](#-example-use-cases)
  - [💡 Tips \& Recommendations](#-tips--recommendations)
  - [📊 Workflow Diagram](#-workflow-diagram)

---

## 📦 Features

- 🕒 Customizable interval via `input_number` in **hours**, **minutes**, and **seconds**
- ⚙️ Executes any action at the end of the interval
- 💾 Uses `input_datetime` to store last execution time
- 🧠 Prevents premature execution with intelligent time difference check
- 🔘 Includes UI controls: Start, Stop, Reset
- ✅ Automation disables itself after each execution

---

## 🔧 Configuration

### 1. `input_number`: User-defined time interval

These helpers allow the user to set the desired interval through the UI.

```yaml
input_number:
  interval_hours:
    name: Interval Hours
    min: 0
    max: 23
    step: 1
    unit_of_measurement: "h"

  interval_minutes:
    name: Interval Minutes
    min: 0
    max: 59
    step: 1
    unit_of_measurement: "min"

  interval_seconds:
    name: Interval Seconds
    min: 5
    max: 59
    step: 1
    unit_of_measurement: "sec"
```

### 2. `input_datetime`: Stores last execution time

This stores the timestamp of the last time the automation executed. It is used to calculate whether the interval has passed.

```yaml
input_datetime:
  last_execution_time:
    name: Last Execution Time
    has_date: true
    has_time: true
```

### 3. Automation: Timer logic with self-disable

This automation checks every second if the interval has passed. If yes, it performs an action, updates the last execution time, and disables itself.

```yaml
automation:
  - alias: "⏱️TIMER"
    id: auto_flexible_interval
    description: "Automation with configurable interval (h/m/s)"
    mode: single
    max_exceeded: silent
    trigger:
      - platform: time_pattern
        seconds: "/1"
    condition:
      - condition: template
        value_template: >
          {% set h = states('input_number.interval_hours') | int %}
          {% set m = states('input_number.interval_minutes') | int %}
          {% set s = states('input_number.interval_seconds') | int %}
          {% set interval = h * 3600 + m * 60 + s %}
          {% set last = states('input_datetime.last_execution_time') %}
          {% if last not in ['unknown', 'unavailable'] %}
            {% set last_dt = strptime(last, '%Y-%m-%d %H:%M:%S') %}
            {% set delta = (now().replace(tzinfo=None) - last_dt).total_seconds() %}
            {{ delta >= interval }}
          {% else %}
            false
          {% endif %}
    action:
      - service: notify.roditeli
        data:
          title: "🕒 Timer Automation"
          message: "Executed at {{ now().strftime('%H:%M:%S') }}"

      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"

      - delay: 2

      - service: automation.turn_off
        data:
          stop_actions: true
        target:
          entity_id: "{{ this.entity_id }}"
```

### 4. Scripts: Start and Reset helpers

Scripts used to initialize or manually reset the timer.

```yaml
script:
  start_interval_timer:
    alias: "▶️ Start Interval Timer"
    sequence:
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"

      - service: automation.turn_on
        target:
          entity_id: automation.auto_flexible_interval

  reset_interval_timer:
    alias: "🔄 Reset Timer Only"
    sequence:
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
```

### 5. Lovelace UI Example

Full UI stack to control and view the timer.

```yaml
type: vertical-stack
cards:
  - type: entities
    title: ⏱️ Timer Settings
    entities:
      - input_number.interval_hours
      - input_number.interval_minutes
      - input_number.interval_seconds
      - input_datetime.last_execution_time

  - type: button
    name: ▶️ Start Timer
    icon: mdi:play
    tap_action:
      action: call-service
      service: script.start_interval_timer

  - type: button
    name: 🔄 Reset Timer
    icon: mdi:restart
    tap_action:
      action: call-service
      service: script.reset_interval_timer

  - type: button
    name: ⏹️ Stop Timer
    icon: mdi:stop
    tap_action:
      action: call-service
      service: automation.turn_off
      target:
        entity_id: automation.auto_flexible_interval
```

---

## ✅ Example Use Cases

- Turn off a device after a user-defined time delay
- Send a notification reminder after a set period
- Dynamically change automation schedules without editing YAML files

---

## 💡 Tips & Recommendations

- To improve precision, set the `trigger` to check every second: `seconds: "/1"`
- Always use `script.start_interval_timer` to begin a new interval. It sets the correct `input_datetime` value so that the automation doesn't trigger prematurely.
- You can use `reset_interval_timer` without affecting automation state (handy for testing or skipping a scheduled run).

---

## 📊 Workflow Diagram

```mermaid
graph TD
    A[User sets interval via UI] --> B[Click Start Timer (script)]
    B --> C[Record current time in input_datetime]
    C --> D[Enable automation]
    D --> E[Automation checks time every second]
    E --> F{Interval passed?}
    F -- Yes --> G[Run action (notify, etc)]
    G --> H[Update input_datetime to now]
    H --> I[Disable automation]
    F -- No --> E
```

---

> [!TIP]  
> If you enjoyed this project, you can find more interesting repositories [HERE](https://github.com/Bacard1?tab=repositories).  
> If you need help or have any questions, feel free to contact me.

