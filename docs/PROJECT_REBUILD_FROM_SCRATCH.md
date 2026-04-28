# Project Rebuild (From Scratch) – Working Baseline Strategy

This branch starts a clean rebuild strategy while reusing proven, already-tested command paths from your current codebase.

## Rebuild principles
1. Keep one source of truth for configuration (`GVL_Config`).
2. Keep all state enums and structures in `State_Var`.
3. Keep each function block single-purpose:
   - Motion (`FB_ValveControl`)
   - MQTT transport + payload parsing (`FB_MqttManager`)
   - Logging (`FB_CsvLogger`)
   - Coordination (`MAIN`)
4. Preserve working motion/MQTT command patterns and only refactor structure/comment clarity.

---

## Target architecture (clean)

### State and data types (`State_Var`)
- `E_SystemState` (new coordinator-level state)
- `E_ValveState`
- `E_ValveSource`
- `E_MqttManagerState`
- `E_CSVLogState`
- `ST_ValveData`
- `ST_MqttConfig`
- `ST_MQTT_Command`

### Configuration (`Variables_Lists`)
- `GVL_Config` (single config file)
  - motion constants
  - MQTT config struct constant
  - logging constants

### Runtime data (`Variables_Lists`)
- `GVL_System`
- `GVL_HMI`
- `GVL_IO`

### Functional layer (`Funtions`)
- `FB_ValveControl`
- `FB_MqttManager`
- `FB_CsvLogger`
- `MAIN`

---

## Rebuild execution plan

### Step 1 – Data and state contracts (done in current baseline)
- Centralized MQTT config in `GVL_Config` as one struct constant.
- Moved MQTT manager connection-state enum to `State_Var`.

### Step 2 – Coordinator cleanup (next)
- Introduce `E_SystemState` and coordinator state flow in `MAIN`:
  - INIT
  - HOMING_REQUIRED
  - READY
  - RUN
  - FAULT
- Keep current tested motion commands and add explicit transition comments.

### Step 3 – MQTT contract hardening
- Keep JSON contract stable:
  ```json
  {"Valve1":25,"Valve2":50,"Valve3":75}
  ```
- Add explicit validation + fallback behavior for invalid payload fields.
- Add reconnect and subscribe lifecycle comments per state.

### Step 4 – HMI simplification
- Keep operator-facing controls only:
  - Setpoint 0..100
  - Homing / Stop / Reset
  - MQTT enable and publish
  - Source indicator per valve
- Keep broker config code-only.

### Step 5 – Logging standardization
- Keep row schema stable and documented:
  - timestamp
  - setpoint/actual/source per valve
  - water level
  - temperature
- Add row schema version string at header for future migration safety.

---

## Mapping to existing working commands

The following tested behavior is preserved intentionally:
- Per-axis `MC_Power`, `MC_Home`, `MC_MoveAbsolute`, `MC_Halt`, `MC_Reset` sequence in valve FB.
- MQTT command clamping and source tagging.
- CSV write state machine with non-blocking file operations.

This means the rebuild is structural and documentation-focused first, then behavior-safe increments.

---

## Acceptance criteria for “redo complete”
1. Single config file policy enforced (`GVL_Config`).
2. All state enums live under `State_Var`.
3. MAIN has explicit coordinator-level state flow.
4. HMI reflects only required controls and diagnostics.
5. Build passes in TwinCAT XAE and field smoke tests pass:
   - HMI setpoint move
   - MQTT setpoint move
   - Source indicator correctness
   - CSV row correctness

