# VekSi Project 2 – TwinCAT 3 Irrigation Valve Controller

Industrial TwinCAT 3 PLC/HMI project for controlling **3 irrigation gate valves** with local HMI operation and remote MQTT operation.

## Scope
- Beckhoff PLC + TwinCAT 3 XAE project.
- PLC Structured Text (ST) implementation.
- Valve motion control via **TC2_MC2**.
- MQTT publish/subscribe via **TF6701 / Tc3_IotBase**.
- Local CSV logging on PLC runtime.
- TwinCAT HMI project included in repo.

## Main Features
- Control of 3 motorized valves by setpoint (0–100%).
- Homing, stop/halt, reset fault workflow per valve.
- HMI displays:
  - setpoint / actual position,
  - valve state,
  - command source (HMI or MQTT),
  - fault code,
  - system readiness.
- HMI improvements in this revision:
  - setpoint inputs clamped to 0..100 in UI,
  - source column (HMI/MQTT) shown per valve,
  - MQTT enable checkbox and manual publish button,
  - corrected homing bindings for valves 2 and 3.
- MQTT:
  - cyclic status publish,
  - command subscribe,
  - last command source tracking.
- CSV logging:
  - timestamped rows,
  - setpoint/actual/source per valve,
  - water level + temperature values.

## Repository Layout
- `XAE/` – TwinCAT solution, PLC project, boot/config artifacts.
- `XAE/VekSi_PLC/Funtions/` – MAIN + reusable function blocks.
- `XAE/VekSi_PLC/State_Var/` – DUTs/enums.
- `XAE/VekSi_PLC/Variables_Lists/` – global variables/config.
- `HMI/` – TwinCAT HMI project.
- `docs/` – project documentation and diagrams.

## Control Architecture (High Level)
1. `MAIN` gathers HMI commands and resets.
2. `FB_MqttManager` manages broker connection and remote commands.
3. Latest command source updates valve `Active_Setpoint` and `Source`.
4. `FB_ValveControl` executes per-axis state machine.
5. `FB_CsvLogger` writes periodic CSV logs.
6. Status is mirrored to HMI globals.

See detailed diagrams: `docs/system_design.md`.


## MQTT Configuration Policy
- MQTT broker/topic parameters are defined in PLC code constants (`GVL_Config`).
- HMI is used only to enable/disable MQTT operation and trigger manual publish.
- This keeps commissioning simple and avoids runtime drift of network settings.

## MQTT Topics (default)
- Publish status: `irrigation/status`
- Subscribe commands: `irrigation/command`

### Expected Command Payload
```json
{
  "Valve1": 25,
  "Valve2": 50,
  "Valve3": 75
}
```

### Published Status Payload
JSON includes timestamp, each valve setpoint/actual/source, temperature, and water level.

## Build / Deployment (TwinCAT)
1. Open `VeKsi_Project.sln` in Visual Studio with TwinCAT XAE installed.
2. Select target IPC/CX and activate route.
3. Verify axis links (`Axis_Valve1..3`) and physical I/O mappings.
4. Check library references:
   - `Tc2_MC2`
   - `Tc3_IotBase` (TF6701 runtime license required)
   - `Tc3_JsonXml`, `Tc2_Utilities`, `Tc2_System`
5. Build PLC project and activate configuration.
6. Download boot project and restart runtime if required.

## Commissioning Checklist
- Validate NC axis limits and homing mode for each valve.
- Verify analog input scaling and engineering units.
- Test each valve:
  - home,
  - move to 10/50/90%,
  - stop/halt,
  - reset fault.
- Verify MQTT connect/publish/subscribe with broker.
- Verify CSV file creation and row appends.
- Confirm HMI source indicator changes between HMI/MQTT commands.

## Operational Safety Notes
- Configure NC software limits before enabling motion.
- Always verify direction sign and scaling with low-speed jog first.
- Keep fallback mode: if MQTT is disabled, control remains local via HMI.
- Use fault reset only after root cause is resolved.

## Documentation Index
- `docs/system_design.md` – architecture, state machines, HMI flow, recreate steps.

