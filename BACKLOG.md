# Feature Backlog

This document tracks planned features and enhancements for the Dreame Multi-Floor Button Control Blueprint.

## Status Legend

- 🔵 **Planned** - Feature identified, not yet started
- 🟡 **In Progress** - Currently being developed
- 🟢 **Completed** - Implemented and released
- 🔴 **Blocked** - Waiting on dependencies or decisions
- ⚪ **Deferred** - Postponed to future release

---

## Cleaning Intensity Control

**Status:** 🔵 Planned
**Priority:** High
**Effort:** Medium

### Description
Add configurable cleaning intensity modes via button controls:
- **Light**: Quick cleaning for daily maintenance
- **Standard**: Regular cleaning mode (current default)
- **Intense**: Deep cleaning with multiple passes

### Technical Considerations
- Use `select.{name}_suction_level` entity (if available)
- May require integration support for intensity settings
- Could be mapped to existing buttons or require additional button actions

### Acceptance Criteria
- [ ] Button action to cycle through intensity modes
- [ ] Visual feedback via notification (debug mode)
- [ ] Intensity setting persists across cleaning sessions
- [ ] Works with both segment-based and full map cleaning

---

## Calendar-Based Scheduling

**Status:** 🔵 Planned
**Priority:** Medium
**Effort:** High

### Description
Integration with Home Assistant calendar entities to schedule automated cleaning sessions.

### Technical Considerations
- Use `calendar.*` entity as trigger
- Parse calendar event titles for map/mode information
- Consider time window before event starts
- Handle calendar sync delays

### Acceptance Criteria
- [ ] Read cleaning schedule from HA calendar
- [ ] Parse event data (map, mode, intensity)
- [ ] Automatic cleaning start at scheduled time
- [ ] Skip cleaning if robot is already active

---

## Presence-Based Cleaning Window

**Status:** 🔵 Planned
**Priority:** Medium
**Effort:** Medium

### Description
Automatically start cleaning when all occupants are away and stop when someone returns home.

### Technical Considerations
- Requires presence detection (person entities, device trackers)
- Define "away" threshold (all persons away for X minutes)
- Handle edge cases (guest mode, manual override)
- Integration with existing pause/resume logic

### Acceptance Criteria
- [ ] Configurable presence sensors input
- [ ] Configurable delay before starting (home empty for X min)
- [ ] Automatic pause when presence detected
- [ ] Option to resume after occupants leave again

---

## Auto-Pause on Home Arrival

**Status:** 🔵 Planned
**Priority:** High
**Effort:** Low

### Description
Immediately pause active cleaning when someone arrives home.

### Technical Considerations
- Trigger on person/device tracker state change (away → home)
- Check if vacuum is in active cleaning state
- Send pause command immediately
- Optional notification with resume option

### Acceptance Criteria
- [ ] Configurable person/device tracker entities
- [ ] Pause only during active cleaning (not returning/docking)
- [ ] Optional notification: "Cleaning paused due to arrival"
- [ ] Works independently of button controls

---

## Room-Based Presence Detection

**Status:** 🔵 Planned
**Priority:** Low
**Effort:** High

### Description
Pause cleaning when someone enters the room currently being cleaned.

### Technical Considerations
- Requires room-level presence sensors (motion, mmWave, etc.)
- Map room names to vacuum segments
- Complex state tracking (which room is being cleaned)
- May not be supported by all Dreame models

### Acceptance Criteria
- [ ] Configurable room → sensor mapping
- [ ] Detect current cleaning location from vacuum state
- [ ] Pause when presence detected in active room
- [ ] Resume after person leaves (with timeout)

### Dependencies
- Segment-based cleaning must be active
- Room presence sensors required
- Vacuum integration must expose current segment/room

---

## Scheduled Floor-Change Notifications

**Status:** 🔵 Planned
**Priority:** High
**Effort:** Medium

### Description
Send scheduled reminders to prepare non-base-station floors for cleaning, with actionable notification link to start cleaning.

### Technical Considerations
- Use time-based trigger or calendar integration
- Actionable notification with deep link to automation
- "Prepare floor X for cleaning" message
- One-tap start cleaning action

### Acceptance Criteria
- [ ] Configurable notification schedule per floor/map
- [ ] Actionable notification with "Start Cleaning" button
- [ ] Button triggers map switch + cleaning start
- [ ] Optional: snooze/reschedule option

---

## Sequential Sweep-Then-Mop Automation

**Status:** 🔵 Planned
**Priority:** Medium
**Effort:** Medium

### Description
Automatically perform sweeping pass first, then return to base, then start mopping pass on the same map.

### Technical Considerations
- Requires two cleaning cycles per floor
- Must wait for first cycle completion
- May require manual water tank refill between cycles
- Check if integration supports mid-cycle mode switching

### Acceptance Criteria
- [ ] Button action or automation trigger
- [ ] First pass: sweeping-only mode
- [ ] Wait for completion + return to base
- [ ] Second pass: mopping-only mode (same map)
- [ ] Optional notification between passes for tank refill

### Open Questions
- Can mode be switched without returning to base?
- Does integration support programmatic mode switching during multi-cycle automation?

---

## Implementation Notes

### Priority Guidelines
- **High**: User-requested features or safety/convenience improvements
- **Medium**: Nice-to-have enhancements
- **Low**: Advanced features requiring extensive setup

### Effort Estimation
- **Low**: <4 hours implementation + testing
- **Medium**: 4-8 hours implementation + testing
- **High**: >8 hours or requires external dependencies

### Feature Requests

To request a new feature:
1. Open an issue on [GitHub](https://github.com/errormastern/Dreame_Multifloor_Button_Control/issues)
2. Use the "Feature Request" template
3. Provide use case and expected behavior

---

**Last Updated:** 2025-10-29
