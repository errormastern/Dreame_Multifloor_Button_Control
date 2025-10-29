# Dreame Vacuum Multi-Floor Button Control

[![Version](https://img.shields.io/badge/version-0.2.5-blue.svg)](https://github.com/errormastern/Dreame_Multifloor_Button_Control/releases)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10%2B-green.svg)](https://www.home-assistant.io/)
[![License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-alpha-red.svg)](https://github.com/errormastern/Dreame_Multifloor_Button_Control)

Multi-floor control for Dreame vacuum cleaners via button triggers (MQTT, Device, State, or Event)

## Features

- **Zero Configuration**: Select vacuum entity, all else auto-detected
- **Flexible Triggers**: MQTT, Device, State, or Event per function
- **Intelligent Start/Pause**: Base station detection and adaptive behavior
- **Room/Segment Cleaning**: Configurable repeat counts
- **Self-Clean Automation**: Automatically enabled/disabled based on current map
- **Map Switching**: Up to 3 maps
- **Debug Mode**: Persistent notifications for troubleshooting

## Requirements

- Home Assistant ≥ 2024.10.0
- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum) ≥ v2.0.0b19
- At least one saved map configured in robot
- Button device (optional: Zigbee2MQTT switch or any Home Assistant trigger)

## Installation

### Via Import Button

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/errormastern/Dreame_Multifloor_Button_Control/raw/main/vacuum_control.yaml)

### Manual Import

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Click **Import Blueprint**
3. Enter URL: `https://github.com/errormastern/Dreame_Multifloor_Button_Control/raw/main/vacuum_control.yaml`
4. Click **Preview** → **Import**

## Configuration

### Basic Setup

1. Create automation from blueprint
2. Select your vacuum entity (e.g., `vacuum.dreame_x10`)
3. Configure triggers for desired functions (see below)
4. Save automation

All entities (status, mode, map, camera) are automatically detected from the vacuum entity.

### Function Triggers

Configure triggers for each function you want to use:

| Function | Trigger ID | Default MQTT/Device Action | Description |
|----------|------------|----------------------------|-------------|
| Sweep Only | `fn_sweep` | `1_single` | Set cleaning mode to sweep only |
| Sweep + Mop | `fn_mop` | `2_single` | Set cleaning mode to sweep and mop |
| Start/Pause/Resume | `fn_start` | `3_single` | Smart start/pause/resume logic |
| Map 1 Switch | `fn_map1` | `4_single` | Switch to Map 1 |
| Map 2 Switch | `fn_map2` | `5_single` | Switch to Map 2 |
| Map 3 Switch | `fn_map3` | `6_single` | Switch to Map 3 |

#### Trigger Examples

**MQTT Trigger** (Zigbee2MQTT button):
```yaml
trigger: mqtt
topic: zigbee2mqtt/vacuum_button
payload: '{"action": "1_single"}'
```

**Device Trigger** (Zigbee2MQTT device):
```yaml
trigger: device
device_id: your_device_id
type: action
subtype: 1_single
```

**State Trigger** (any entity):
```yaml
trigger: state
entity_id: input_button.sweep_only
```

**⚠️ Important:** MQTT/Device triggers auto-detect action values (`1_single`, `2_single`, etc.). For State/Event triggers, manually set Trigger ID in trigger advanced options (e.g., `fn_sweep`, `fn_mop`, `fn_start`, `fn_map1`, `fn_map2`, `fn_map3`).

## Functions

### Mode Switching

**Sweep Only** and **Sweep + Mop** functions set the cleaning mode via `select.{robot}_cleaning_mode` entity.

Default values:
- Sweep Only: `sweeping`
- Sweep + Mop: `sweeping_and_mopping`

Adjust in **Cleaning Settings** if your robot uses different values.

### Smart Start/Pause/Resume

Adaptive behavior based on robot status:

| Robot Status | Action |
|--------------|--------|
| Idle | Start cleaning |
| Cleaning | Pause |
| Paused | Resume |

**Multi-Floor Logic:**
- **Base station map**: Starts cleaning immediately
- **Other maps**: Starts cleaning, waits for cleaning status, then pauses for manual transport

### Map Switching

Switches to selected map and adjusts self-clean switch:
- **Base station map**: Self-clean ON (returns to base after cleaning)
- **Other maps**: Self-clean OFF (manual transport required)

Supports up to 3 maps (Map 1, Map 2, Map 3). Maps are auto-detected from `camera.{robot}_map_1`, `_map_2`, `_map_3` entities. Custom map names are used if configured.

## Advanced Settings

### Segment Service

**Enabled** (default): Uses `dreame_vacuum.vacuum_clean_segment` with room/segment IDs and repeat counts.

**Disabled**: Falls back to `vacuum.start` for full map cleaning.

### Timeouts

- **Start Timeout** (120s): Max wait for robot to reach cleaning status after start command. Automation aborts on timeout.
- **Moistening Timeout** (60s): Max wait for mop moistening status. Continues on timeout (optional feature).

### Cleaning States

Comma-separated list of status values indicating active cleaning. Used for pause detection.

Default: `cleaning,returning,zone_cleaning,room_cleaning,sweeping,mopping,sweeping_and_mopping`

Only change if your robot reports different status values (check `sensor.{robot}_status`).

## Troubleshooting

### Automation Not Triggering

1. Verify at least one function has configured triggers
2. Check trigger configuration (topic, device_id, entity_id)
3. Enable **Debug Mode** to see trigger details in persistent notifications

### Robot Not Starting/Pausing

1. Verify entity auto-detection: Check `sensor.{robot}_status`, `select.{robot}_cleaning_mode`, `select.{robot}_selected_map`
2. Check cleaning mode values in **Cleaning Settings** match your robot's options (Developer Tools → States)
3. Review cleaning states configuration

### Segments Not Found

1. Verify `camera.{robot}_map` entity has `rooms` or `segments` attribute
2. Check robot is online and map is loaded
3. Disable **Use Segment Service** to use fallback `vacuum.start`

### Debug Mode

Enable in **Advanced Settings** to show detailed information on every trigger:
- Triggered function and trigger details
- Robot status, map, and mode
- Auto-detected maps and base station
- Available rooms/segments

## Automation Mode

**Mode:** `queued` (max: 10)

Processes button presses sequentially without cancellation. Required for Zigbee2MQTT buttons that send press + release events.

## Status

This blueprint is in **alpha testing** (v0.2.5). Core functionality is implemented and tested with Dreame X10+. Feedback and bug reports welcome via GitHub Issues.

## Links

- **Repository**: https://github.com/errormastern/Dreame_Multifloor_Button_Control
- **Dreame Vacuum Integration**: https://github.com/Tasshack/dreame-vacuum
- **Zigbee2MQTT**: https://www.zigbee2mqtt.io

## License

MIT License - Free to use and modify.