# 🤖 Dreame Vacuum - Multi-Floor Control

[![Version](https://img.shields.io/badge/version-0.8.0%20beta-blue.svg)](https://github.com/errormastern/dreame-multifloor-control/releases)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10%2B-green.svg)](https://www.home-assistant.io/)
[![License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-beta-yellow.svg)](https://github.com/errormastern/dreame-multifloor-control)

⚠️ Beta Release - Testing in progress

Control Dreame vacuum cleaners across multiple floors with scheduled cleaning and notification-based transport workflow. Maps with base stations clean automatically; maps without base stations use notifications with action buttons for manual transport.

## ✨ Features

🤖 Auto-detection of vacuum entities (select vacuum, rest detected automatically)<br>
📅 Per-map schedules with sweep/mop modes (3 maps, 6 schedules total)<br>
🔔 Notification workflow with action buttons for transport preparation<br>
📱 iOS lock screen notifications with configurable interruption levels<br>
🎛️ Manual control via MQTT, device triggers, state changes, or events<br>
🏠 Segment-based cleaning with configurable repeats per map<br>
✨ **NEW:** Optional customized cleaning - uses room settings from Dreame app (cleaning order, per-room suction/water/repeats)<br>
⚠️ Safety checks: Schedule conflict detection, dock status validation, emergency map validation<br>
🌐 Localization support for multilingual notifications<br>
🐛 Debug mode with timing measurements and execution tracking

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
3. Configure triggers for functions you need (see workflows below)
4. Save and test

Related entities (status, mode, map, camera) are auto-detected from the vacuum entity.

## 🔄 Workflows

This blueprint provides two main workflows for controlling your vacuum across multiple floors.

### 📅 Scheduled Cleaning Workflow

**Purpose:** Time-based cleaning with automatic preparation and transport notifications.

**Setup:**
1. Create Schedule helpers in Home Assistant (Settings → Devices & Services → Helpers → Schedule)
2. Configure Map 1/2/3 schedules in blueprint (assign schedule entities)
3. Set up notification service for maps without base station

**Behavior depends on map type:**

**Maps WITH base station:**
- Schedule triggers → Robot starts cleaning immediately
- No manual intervention needed

**Maps WITHOUT base station:**
1. Schedule triggers → Notification with "Prepare Robot" and "Skip" buttons
2. Press "Prepare Robot" → Robot washes mop (if sweep+mop), starts cleaning, pauses automatically
3. Pickup notification → Transport robot to target floor
4. Press "Start Cleaning" button → Robot resumes cleaning

**Conflict Detection:**
- Only one schedule runs at a time
- New schedules abort silently if robot is already cleaning

### 🎛️ Manual Control Workflow

**Purpose:** Direct control via buttons, switches, or other triggers.

**Available Functions:**

| Function | Description | Use Case |
|----------|-------------|----------|
| **Sweep Only Mode** | Set cleaning mode to sweep-only | Quick cleaning without mopping |
| **Sweep + Mop Mode** | Set cleaning mode with mop | Full cleaning with water |
| **Smart Start/Pause/Resume** | Context-aware control | Main cleaning button |
| **Map 1 / Map 2 / Map 3** | Switch between floor maps | Multi-floor control |

**Smart Start/Pause/Resume Logic:**

The function automatically adapts to robot status:

| Robot Status | Current Map | Action |
|--------------|-------------|--------|
| **Cleaning** | Any | Pause immediately |
| **Paused** | Any | Resume cleaning |
| **Idle (docked)** | Base station map | Start cleaning on current map |
| **Idle (docked)** | Other map | Run preparation workflow → pause for transport |

**Preparation Workflow (Non-Base Station Maps):**

For sweep+mop mode on maps without base station:

1. Robot washes mop at base station (~3-4 minutes)
2. Robot starts cleaning and undocks
3. After short delay (~4.5s), robot pauses automatically
4. User transports robot to target floor
5. Press button again to resume cleaning

> **Why the delay?** Allows robot to move away from charging contacts for easier pickup.

**Trigger Setup:**

Use any Home Assistant trigger type:
- **MQTT triggers**: Action values auto-detected from payload
- **Device triggers**: Action values auto-detected
- **State/Event triggers**: Set Trigger ID manually (e.g., `fn_start`)

## 🌐 Localization

The blueprint supports multilingual notifications. Customize display texts in the Localization section:
- Sweep/Mop mode labels
- Button labels (Prepare, Skip, Start, Cancel)
- Used in notifications and action buttons

Internal logic remains English - only user-facing texts are localized.

## 📚 Configuration Details

Detailed configuration documentation (all blueprint sections, settings, and examples) will be provided in a separate configuration guide.

## Technical Notes

**Automation Mode:** `queued` (max: 10) - Required for button devices sending press + release events.

**Tested with:** Xiaomi X10+ (Dreame L10s Ultra models should also work, as well as from Dreame Integration supported models)

**Feedback:** Report issues or request features via [GitHub Issues](https://github.com/errormastern/dreame-multifloor-control/issues)

## Links

- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum) - Required custom integration
- [Repository](https://github.com/errormastern/dreame-multifloor-control) - Source code and releases
- [Issues](https://github.com/errormastern/dreame-multifloor-control/issues) - Bug reports and feature requests

---

**License**: MIT
