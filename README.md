# 🤖 Dreame Vacuum - Multi-Floor Control

[![Version](https://img.shields.io/badge/version-0.3.6-blue.svg)](https://github.com/errormastern/dreame-multifloor-control/releases)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10%2B-green.svg)](https://www.home-assistant.io/)
[![License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-alpha-red.svg)](https://github.com/errormastern/dreame-multifloor-control)

Control Dreame vacuum cleaners across multiple floors with scheduled cleaning and notification-based transport workflow. Maps with base stations clean automatically; maps without base stations use notifications with action buttons for manual transport.

## ✨ Features

- Auto-detection of vacuum entities (select vacuum, rest detected automatically)
- 📅 Per-map schedules with sweep/mop modes (3 maps, 6 schedules total)
- 🔔 Notification workflow with action buttons for transport preparation
- 📱 iOS lock screen notifications with configurable interruption levels
- 🎛️ Manual control via MQTT, device triggers, state changes, or events
- 🏠 Segment-based cleaning with configurable repeats per map
- ⚠️ Conflict detection (only one schedule runs at a time)
- 🐛 Debug mode with timing measurements and execution tracking

## 📋 Requirements

- Home Assistant ≥ 2024.10.0
- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum) ≥ v2.0.0b19
- At least one saved map configured in robot
- Optional: Schedule helpers for time-based automation
- Optional: Mobile app for notifications (iOS or Android)

## 💾 Installation

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/errormastern/dreame-multifloor-control/raw/main/vacuum_control.yaml)

Or manually: **Settings** → **Automations & Scenes** → **Blueprints** → **Import Blueprint** → Paste URL above

## 🚀 Quick Start

1. Create automation from blueprint
2. Select your vacuum entity (e.g., `vacuum.xiaomi_x10`)
3. Configure triggers for functions you need (see [Manual Control](#manual-control) below)
4. Save and test

Related entities (status, mode, map, camera) are auto-detected from the vacuum entity.

## 📅 Schedule Setup

### 1️⃣ Create Schedule Helpers

Create schedule entities in Home Assistant (Helpers → Schedule). Assign them to maps in the blueprint automation (Map 1/2/3 Configuration sections).

### 2️⃣ Configure Notifications (Optional)

**Required for maps without base station.**

Configure scheduled and pickup notifications with:
- Notification service (e.g., `notify.mobile_app_iphone`)
- Title/message with [template variables](#template-variables)
- Action button names
- **iOS Settings**: Interruption level (`passive`/`active`/`time-sensitive`/`critical`), sound, critical alerts
- Repeat settings (1-3 times, 0-240 min interval)

### Template Variables

| Variable | Description |
|----------|-------------|
| `{{ robot_name }}` | Robot name |
| `{{ map_name }}` | Target map name |
| `{{ current_map }}` | Current map name |
| `{{ cleaning_mode_display }}` | Sweep Only / Sweep + Mop |
| `{{ schedule_name }}` | Triggered schedule name |
| `{{ current_time }}` | Current time (HH:MM) |
| `{{ wait_minutes }}` | Wait minutes (repeats) |
| `{{ repeat_number }}` | Repeat count |
| `{{ base_station_map }}` | Base station map name |

### Workflow

**Maps WITH base station:**
- Schedule triggers → Robot starts cleaning automatically
- No notifications sent

**Maps WITHOUT base station:**
1. Schedule triggers → Scheduled notification with action buttons
2. Press "Prepare Robot" → Robot starts, moistens mop, pauses automatically
3. Pickup notification → Pick up robot and transport to target floor
4. Press "Start Cleaning" → Robot resumes cleaning

**Conflict Detection:**
- Only one schedule runs at a time
- If robot is already cleaning, new schedules abort silently

## 🎛️ Manual Control

Each function can use any Home Assistant trigger. For **MQTT/Device triggers** (e.g., Zigbee2MQTT buttons), action values are auto-detected. For **State/Event triggers**, set the Trigger ID manually in advanced trigger options.

| Function | Trigger ID | Description |
|----------|------------|-------------|
| Sweep Only Mode | `fn_sweep` | Set cleaning mode to sweep-only |
| Sweep + Mop Mode | `fn_mop` | Set cleaning mode to sweep and mop |
| Start/Pause/Resume | `fn_start` | Context-aware workflow based on robot status |
| Map 1 / Map 2 / Map 3 | `fn_map1` / `fn_map2` / `fn_map3` | Switch to selected map |

### Trigger Examples

MQTT/Device triggers auto-detect action values. For State/Event triggers, set Trigger ID manually (e.g., `fn_start`).

**Note:** Use Schedule Helpers for time-based automation, not manual triggers.

## Start/Pause/Resume Workflow

The function adapts to robot status and location:

| Robot Status | Current Map | Action |
|--------------|-------------|--------|
| **Cleaning** | Any | Pause immediately |
| **Paused** | Any | Resume cleaning |
| **Idle** | Base station map | Start cleaning (self-clean enabled) |
| **Idle** | Other map | Run preparation workflow → pause for transport |

### Preparation Workflow (Non-Base Station Maps)

For sweep+mop mode:

1. Enable self-clean → Triggers preparation program (moistening, tank filling)
2. Start cleaning → Wait for undocking
3. Optional delay (default 2s, ~10cm from station)
4. Pause robot
5. Disable self-clean → Prevents return to base during cleaning

> Self-clean must be disabled after pause to prevent robot returning to base while cleaning on target floor.

## ⚙️ Advanced Settings

### Pause Delay After Undocking
Default: 2.0s (Range: 0.0-5.0s). Time to wait after leaving station before pausing. Allows robot to move away from contacts for easier pickup.

### Timeouts
Adjust if robot needs more time:
- **Start**: 120s (30-300s)
- **Moistening**: 60s (10-180s)
- **Undocking**: 30s (10-60s)

### Segment Cleaning
**Enabled** (default): Uses `dreame_vacuum.vacuum_clean_segment` with configurable repeats.
**Disabled**: Falls back to `vacuum.start` (full map).

### Debug Mode
Shows persistent notifications with timing measurements and execution details. Helpful for optimizing timeouts.

## 🔧 Troubleshooting

**Automation not triggering:** Verify triggers configured, enable debug mode.

**Robot not starting/pausing:** Check entity auto-detection and cleaning mode values in debug mode.

**Segments not working:** Verify `camera.{robot}_map` has `segments` attribute, or disable segment service.

**iOS notifications not on lock screen:** Set interruption level to `active` or higher. Critical alerts require iOS permission.

**Android notifications:** Verify notification service and channel settings.

## Technical Notes

**Automation Mode:** `queued` (max: 10) - Required for button devices sending press + release events.

**Status:** Alpha (v0.3.6) - Tested with Dreame X10+. Feedback via [GitHub Issues](https://github.com/errormastern/dreame-multifloor-control/issues).

## Links

- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum) - Required custom integration
- [Repository](https://github.com/errormastern/dreame-multifloor-control) - Source code and releases
- [Issues](https://github.com/errormastern/dreame-multifloor-control/issues) - Bug reports and feature requests

---

**License**: MIT
