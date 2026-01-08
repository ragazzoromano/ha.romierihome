# AI Agent Guide: Romieri Home Assistant Config

This repo is a Home Assistant configuration (YAML-first). Focus on modular irrigation logic, MQTT-powered sensors/switches, and Lovelace dashboards.

## Big Picture
- Core entry: [configuration.yaml](../configuration.yaml) uses `default_config` and `homeassistant:packages` to include irrigation files: [irrigation.yaml](../irrigation.yaml), [irrigation_schedule.yaml](../irrigation_schedule.yaml), [irrigation_scripts.yaml](../irrigation_scripts.yaml).
- Automations centralized in [automations.yaml](../automations.yaml); dashboards in [Dashboards/](../Dashboards) (e.g., [Dashboards/Irrigation.yaml](../Dashboards/Irrigation.yaml)).
- State inputs live in Home Assistant helpers: `input_boolean`, `input_number`, `input_datetime`; templates build derived `sensor` entities.

## Data & Integrations
- MQTT sensors from Tasmota/ESP topics (e.g., `DHT22/sensor1/SENSOR`, `PZEM004_H/elSensorH/SENSOR`, rain `tele/elSensorR01/SENSOR`). See [configuration.yaml](../configuration.yaml).
- Node-RED publishes overview and grid via `noderedHA/SENSOR` and `noderedHA/GRIDSENSOR`.
- Irrigation pump energy via `sensor.irrigation_pump_power` + `integration` → utility meters ([irrigation.yaml](../irrigation.yaml)).
- IFTTT used for alarm (`event: RomieriAlarm`) and mobile_app notifications (see [automations.yaml](../automations.yaml)).

## Irrigation Architecture
- 8 zones as MQTT `switch.zona_1..8` with `unique_id` and Tasmota topics (see [irrigation.yaml](../irrigation.yaml)).
- Durations: normal mode uses `input_number.slider_zona_N` (minutes); advanced weekly mode uses per-day `input_boolean.<day>_zone_N` and `input_number.<day>_zone_N_duration` (see [irrigation_schedule.yaml](../irrigation_schedule.yaml)).
- Global factor: `input_number.irrigation_global_factor` scales final seconds via template sensors; zones 3 and 7 use pure time (no factor).
- Runtime helpers compute remaining/elapsed time using `now()` and `switch.zona_N.last_changed` (see template sensors in [irrigation.yaml](../irrigation.yaml)).
- Scripts:
  - Per-zone starters `script.grdn_start_zonaN` select advanced vs normal and delay by computed final time ([irrigation_scripts.yaml](../irrigation_scripts.yaml)).
  - Cycle scripts `script.grdn_inizia_ciclo_irrigazione` and `script.grdn_inizia_ciclo_gocciagoccia` chain zone runs; `script.grdn_ferma_irrigazione` stops everything.
  - Utility scripts to copy/apply/reset weekly schedules and to select UI day (`script.irrigation_*`).

## Automations & Triggers
- Weekly irrigation: `GRDN_Irrigazione_Settimanale` triggers at `input_datetime.time_start_irrigazione`; chooses advanced (per-day zone flags) vs normal (day-of-week booleans). See [automations.yaml](../automations.yaml).
- Additional variants at sunrise (`sunrise` trigger). Notifications via `notify.mobile_app_*`.

## Automation Entity ID Reference
This table documents the mapping between automation entity IDs used in dashboards and their corresponding automation aliases and IDs in `automations.yaml`. Use this reference when working with automation references in dashboard files.

| Entity ID | Alias | Automation ID | Dashboard Usage |
|-----------|-------|---------------|-----------------|
| `automation.irrigazione_avvio_settimanale_programmato` | Irrigazione - Avvio - Settimanale programmato | 1632223938466 | Irrigation.yaml (lines 52, 560) |
| `automation.irrigazione_avvio_alba_settimanale` | Irrigazione - Avvio - Alba settimanale | 1632224023054 | Irrigation.yaml (lines 55, 568) |
| `automation.irrigazione_chiusura_tapparelle_zona_6` | Irrigazione - Chiusura tapparelle - Zona 6 | 1633346222844 | Irrigation.yaml (line 430) |
| `automation.irrigazione_chiusura_tapparelle_zona_8` | Irrigazione - Chiusura tapparelle - Zona 8 | 1648500187701 | Irrigation.yaml (line 433) |
| `automation.grdn_gocciagoccia_settimanale` | GRDN_GocciaGoccia_Settimanale | 1767541092476 | Irrigation.yaml (line 484) |
| `automation.robot_ritorno_base_pioggia_irrigazione_attiva` | Robot - Ritorno base - Pioggia/Irrigazione attiva | 1658506463789 | BiPedo.yaml (lines 95, 111) |
| `automation.allarme_attivazione_uscita_da_casa` | Allarme - Attivazione - Uscita da casa | 1609686557369 | Overview.yaml (lines 667, 735) |
| `automation.allarme_attivazione_manuale` | Allarme - Attivazione - Manuale | 1614111057350 | Overview.yaml (lines 676, 736) |
| `automation.allarme_disattivazione_rientro_a_casa` | Allarme - Disattivazione - Rientro a casa | 1609686952547 | Overview.yaml (line 737) |
| `automation.allarme_disattivazione_manuale` | Allarme - Disattivazione - Manuale | 1614083771225 | Overview.yaml (lines 688, 738) |
| `automation.alarm_enable_partial_ifttt` | Alarm Enable Partial IFTTT | 1674131878590 | Overview.yaml (lines 703, 739) |

**Important Notes:**
- Dashboard entity references must exactly match the entity_id derived from the automation's `alias` field in `automations.yaml`
- Home Assistant automatically generates entity IDs from aliases by converting to lowercase, replacing spaces with underscores, and removing special characters
- When renaming automation aliases, remember to update all corresponding dashboard references
- The numeric `id` field is only for Home Assistant's internal use and should not be referenced in dashboards

## Conventions
- Entities and sensors are bilingual but consistent: `switch.zona_N`, `sensor.Irrigation Zona N Final Time`, minute variants `... Minutes`.
- Template naming follows `irrigation_zona_N_final_time` and `<day>_zone_N_final_time` with `unique_id` for registry stability.
- MQTT topics use Tasmota conventions `cmnd/`, `stat/`, `tele/`; keep payloads `ON/OFF` and LWT Online/Offline.
- Advanced mode resolves day via `now().strftime('%A').lower()`; ensure entity names match this pattern.

## Developer Workflow
- After YAML edits, reload in UI: Settings → Server Controls → Reload Automations/Scripts; Restart Core for `homeassistant:packages` changes.
- Validate YAML in UI Developer Tools → YAML; watch Logs for template errors.
- Secrets live in [secrets.yaml](../secrets.yaml); reference with `!secret` for credentials (e.g., recorder DB). Do not hardcode secrets in repo.

## Common Changes (examples)
- Add a new zone: update [irrigation.yaml](../irrigation.yaml) (MQTT switch, duration sliders, final-time + minute templates) and extend scripts/cycles in [irrigation_scripts.yaml](../irrigation_scripts.yaml).
- Adjust weekly program: modify per-day booleans/durations and templates in [irrigation_schedule.yaml](../irrigation_schedule.yaml); use `script.irrigation_copy_day_schedule` or `script.irrigation_apply_to_all_days`.
- Surface data in dashboards: edit cards in [Dashboards/Irrigation.yaml](../Dashboards/Irrigation.yaml) to show runtime and energy sensors.

## Backups
- Scope: back up irrigation modules, automations, and dashboards before structural edits: [irrigation.yaml](../irrigation.yaml), [irrigation_schedule.yaml](../irrigation_schedule.yaml), [irrigation_scripts.yaml](../irrigation_scripts.yaml), [automations.yaml](../automations.yaml), [Dashboards/](../Dashboards).
- Git snapshot: commit a clean state and create a branch/tag (e.g., `backup/irrigation-YYYYMMDD`).
- Quick local backup (PowerShell on Windows):

```powershell
Compress-Archive -Path \
  .\irrigation.yaml, \
  .\irrigation_schedule.yaml, \
  .\irrigation_scripts.yaml, \
  .\automations.yaml, \
  .\Dashboards\* \
  -DestinationPath .\backups\ha-config-$(Get-Date -Format yyyyMMdd-HHmm).zip
```
- Restore: check out the backup branch/tag or unzip selected files back into the config folder, then use UI → Server Controls to reload/restart as needed.
