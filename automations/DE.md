# 🧲 AUTOMATISIERUNGSARTEN
[![Home Assistant](https://img.shields.io/badge/🏠_Home_Assistant-41BDF5?logo=homeassistant)](https://www.home-assistant.io/) [![Donate via PayPal](https://img.shields.io/badge/PayPal-Donate-blue?logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=AAWFZVF2XCP5A)
![Script](https://img.shields.io/badge/logo-yaml-green?logo=yaml)
[![Български](https://img.shields.io/badge/BG_Български-език-green?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg
)](BG.md)
[![Deutch](https://img.shields.io/badge/DE_Deutsche-sprache-green?logo=translate&labelColor=gray&style=flat-square&link=https://example.com/bg
)](DE.md)
[![English](https

Der Typ der Automatisierung wird durch den Abschnitt "mode:" in ihrer Konfiguration bestimmt. Abhängig davon, welchen "mode:" Sie festlegen, ändert sich das Verhalten und die Ausführung. Dies hat jedoch wenig Auswirkung auf Automatisierungen mit wenigen Komponenten, aber bei vielen Komponenten ist es von großer Bedeutung.

---

## 📦 INHALT

- [🧲 AUTOMATISIERUNGSARTEN](#-automatisierungsarten)
	- [📦 INHALT](#-inhalt)
	- [🚀 ARTEN](#-arten)
	- [⚗️ single](#️-single)
		- [📊 Was zeigt es genau:](#-was-zeigt-es-genau)
		- [❗ Was bedeutet das in der Praxis:](#-was-bedeutet-das-in-der-praxis)
		- [📌 Beispielszenario:](#-beispielszenario)
	- [⚗️ restart](#️-restart)
		- [🔁 Was zeigt das Diagramm:](#-was-zeigt-das-diagramm)
		- [🔄 Beispielszenario:](#-beispielszenario-1)
	- [⚗️ queued](#️-queued)
		- [📊 Was zeigt das Diagramm genau:](#-was-zeigt-das-diagramm-genau)
		- [📌 Beispielszenario:](#-beispielszenario-2)
	- [⚗️ parallel](#️-parallel)
		- [📊 Was zeigt das Diagramm genau:](#-was-zeigt-das-diagramm-genau-1)
		- [⚡ Was passiert in der Praxis:](#-was-passiert-in-der-praxis)
		- [📌 Beispielszenario:](#-beispielszenario-3)
	- [⚗️ queued mit max:](#️-queued-mit-max)
		- [Wie es funktioniert:](#wie-es-funktioniert)
		- [Szenario:](#szenario)
	- [⚗️ parallel mit max:](#️-parallel-mit-max)
		- [Wie es funktioniert:](#wie-es-funktioniert-1)
		- [Szenario:](#szenario-1)

---

## 🚀 ARTEN

| `mode`              | Beschreibung                                                                                       | Beispiel für die Anwendung                                                            |
| ------------------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `single`            | **Nur eine Ausführung** ist erlaubt. Ein neuer Trigger, während die aktuelle Ausführung läuft, wird **ignoriert**. | Ein Alarm, der nicht erneut ausgelöst werden soll, solange der aktuelle Alarm aktiv ist. |
| `restart`           | Ein neuer Trigger **stoppt die aktuelle Ausführung** und startet sie von Neuem.                          | Ein Timer für eine Lampe – bei neuer Bewegung wird der Timer neu gestartet.                   |
| `queued`            | Löst eine neue Instanz aus und stellt sie in eine **Warteschlange**, die nach der vorherigen Instanz ausgeführt wird.         | Nachrichten an verschiedene Benutzer nacheinander, ohne dass sie verloren gehen.             |
| `parallel`          | Erlaubt **mehrere gleichzeitige Ausführungen**. Alle Instanzen werden parallel ausgeführt.      | Unabhängige Aktionen – zum Beispiel das Einschalten von Lichtern in verschiedenen Räumen.         |
| `queued` mit `max:`   | Begrenzte Warteschlange – z.B. `queued` mit `max: 3` erlaubt bis zu 3 wartende Ausführungen.          | Schutz vor Überlastung bei häufigen Ereignissen.                                     |
| `parallel` mit `max:` | Begrenzte Anzahl gleichzeitiger Ausführungen – z.B. `parallel` mit `max: 2`.                          | Maximal 2 Alarme gleichzeitig, weitere werden ignoriert.                          |

---

## ⚗️ single
---
> Diagramm:

![single diagramm](/automations/img/single_diagramm.png) 

Das Diagramm für mode: single zeigt den einfachsten Betriebsmodus einer Automatisierung in Home Assistant.

### 📊 Was zeigt es genau:
Trigger 1 — der erste Trigger startet die Automatisierung, die Aktionen (im Feld Actions) werden ausgeführt. <br>
Trigger 2 und Trigger 3 — weitere Trigger, während die Automatisierung noch läuft, werden ignoriert. <br>
Das Diagramm enthält keine Warteschlange oder Parallelität — es gibt nur eine aktive Sitzung gleichzeitig. <br>

### ❗ Was bedeutet das in der Praxis:
Wenn die Automatisierung bereits läuft, wird jeder neue Trigger ignoriert.<br>
Es gibt keinen Neustart, kein Warten, keine parallele Ausführung.<br>
Dies ist die sicherste Methode, um Überlastung zu vermeiden, aber wichtige Ereignisse könnten übersehen werden.<br>

### 📌 Beispielszenario:

```yaml
alias: "Signal beim Betreten"
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

Was passiert:<br>
Wenn die Tür zweimal innerhalb von 5 Sekunden geöffnet wird, ertönt nur einmal "ding dong" — der zweite Trigger wird ignoriert.


## ⚗️ restart
---
> Diagramm:

![restart diagramm](/automations/img/restart_digramm.png)

Das Diagramm für mode: restart zeigt das folgende Verhalten einer Automatisierung in Home Assistant:

### 🔁 Was zeigt das Diagramm:
Trigger (Auslöser) — wenn ein Ereignis die Automatisierung aktiviert.<br>
Actions (Aktionen) — die Automatisierung beginnt mit der Ausführung ihrer Aktionen.<br>
Restart automation (Neustart der Automatisierung) — wenn ein neuer Trigger auftritt, bevor die Aktionen abgeschlossen sind, wird die Automatisierung unterbrochen und von vorne gestartet, mit der ersten Aktion.

### 🔄 Beispielszenario:
Du hast eine Automatisierung, die ein Licht einschaltet und 1 Minute wartet, bevor es ausgeschaltet wird:

```yaml
- service: light.turn_on
- delay: "00:01:00"
- service: light.turn_off
```

Wenn innerhalb dieser 1 Minute die Automatisierung erneut ausgelöst wird (z.B. durch erneute Bewegung):<br>
In mode: restart wird die aktuelle Ausführung unterbrochen, und die Automatisierung beginnt von vorne — mit light.turn_on und einer neuen Verzögerung.

## ⚗️ queued
---
> Diagramm:

![queued diagramm](/automations/img/queued_diagramm.png)

Das Diagramm für mode: queued zeigt das Verhalten einer Automatisierung, wenn:<br>
sie mehrmals ausgelöst wird,<br>
aber jede Ausführung nacheinander erfolgen soll (und nicht verloren gehen oder parallel ausgeführt werden soll).

### 📊 Was zeigt das Diagramm genau:
- Trigger (Auslöser) — die Automatisierung wird aktiviert.
- Idle (inaktiv) — wenn die Automatisierung nicht aktiv ist, beginnt die Ausführung der Aktionen.
- Queued (max: 3) — wenn die Automatisierung bereits läuft, werden weitere Trigger in eine Warteschlange gestellt, bis zu maximal 3.
- Action — die Aktionen werden nacheinander aus der Warteschlange ausgeführt.
- Maximum reached — wenn mehr als 3 Trigger in der Warteschlange sind, werden weitere verworfen.

### 📌 Beispielszenario:
Eine Automatisierung sendet eine Benachrichtigung, wenn die Tür geöffnet wird:

```yaml
- service: notify.mobile_app
  data:
    message: "Die Tür wurde geöffnet!"
- delay: "00:00:10"
```

Wenn die Tür 5 Mal schnell hintereinander geöffnet wird:
- 1. Trigger — wird sofort ausgeführt.
- 2., 3., 4. — warten in der Warteschlange (bis max: 3).
- 5. Trigger — wird ignoriert, da die Warteschlange voll ist.

## ⚗️ parallel
---
> Diagramm:

![parallel diagramm](/automations/img/parallel_diagramm.png)

Das Diagramm für mode: parallel zeigt, wie eine Automatisierung funktioniert, wenn mehrere gleichzeitige Ausführungen erlaubt sind:

### 📊 Was zeigt das Diagramm genau:
Trigger 1, Trigger 2, Trigger 3 — alle Trigger starten die Automatisierung unabhängig voneinander.<br>
Jeder Trigger führt zu einer neuen Ausführung der Aktionen — es gibt kein Warten, keine Unterbrechung.<br>
Alle Aktionen werden gleichzeitig, parallel ausgeführt (dargestellt durch parallele Linien/Ströme im Diagramm).

### ⚡ Was passiert in der Praxis:
Es gibt standardmäßig keine Begrenzung — bei 10 Triggern gibt es 10 parallele Ausführungen.<br>
Dies kann zu einer schnellen Ansammlung von Prozessen führen, besonders bei Verzögerungen (delay, wait) oder rechenintensiven Diensten.

### 📌 Beispielszenario:

```yaml
alias: "Licht bei Bewegung einschalten"
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

Was passiert:<br>
Wenn der Bewegungssensor innerhalb von 10 Sekunden 3 Mal ausgelöst wird, gibt es 3 Ausführungen:<br>
Die Lampe wird 3 Mal eingeschaltet.<br>
Jede Ausführung hat ihren eigenen Timer zum Ausschalten (nach 30 Sekunden).

## ⚗️ queued mit max:
---
> Diagramm:

![queued mit max:](/automations/img/queued_max_diagramm.png)

### Wie es funktioniert:
Bei jedem Trigger der Automatisierung wird eine neue Ausführung in die Warteschlange gestellt.<br>
Wenn die aktuelle Automatisierung noch läuft, wartet die neue auf ihren Turn.<br>
Mit max: legst du fest, wie viele Ausführungen gleichzeitig in der Warteschlange warten können.<br>
Wenn die Anzahl der wartenden Ausführungen max überschreitet, wird die neueste verworfen.

Beispiel:
```yaml
mode: queued
max: 3
```

### Szenario:
Du hast eine Automatisierung, die eine Benachrichtigung sendet, wenn die Tür geöffnet wird.<br>
Wenn die Tür häufig geöffnet und geschlossen wird (z.B. ein Kind spielt), könntest du ohne max Dutzende von Ausführungen haben.<br>
Mit max: 3 warten nur 3 Ereignisse — weitere werden ignoriert, um das System vor Überlastung zu schützen.

## ⚗️ parallel mit max:
---
> Diagramm:

![parallel mit max:](/automations/img/paralel_max_diagramm.png)

### Wie es funktioniert:
Erlaubt mehrere gleichzeitige Ausführungen, anstatt sie warten zu lassen.<br>
Aber mit max: kannst du begrenzen, wie viele gleichzeitig aktiv sein können.<br>
Neue Ausführungen werden ignoriert, wenn bereits max Ausführungen aktiv sind.

Beispiel:
```yaml
mode: parallel
max: 2
```

### Szenario:
Du hast eine Automatisierung, die einen Alarm in verschiedenen Räumen auslöst.<br>
Wenn zwei Personen gleichzeitig den Alarm auslösen, sollen beide ihn hören.<br>
Aber du möchtest nicht 20 parallele Alarme haben, wenn etwas Ungewöhnliches passiert.<br>
Mit max: 2 erlaubst du nur zwei Alarme gleichzeitig, weitere werden ignoriert.

---
---
> [!TIP]
> Wenn dir dieses Projekt gefallen hat, findest du [HIER](https://github.com/Bacard1?tab=repositories) weitere interessante Repositories von mir.<br>
> Wenn du Schwierigkeiten hast oder Fragen hast, zögere nicht, mich zu kontaktieren.