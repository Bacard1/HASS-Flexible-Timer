
# ⏱️ HASS - ГЪВКАВА ТАЙМЕР АВТОМАТИЗАЦИЯ  
[![PayPal дарение](https://img.shields.io/badge/PayPal-Дари-синьо?logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=AAWFZVF2XCP5A)
![Скрипт](https://img.shields.io/badge/logo-yaml-green?logo=yaml)
[![English](https://img.shields.io/badge/EN_English-language-green?logo=translate&labelColor=gray&style=flat-square)](README.md)
[![Български](https://img.shields.io/badge/BG_Български-език-green?logo=translate&labelColor=gray&style=flat-square)](BG.md)
[![Home Assistant](https://img.shields.io/badge/🏠_Home_Assistant-41BDF5?logo=homeassistant)](https://www.home-assistant.io/)  

🌟 **Този проект демонстрира напълно динамична, самоизключваща се таймерна автоматизация в Home Assistant, конфигурируема през UI (часове/минути/секунди).**  

---

## 📚 Съдържание  
- [⏱️ HASS - ГЪВКАВА ТАЙМЕР АВТОМАТИЗАЦИЯ](#️-hass---гъвкава-таймер-автоматизация)
  - [📚 Съдържание](#-съдържание)
  - [📦 ОСНОВНИ ФУНКЦИИ](#-основни-функции)
  - [🔧 КОНФИГУРАЦИЯ](#-конфигурация)
    - [1. Задаване на интервал `input_number`](#1-задаване-на-интервал-input_number)
    - [2. `input_datetime`: Запазва времето на последно изпълнение](#2-input_datetime-запазва-времето-на-последно-изпълнение)
    - [3. Автоматизация с таймерна логика](#3-автоматизация-с-таймерна-логика)
    - [4. Скриптове за управление](#4-скриптове-за-управление)
    - [5. Lovelace UI пример](#5-lovelace-ui-пример)
  - [🚀 Примери за употреба](#-примери-за-употреба)
  - [💡 Професионални съвети](#-професионални-съвети)
  - [📊 Диаграма на процеса](#-диаграма-на-процеса)
  - [🎨 Визуализация на Таймер Картата](#-визуализация-на-таймер-картата)
    - [📱 Как изглежда картата?](#-как-изглежда-картата)
  - [📦 Необходими допълнителни пакети](#-необходими-допълнителни-пакети)
  - [🔍 Подробности за кода](#-подробности-за-кода)
    - [Bubble Card Конфигурация](#bubble-card-конфигурация)
    - [Numberbox Стилове](#numberbox-стилове)
    - [Условия за видимост](#условия-за-видимост)
  - [💡 Персонализиране](#-персонализиране)

---

## 📦 ОСНОВНИ ФУНКЦИИ  
- 🕒 **UI конфигурация** (часове/минути/секунди)  
- ⚡ **Самоизключване** след изпълнение  
- 🔄 **Нулиране** без рестартиране  
- 📅 **Запазва** последно изпълнение  
- 🛡️ **Предотвратява** грешни задействания  

---

## 🔧 КОНФИГУРАЦИЯ  

### 1. Задаване на интервал `input_number`  
```yaml
input_number:
  interval_hours:
    name: "Интервал - Часове"
    min: 0
    max: 24
    step: 1
    unit_of_measurement: "ч"
    
  interval_minutes:
    name: "Интервал - Минути"
    min: 0
    max: 59
    step: 1
    unit_of_measurement: "мин"
    
  interval_seconds:
    name: "Интервал - Секунди"
    min: 5  # Минимум 5 сек за стабилност
    max: 59
    step: 1
    unit_of_measurement: "сек"
```
> 💡 *Тук дефинирате интервала. Стойностите се сумират автоматично (1ч + 30мин = 5400 сек).*

---

### 2. `input_datetime`: Запазва времето на последно изпълнение  
```yaml
input_datetime:
  last_execution_time:
    name: "Последно изпълнение"
    has_date: true
    has_time: true
```
> ⏳ *Този компонент записва кога е стартирал таймерът за последен път. Критично за изчисленията!*

---

### 3. Автоматизация с таймерна логика  
```yaml
automation:
  - alias: "⏱️ ТАЙМЕР"
    id: auto_flexible_interval
    description: "Изпълнява действие след зададен интервал"
    trigger:
      - platform: time_pattern
        seconds: "/1"  # Проверява всяка секунда
    condition: >
      {% set h = states('input_number.interval_hours') | int %}
      {% set m = states('input_number.interval_minutes') | int %}
      {% set s = states('input_number.interval_seconds') | int %}
      {% set interval = h * 3600 + m * 60 + s %}
      {% set last = states('input_datetime.last_execution_time') %}
      {{ (now() - strptime(last, '%Y-%m-%d %H:%M:%S')).total_seconds() >= interval if last not in ['unknown', 'unavailable'] else false %}
    action:
      - service: notify.all_devices  # 👈 Променете с вашия сервис!
        data:
          message: "🔄 Действие изпълнено в {{ now().strftime('%H:%M:%S') }}"
          
      - service: input_datetime.set_datetime
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          
      - delay: 00:00:02  # Кратка пауза преди изключване
      
      - service: automation.turn_off  # 🔌 САМОИЗКЛЮЧВАНЕ
        target:
          entity_id: "{{ this.entity_id }}"
```
> 🔍 **Как работи?**  
> 1️⃣ Проверява всяка секунда дали е изтекъл интервалът  
> 2️⃣ Изчислява общо време в секунди  
> 3️⃣ Сравнява с последното изпълнение  
> 4️⃣ Ако условието е изпълнено → изпълнява действие и се изключва  

---

### 4. Скриптове за управление  
```yaml
script:
  start_interval_timer:
    alias: "▶️ Старт на Таймера"
    sequence:
      - service: input_datetime.set_datetime  # 📌 Задава начално време
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          
      - service: automation.turn_on  # 🚀 Пуска автоматизацията
        target:
          entity_id: automation.auto_flexible_interval

  reset_interval_timer:
    alias: "🔄 Нулиране на Таймера"
    sequence:
      - service: input_datetime.set_datetime  # 🔄 Обновява времето без спиране
        data:
          entity_id: input_datetime.last_execution_time
          datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
```

---

### 5. Lovelace UI пример  
```yaml
type: vertical-stack
cards:
  - type: entities
    title: "⏱️ Контролен Панел"
    entities:
      - input_number.interval_hours
      - input_number.interval_minutes
      - input_number.interval_seconds
      - input_datetime.last_execution_time
      
  - type: horizontal-stack
    cards:
      - type: button
        name: "▶️ Старт"
        icon: mdi:play
        tap_action:
          action: call-service
          service: script.start_interval_timer
          
      - type: button
        name: "🔄 Нулирай"
        icon: mdi:restart
        tap_action:
          action: call-service
          service: script.reset_interval_timer
          
      - type: button
        name: "⏹️ Спри"
        icon: mdi:stop
        tap_action:
          action: call-service
          service: automation.turn_off
          target:
            entity_id: automation.auto_flexible_interval
```
> 🎨 *Персонализирайте с картографи (`card-mod`) за 3D бутони и анимации!*

---

## 🚀 Примери за употреба  
- **🌡️ Климатик**: Изключване след 2 часа работа  
- **💡 Осветление**: Нощен режим (30мин)  
- **🔔 Известия**: Повикване на всеки 15 минути  

---

## 💡 Професионални съвети  
1. **⏱️ Прецизност**: Използвайте `seconds: "/1"` за секундна точност  
2. **🔄 Нулиране**: Винаги използвайте скрипта `reset_interval_timer` вместо ръчно задаване  
3. **📊 Визуализация**: Добавете `history_graph` за проследяване на изпълненията  

---

## 📊 Диаграма на процеса

![img](/img/diagram.png)

## 🎨 Визуализация на Таймер Картата

### 📱 Как изглежда картата?
Вашата таймер карта ще изглежда като модерен, полупрозрачен контролен панел с три основни секции:

1. **⏱️ Настройки на интервала**  
   - Три колони за часове, минути и секунди със стилизирани бутони +/-  
   - Показва текущо зададеното време (напр. "0h 5min 30s")  
   - Последен ред показва последно изпълнение (напр. "5/20 8:30 PM")

2. **🔼 START бутон**  
   - Зелен акцент (обикновено)  
   - Стартира таймера чрез `script.reset_interval_timer`  
   - Видим само когато автоматизацията е изключена  

3. **🔴 STOP бутон**  
   - Червен акцент  
   - Спира автоматизацията чрез `script.turn_on`  
   - Видим само когато автоматизацията е активна  

## 📦 Необходими допълнителни пакети
За да работи картата, трябва да инсталирате следните добавки:

```yaml
# В HACS (Home Assistant Community Store) трябва да имате:
- bubble-card (за pop-up ефекта)
- numberbox-card (за стилизираните цифрови полета)
- card-mod (за CSS персонализации)
- button-card (ако искате допълнителни ефекти)
```

## 🔍 Подробности за кода

### Bubble Card Конфигурация
```yaml
type: custom:bubble-card
name: TIMER
card_type: pop-up  # Създава "изскачащ" ефект
hide_backdrop: true  # Скрива фона
```

### Numberbox Стилове
```yaml
type: custom:numberbox-card
entity: input_number.interval_seconds
name: Seconds
unit: s.  # Показва единиците
card_mod:
  style: |
    ha-card {
      border-radius: 15px;  # Заоблени ъгли
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);  # 3D ефект
    }
```

### Условия за видимост
```yaml
visibility:
  - condition: state
    entity: automation.taimer
    state_not: "on"  # Показва START само когато таймерът е изключен
```

## 💡 Персонализиране
1. **Цветове**: Променете `rgba(0, 0, 0, 0.35)` на друг цвят (напр. `rgba(25, 25, 112, 0.4)` за midnight blue)
2. **Анимации**: Добавете `transition: all 0.3s ease;` за плавни ефекти при hover
3. **Икони**: Сменете `mdi:restart` с други икони от [Material Design Icons](https://materialdesignicons.com/)

Резултат:  
![Визуализация на таймера](/img/timer_card.png)

---
---
> [!TIP]
> Ако този проект ви е харесъл, [ТУК](https://github.com/Bacard1?tab=repositories) ще намерите още интересни гранилища направени от мен.<br>
> Ако срещате затруднения или имате въпроси не се колебайте да се свържете с мен.


