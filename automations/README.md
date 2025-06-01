# 🧲 TYPES OF AUTOMATIONS
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?color=ff00d8)](https://opensource.org/licenses/MIT)
![GitHub last commit](https://img.shields.io/github/last-commit/Bacard1/homeassistant.svg?color=ff00d8)
![Downloads](https://img.shields.io/npm/dm/homeassistant.svg?color=ff00d8)
[![hacs_badge](https://img.shields.io/badge/HACS-2025.5.3-orange.svg?color=ff00d8)](https://github.com/hacs/integration)

[![Home Assistant](https://img.shields.io/badge/.-HOME_ASSISTANT-blue?logo=homeassistant)](https://www.home-assistant.io/) 
[![Donate via PayPal](https://img.shields.io/badge/PayPal-DONATE-blue?logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=AAWFZVF2XCP5A)
![Script](https://img.shields.io/badge/Script-YAML-blue?logo=yaml)

[![Български](https://img.shields.io/badge/BG-ЕЗИК-gree?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg
)](BG.md)
[![Deutch](https://img.shields.io/badge/DE-SPRACHE-gree?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg
)](DE.md)
[![English](https://img.shields.io/badge/EN-LANGUAGE-gree?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg)](README.md)

The type of automation is defined by the `mode:` section at its end. Depending on the value of `mode:`, it changes how the automation behaves during execution. This doesn't affect simple automations much, but is crucial for complex or frequently triggered ones.

---

## 📦 CONTENTS

- [🧲 TYPES OF AUTOMATIONS](#-types-of-automations)
	- [📦 CONTENTS](#-contents)
	- [🚀 TYPES](#-types)
	- [⚗️ single](#️-single)
		- [📊 What the diagram shows:](#-what-the-diagram-shows)
		- [❗ What it means in practice:](#-what-it-means-in-practice)
		- [📌 Example scenario:](#-example-scenario)
	- [⚗️ restart](#️-restart)
		- [🔁 What the diagram shows:](#-what-the-diagram-shows-1)
		- [🔄 Example scenario:](#-example-scenario-1)
	- [⚗️ queued](#️-queued)
		- [📊 What the diagram shows:](#-what-the-diagram-shows-2)
		- [📌 Example scenario:](#-example-scenario-2)
	- [⚗️ parallel](#️-parallel)
		- [📊 What the diagram shows:](#-what-the-diagram-shows-3)
		- [⚡ What happens in practice:](#-what-happens-in-practice)
		- [📌 Example scenario:](#-example-scenario-3)
	- [⚗️ queued with max:](#️-queued-with-max)
		- [How it works:](#how-it-works)
		- [Scenario:](#scenario)
	- [⚗️ parallel with max:](#️-parallel-with-max)
		- [How it works:](#how-it-works-1)
		- [Scenario:](#scenario-1)

---

## 🚀 TYPES

| `mode`              | Description                                                                                 | Usage Example                                                                  |
|---------------------|---------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| `single`            | **Only one execution** is allowed. New triggers are **ignored** while one is running.       | An alarm that shouldn't restart until the previous one has completed.          |
| `restart`           | A new trigger **cancels the current execution** and starts from the beginning.              | A light timer – restarts on new motion.                                        |
| `queued`            | Each new trigger is **queued** and runs after the previous finishes.                        | Notifications sent to multiple users sequentially.                             |
| `parallel`          | Allows **multiple executions simultaneously**.                                               | Turning on lights in different rooms independently.                            |
| `queued` with `max:`| Limited queue – e.g., `queued` with `max: 3` allows up to 3 waiting executions.              | Protection from overload in rapid triggering situations.                       |
| `parallel` with `max:` | Limits number of concurrent executions – e.g., `parallel` with `max: 2`.                    | Only 2 alarms can play at the same time, others are ignored.                   |

---

## ⚗️ single

> Diagram:

![single diagramm](/automations/img/single_diagramm.png)

The `single` mode diagram illustrates the simplest automation behavior in Home Assistant.

### 📊 What the diagram shows:

- Trigger 1 — starts the automation and executes the actions.
- Trigger 2 and 3 — are ignored while the automation is still running.
- No queue or parallelism — only one active session at a time.

### ❗ What it means in practice:

- If the automation is already running, new triggers are ignored.
- No restart, no queuing, no parallel execution.
- This avoids overload, but can miss important events.

### 📌 Example scenario:

```yaml
alias: "Entrance Notification"
trigger:
  - platform: state
    entity_id: binary_sensor.door
    to: "on"
action:
  - service: media_player.play_media
    data:
      entity_id: media_player.living_room_speaker
      media_content_id: "ding dong"
      media_content_type: "music"
  - delay: "00:00:10"
mode: single
```

## ⚗️ restart

> Diagram:

![restart diagramm](/automations/img/restart_digramm.png)

### 🔁 What the diagram shows:

- A trigger starts the automation.
- If another trigger comes before the actions finish, the current run is stopped and restarted from the beginning.

### 🔄 Example scenario:

```yaml
- service: light.turn_on
- delay: "00:01:00"
- service: light.turn_off
```

A new motion event within the 1-minute delay will restart the automation from the top.

## ⚗️ queued
> Diagram:

![queued diagramm](/automations/img/queued_diagramm.png)

### 📊 What the diagram shows:

- Every new trigger gets queued if the automation is still running.
- Each queue item is executed sequentially.
- `max:` defines the maximum length of the queue.

### 📌 Example scenario:

```yaml
- service: notify.mobile_app
  data:
    message: "Door is open!"
- delay: "00:00:10"
```

With 5 fast door triggers and `max: 3`:

- 1 is executed
- 3 are queued
- The 5th is discarded

## ⚗️ parallel

> Diagram:

![parallel diagramm](/automations/img/parallel_diagramm.png)

### 📊 What the diagram shows:

- All triggers execute immediately in parallel.
- No waiting or stopping ongoing actions.

### ⚡ What happens in practice:

- Unlimited executions by default.
- Can overload the system if not monitored.

### 📌 Example scenario:

```yaml
alias: "Motion Light"
trigger:
  - platform: state
    entity_id: binary_sensor.motion
    to: "on"
action:
  - service: light.turn_on
    data:
      entity_id: light.hall
  - delay: "00:00:30"
  - service: light.turn_off
    data:
      entity_id: light.hall
mode: parallel
```

Each motion trigger will run its own 30-second light cycle.

## ⚗️ queued with max:

> Diagram:

![queued с max:](/automations/img/queued_max_diagramm.png)

### How it works:
```yaml
mode: queued
max: 3
```

- Adds new triggers to a queue.
- Maximum 3 triggers can wait.
- Others are discarded.

### Scenario:

Avoid overloading when a door is frequently opened/closed (e.g. children playing).

## ⚗️ parallel with max:

> Diagram:

![parallel с max:](/automations/img/paralel_max_diagramm.png)

### How it works:

```yaml
mode: parallel
max: 2
```

- Multiple runs allowed, but only 2 at the same time.
- Extra triggers are ignored.

### Scenario:

Two users can trigger an alarm in different rooms — more than 2 won’t start new alarms.

---
---
> [!TIP]
> If you like this project, check out [more of my repositories here](https://github.com/Bacard1?tab=repositories).  
> If you need help or have questions, feel free to contact me.
