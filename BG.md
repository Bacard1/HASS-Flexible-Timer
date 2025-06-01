![BANNER](/img/banner.png)
[![PayPal дарение](https://img.shields.io/badge/PayPal-Дари-синьо?logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=AAWFZVF2XCP5A)
![Скрипт](https://img.shields.io/badge/logo-yaml-green?logo=yaml)
[![English](https://img.shields.io/badge/ENGLISH-language-green?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/en)](README.md)


# ⏱️ Home Assistant - Гъвкава Таймер Автоматизация

> Този проект демонстрира как да създадете напълно динамична, самоизключваща се таймерна автоматизация в Home Assistant, конфигурируема през потребителския интерфейс (ч/мин/сек).

---

## 📚 Съдържание

- [⏱️ Home Assistant - Гъвкава Таймер Автоматизация](#️-home-assistant---гъвкава-таймер-автоматизация)
  - [📚 Съдържание](#-съдържание)
  - [📦 Основни функции](#-основни-функции)
  - [🔧 Конфигурация](#-конфигурация)
    - [1. `input_number`: Задаване на интервал](#1-input_number-задаване-на-интервал)
    - [2. `input_datetime`: Съхранява времето на последно изпълнение](#2-input_datetime-съхранява-времето-на-последно-изпълнение)
    - [3. Автоматизация с таймерна логика и самоизключване](#3-автоматизация-с-таймерна-логика-и-самоизключване)
    - [4. Скриптове за старт и нулиране](#4-скриптове-за-старт-и-нулиране)
    - [5. Lovelace UI пример](#5-lovelace-ui-пример)
  - [✅ Примери за употреба](#-примери-за-употреба)
  - [💡 Съвети](#-съвети)
  - [📊 Диаграма на процеса](#-диаграма-на-процеса)

---

## 📦 Основни функции

- 🕒 Задаване на интервал през UI (часове, минути, секунди)
- ⚙️ Изпълнение на действия след края на интервала
- 💾 Съхранява време на последно изпълнение
- 🧠 Предотвратява преждевременно задействане
- 🔘 Контроли през интерфейса: Старт, Стоп, Нулиране
- ✅ Автоматизацията се самоизключва след изпълнение

---

## 🔧 Конфигурация

### 1. `input_number`: Задаване на интервал

```yaml
input_number:
  interval_hours:
    name: Интервал - Часове
    min: 0
    max: 23
    step: 1
    unit_of_measurement: "ч"

  interval_minutes:
    name: Интервал - Минути
    min: 0
    max: 59
    step: 1
    unit_of_measurement: "мин"

  interval_seconds:
    name: Интервал - Секунди
    min: 5
    max: 59
    step: 1
    unit_of_measurement: "сек"
```

### 2. `input_datetime`: Съхранява времето на последно изпълнение

```yaml
input_datetime:
  last_execution_time:
    name: Последно изпълнение
    has_date: true
    has_time: true
```

### 3. Автоматизация с таймерна логика и самоизключване

```yaml
automation:
  - alias: "⏱️ТАЙМЕР"
    id: auto_flexible_interval
    description: "Автоматизация с интервал в ч/мин/сек"
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
          title: "🕒 Таймер Автоматизация"
          message: "Изпълнено в {{ now().strftime('%H:%M:%S') }}"

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

### 4. Скриптове за старт и нулиране

```yaml
script:
  start_interval_timer:
    alias: "▶️ Стартирай Таймера"
    sequence:
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"

      - service: automation.turn_on
        target:
          entity_id: automation.auto_flexible_interval

  reset_interval_timer:
    alias: "🔄 Нулирай Таймера"
    sequence:
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
```

### 5. Lovelace UI пример

```yaml
type: vertical-stack
cards:
  - type: entities
    title: ⏱️ Настройки на Таймера
    entities:
      - input_number.interval_hours
      - input_number.interval_minutes
      - input_number.interval_seconds
      - input_datetime.last_execution_time

  - type: button
    name: ▶️ Старт
    icon: mdi:play
    tap_action:
      action: call-service
      service: script.start_interval_timer

  - type: button
    name: 🔄 Нулирай
    icon: mdi:restart
    tap_action:
      action: call-service
      service: script.reset_interval_timer

  - type: button
    name: ⏹️ Спри
    icon: mdi:stop
    tap_action:
      action: call-service
      service: automation.turn_off
      target:
        entity_id: automation.auto_flexible_interval
```

---

## ✅ Примери за употреба

- Изключване на устройство след зададено време
- Изпращане на известие след определен интервал
- Промяна на таймер без редакция на YAML

---

## 💡 Съвети

- Използвай `seconds: "/1"` за максимална точност
- Винаги използвай `start_interval_timer`, за да се зададе начално време
- Скриптът `reset_interval_timer` е идеален за нулиране без рестарт

---

## 📊 Диаграма на процеса

```yaml
graph TD
    A[Потребител задава интервал] --> B[Натиска Старт (скрипт)]
    B --> C[Записва текущо време]
    C --> D[Активира автоматизацията]
    D --> E[Автоматизация проверява всяка секунда]
    E --> F{Интервалът изтече?}
    F -- Да --> G[Изпълнява действие]
    G --> H[Обновява времето]
    H --> I[Изключва автоматизацията]
    F -- Не --> E
```

---
> [!TIP]
> Ако този проект ви е харесъл, [ТУК](https://github.com/Bacard1?tab=repositories) ще намерите още интересни гранилища направени от мен.<br>
> Ако срещате затруднения или имате въпроси не се колебайте да се свържете с мен.


