# Marstek Local API für Home Assistant

> **Firmware-Hinweis:** Die Local-API-Firmware von Marstek ist noch nicht ausgereift; die meisten Auffälligkeiten stammen von den Geräten selbst.
> Bitte melde Issues zuerst an Marstek, sofern der Fehler nicht eindeutig auf dieses Projekt zurückzuführen ist.

Home-Assistant-Integration, die Marstek Venus C/D/E Batterien direkt über die offizielle Local API (UDP) anspricht. Sie liefert lokale Telemetrie, Modus-Steuerung und Aggregation über mehrere Geräte – ohne Cloud-Abhängigkeit.

---

## 1. Local API aktivieren

1. Stelle sicher, dass deine Geräte die aktuelle Firmware nutzen.
2. Aktiviere die *Local API / Open API* auf jedem Gerät über das Tool [Marstek Venus Monitor](https://rweijnen.github.io/marstek-venus-monitor/latest/).
3. Notiere den UDP-Port (Standard: `30000`) und prüfe die Erreichbarkeit im LAN.

<img width="230" height="129" alt="afbeelding" src="https://github.com/user-attachments/assets/035de357-fbe6-4224-8249-03abb3078fa1" />

---

## 2. Integration installieren

### Über HACS
1. Klicke auf diesen Button:

[![Open this repository in HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=cappa83&repository=ha-marstek-local-api&category=integration)

Oder:
1. Öffne **HACS → Integrationen → Benutzerdefinierte Repositories**.
2. Füge `https://github.com/cappa83/ha-marstek-local-api` als *Integration* hinzu.
3. Installiere **Marstek Local API** und starte Home Assistant neu.

### Manuell
1. Kopiere `custom_components/marstek_local_api` in deinen Home-Assistant-Ordner `custom_components`.
2. Starte Home Assistant neu.

---

## 3. Geräte hinzufügen

1. Öffne **Einstellungen → Geräte & Dienste → Integration hinzufügen** und suche **Marstek Local API**.
2. Die Erkennung listet alle gefundenen Batterien. Wähle ein Gerät oder **Alle Geräte**, um einen gemeinsamen Multi-Device-Eintrag zu erzeugen.
3. Wenn die Suche ein Gerät nicht findet, wähle **Manueller IP-Eintrag** und gib Host/Port an.

Nach dem Setup kannst du über **Einstellungen → Geräte & Dienste → Marstek Local API → Konfigurieren**:
- Geräte umbenennen, das Polling-Intervall anpassen oder weitere Geräte hinzufügen/entfernen.
- Die Erkennung erneut starten, wenn neue Geräte im Netz sind.

> **Wichtig:** Wenn alle Batterien in einem gemeinsamen Eintrag zusammengefasst bleiben sollen (inkl. virtuellem **Marstek System**), nutze **Konfigurieren**. Der Standard-Button „Gerät hinzufügen“ in Home Assistant erstellt einen neuen Eintrag mit eigener System-Entität.

<img width="442" height="442" alt="afbeelding" src="https://github.com/user-attachments/assets/45001642-412e-4c85-aace-b495639959ff" />

---

## 4. Einzelgerät vs. virtuelles Systemgerät

- **Einzelgerät:** entsteht, wenn du eine einzelne Batterie hinzufügst. Pro Eintrag werden die Geräte-Entitäten und Modus-Buttons bereitgestellt.
- **Multi-Device-Eintrag:** entsteht bei **Alle Geräte** oder beim Nachrüsten über die Optionen. Die Integration erzeugt zusätzlich ein virtuelles Gerät **„Marstek System“**.
  - Dieses System aggregiert Werte (Gesamtkapazität, Gesamtleistung, SOC-Mittelwert, etc.).
  - Jede physische Batterie bleibt als eigenes Gerät mit eigenen Sensoren erhalten.

<img width="1037" height="488" alt="afbeelding" src="https://github.com/user-attachments/assets/40bcb48a-02e6-4c85-85a4-73751265c6f8" />

---

## 5. Optionen & Polling

In **Konfigurieren** stehen u. a. folgende Optionen zur Verfügung:

- **Update-Intervall (scan_interval):** Basisintervall für reguläre Abfragen (Standard: 60 s).
- **Mode-Poll-Intervall (mode_poll_interval):** separates Intervall für `ES.GetMode` (Standard: 300 s).

Zusätzlich besitzt die Integration einen adaptiven Backoff: bei wiederholten Ausfällen wird das Polling-Intervall temporär erhöht und nach einem erfolgreichen Update wieder auf den Basiswert zurückgesetzt.

---

## 6. Entitäten

| Kategorie | Sensor (Entity-Suffix) | Einheit | Hinweis | Polling-Multiplikator | Standard (s) |
| --- | --- | --- | --- | ---: | ---: |
| **Battery** | `battery_soc` | % | State of charge | 1x | 60 |
|  | `battery_temperature` | °C | Zelltemperatur | 1x | 60 |
|  | `battery_capacity` | kWh | Restkapazität | 1x | 60 |
|  | `battery_rated_capacity` | kWh | Nennkapazität | 1x | 60 |
|  | `battery_available_capacity` | kWh | Verfügbare Energie bis 100 % | 1x | 60 |
|  | `battery_voltage` | V | Spannung | 1x | 60 |
|  | `battery_current` | A | Strom (positiv = laden) | 1x | 60 |
| **Energy System (ES)** | `battery_power` | W | Batterieleistung (positiv = laden) | 1x | 60 |
|  | `battery_power_in` / `battery_power_out` | W | Lade-/Entladeleistung | 1x | 60 |
|  | `battery_state` | Text | `charging` / `discharging` / `idle` | 1x | 60 |
|  | `grid_power` | W | Netzbezug/ Einspeisung (positiv = Bezug) | 1x | 60 |
|  | `offgrid_power` | W | Off-Grid-Last | 1x | 60 |
|  | `pv_power_es` | W | PV-Leistung (ES) | 1x | 60 |
|  | `total_pv_energy` | kWh | PV-Lebensenergie | 1x | 60 |
|  | `total_grid_import` / `total_grid_export` | kWh | Netz-Zählerstände | 1x | 60 |
|  | `total_load_energy` | kWh | Last-Energie gesamt | 1x | 60 |
| **Energy Meter / CT** | `ct_phase_a_power`, `ct_phase_b_power`, `ct_phase_c_power` | W | Phasenleistung | 5x | 300 |
|  | `ct_total_power` | W | CT-Gesamtleistung | 5x | 300 |
| **Mode** | `operating_mode` | Text | Aktueller Modus | 5x | 300 |
| **PV (nur Venus D)** | `pv_power`, `pv_voltage`, `pv_current` | W / V / A | MPPT-Telemetrie | 5x | 300 |
| **Netzwerk** | `wifi_rssi` | dBm | WLAN-Signalstärke | 10x | 600 |
|  | `wifi_ssid`, `wifi_ip`, `wifi_gateway`, `wifi_subnet`, `wifi_dns` | Text | WLAN-Konfiguration | 10x | 600 |
| **Geräteinfo** | `device_model`, `firmware_version`, `ble_mac`, `wifi_mac`, `device_ip` | Text | Identifikation | 10x | 600 |
| **Diagnose** | `last_message_received` | Sekunden | Zeit seit erfolgreichem Poll | 1x | 60 |

Alle Sensoren existieren bei Multi-Device-Konfiguration zusätzlich als aggregierte Varianten am **Marstek System** (Prefix `system_`).

### Mode-Buttons

Jede Batterie stellt drei Buttons zur Modus-Umschaltung bereit:

- `button.marstek_auto_mode` – Auto-Modus
- `button.marstek_ai_mode` – AI-Modus
- `button.marstek_manual_mode` – Manuell

Der Sensor `sensor.marstek_operating_mode` zeigt den aktiven Modus (Auto, AI, Manual, Passive). **Passive** erfordert Parameter (Leistung & Dauer) und kann nur über den Service `set_passive_mode` gesetzt werden.

---

## 7. Services

### Daten-Synchronisation

| Service | Beschreibung | Parameter |
| --- | --- | --- |
| `marstek_local_api.request_data_sync` | Triggert ein sofortiges Update aller Koordinatoren. | Optional `entry_id` (bestimmter Eintrag) und/oder `device_id` (ein Gerät). |

### Manuelle Zeitpläne (Manual Mode)

Die Integration unterstützt drei Services für manuelle Zeitpläne. Im Manual Mode können bis zu 10 Zeitfenster definiert werden (Laden/Entladen + Leistung).

> Wähle bei den Services immer das **Batterie-Gerät** aus. Die Integration leitet den Aufruf korrekt weiter.

> **Hinweis:** Die Marstek Local API unterstützt kein Auslesen der Zeitpläne. Die Konfiguration ist write-only.

| Service | Beschreibung |
| --- | --- |
| `marstek_local_api.set_manual_schedule` | Ein einzelnes Zeitfenster (0–9) setzen. |
| `marstek_local_api.set_manual_schedules` | Mehrere Zeitfenster auf einmal setzen. |
| `marstek_local_api.clear_manual_schedules` | Alle 10 Zeitfenster deaktivieren. |

#### Einzelnen Slot setzen

```yaml
service: marstek_local_api.set_manual_schedule
data:
  device_id: "1234567890abcdef1234567890abcdef"
  time_num: 0
  start_time: "08:00"
  end_time: "16:00"
  days:
    - mon
    - tue
    - wed
    - thu
    - fri
  power: -2000  # Negativ = laden (2000W), positiv = entladen
  enabled: true
```

#### Mehrere Slots setzen

```yaml
service: marstek_local_api.set_manual_schedules
data:
  device_id: "1234567890abcdef1234567890abcdef"
  schedules:
    - time_num: 0
      start_time: "08:00"
      end_time: "16:00"
      days: [mon, tue, wed, thu, fri]
      power: -2000
      enabled: true
    - time_num: 1
      start_time: "18:00"
      end_time: "22:00"
      days: [mon, tue, wed, thu, fri]
      power: 800
      enabled: true
```

#### Alle Slots löschen

```yaml
service: marstek_local_api.clear_manual_schedules
data:
  device_id: "1234567890abcdef1234567890abcdef"
```

> Dieser Call kann mehrere Minuten dauern: die API akzeptiert nur einen Slot pro Aufruf, und viele Geräte lehnen Schreibvorgänge zunächst ab.

#### Parameter

- **time_num**: Slot 0–9.
- **start_time** / **end_time**: 24h-Format `HH:MM`.
- **days**: Wochentage `mon`–`sun`.
- **power**: Leistung in Watt. **Negativ = Laden**, **positiv = Entladen**.
- **enabled**: Aktiv/Deaktiviert.
- **device_id**: Home-Assistant-Geräte-ID.

#### Hinweise

- Ein Wechsel auf „Manual“ aktiviert keine Zeitpläne automatisch – nutze die Services.
- Zeitpläne können überlappen; die Priorität wird vom Gerät geregelt.
- Zeitpläne bleiben nach Neustart des Geräts erhalten.
