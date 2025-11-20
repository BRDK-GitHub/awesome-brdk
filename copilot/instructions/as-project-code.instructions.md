---
description: Coding guidelines for B&R Automation Studio projects (IEC 61131-3 Structured Text, Types, Variables) following BRDK standards.
applyTo: "**/*.{typ,var,st}"
---

# B&R Automation Studio Coding Guidelines

You are an expert in B&R Automation Studio (6.0+) and IEC 61131-3 Structured Text.
Follow these guidelines when generating code for `.st`, `.var`, and `.typ` files within the BRDK framework.

## Naming Conventions

### Variables
- **Local Variables:** Use `CamelCase` starting with lowercase.
  - Example: `actPressure`, `commandCount`.
- **Global Variables:** Use `CamelCase` with prefix `g`.
  - Example: `gMainInterface`, `gMainConfig`.
- **Constants:** Use `SCREAMING_SNAKE_CASE`.
  - Example: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS`.
- **Pointers:** Use prefix `p` + descriptive name.
  - Example: `pAxisStatus`, `pRecipeData`.
- **IO Variables:** Use specific prefixes based on type:
  - `di` (Digital Input): `diStartButton`
  - `do` (Digital Output): `doLampGreen`
  - `ai` (Analog Input): `aiTankLevel`
  - `ao` (Analog Output): `aoValveSetpoint`
  - `at` (Temperature): `atOvenTemp`
  - `si` (Safety Input): `siGuardClosed`

### Types and Structures
- **User Types:** Start lowercase, `CamelCase`, end with `Type`.
  - Example: `recipeType`, `localHwType`.
- **Substructures:** Pattern `<parent><NewName>Type`.
  - Example: `recipeTimeType` (child of `recipeType`).
- **Enumerators:** Name ends with `Enum`. Members are `ENUMNAME_VALUE` in `SCREAMING_SNAKE_CASE`.
  - Example: `alarmReactionEnum` with member `ALARMREACTION_STOP`.

### Function Blocks
- **Instances:** Name should begin with the FB type.
  - Example: `TON_delay`, `TON_0`.
  - Arrays: `TON_` (omit suffix numbers).

## Documentation
- **Descriptions:** All variables and types in `.var` and `.typ` files MUST have a description comment.
- **Units:** Include engineering units in brackets `[unit]` within the description where relevant.
  - Example: `actPressure : REAL; (* Actual hydraulic pressure [bar] *)`

## Project Structure & File Organization

### Package File Management (`Package.pkg`)
The `Package.pkg` file is the backbone of the Automation Studio project structure. It lists all files and sub-packages contained within a directory.
**CRITICAL:** Whenever you create, rename, or delete a file or folder, you **MUST** update the corresponding `Package.pkg` file.

**Example `Package.pkg`:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<?AutomationStudio FileVersion="4.9"?>
<Package xmlns="http://br-automation.co.at/AS/Package">
  <Objects>
    <Object Type="File">Global.typ</Object>
    <Object Type="File">Global.var</Object>
    <Object Type="Package">Main</Object>
    <Object Type="File">MyNewFile.st</Object> <!-- Added file -->
  </Objects>
</Package>
```

### Folder Hierarchy
All projects must follow this structure. Each Equipment Module (e.g., `Infeed`) has its own package.

```text
ProjectName/
├─ Global.typ              # Global type definitions
├─ Global.var              # Global variable declarations
├─ Main/                   # Parent Equipment Module
│  ├─ MainAlarms.tmx       # Alarm texts
│  ├─ global.typ           # Module-specific global types (Interface, Config, Recipe)
│  ├─ global.var           # Module-specific global variables
│  ├─ Main/                # Task Folder (The actual code)
│  │  ├─ local.st          # Main Program: Init, Cyclic, Exit
│  │  ├─ production.st     # Action: Production mode logic
│  │  ├─ maintenance.st    # Action: Maintenance mode logic (optional)
│  │  ├─ manual.st         # Action: Manual mode logic (optional)
│  │  ├─ alarm.st          # Action: Alarm handling logic
│  │  ├─ local.typ         # Local type definitions
│  │  ├─ local.var         # Local variable declarations
│  ├─ Infeed/              # Child Equipment Module
│  │  ├─ ...               # Same structure as Main/
│  ├─ Fillers/             # Indexed Equipment Module
│  │  ├─ local.st          # Loop over instances
│  │  ├─ local.typ         # Array types
│  │  ├─ local.var         # Array variables
│  │  ├─ localNonIndexed.st  # Action: Non-indexed logic (runs once per scan)
│  │  ├─ localNonIndexed.typ # Non-indexed types
│  │  ├─ localNonIndexed.var # Non-indexed variables
│  │  ├─ ...               # Other action files
```

### Task and Action Files
- **`local.st` (Main Program):** Contains `_INIT`, `_CYCLIC`, and `_EXIT` routines.
  - **Responsibility:** Initializes references, calls the PackML state machine, calls global services, and executes code common to all modes.
  - **Structure:** Calls actions like `production`, `maintenance`, `manual`, and `alarm`.
- **`localNonIndexed.st` (Action):** Used in **Indexed Modules** only.
  - **Responsibility:** Contains logic that applies to the module as a whole, not individual instances (e.g., common calculations, aggregation).
  - **Execution:** Called ONCE at the beginning of `_CYCLIC` before the instance loop.
- **`production.st` (Action):** Implements the logic for `MODE_PRODUCTION`.
  - **Responsibility:** Contains the core process logic.
- **`maintenance.st` (Action):** Implements the logic for `MODE_MAINTENANCE`.
  - **Responsibility:** Cleaning routines, service modes, etc.
- **`manual.st` (Action):** Implements the logic for `MODE_MANUAL`.
  - **Responsibility:** Manual control of actuators.
- **`alarm.st` (Action):** Implements alarm handling.
  - **Responsibility:** Configures and calls `BrdkAlarmControl` FBs and handles alarm reactions (Stop, Abort).

### Standard Local Variables (References)
In `local.st` and all action files (`production.st`, etc.), ALWAYS use these standard references to access data. Do not use the raw structure names directly.

- **`this`**: Reference to `localType` (the current instance's local data).
- **`hmi`**: Reference to `hmiType` (HMI binding variables).
- **`em`**: Reference to the `EquipmentModule` Function Block.
- **`interface`**: Reference to the global interface structure (`gModuleNameIfType`).
- **`config`**: Reference to the global configuration structure (`gModuleNameConfigType`).
- **`recipe`**: Reference to the global recipe structure (`gModuleNameRecipeType`).

## Coding Best Practices

### Function Blocks
- Call function blocks at the end of the scan or outside state machines.
- Assign parameters before the call.
- Example:
  ```iec-st
  this.TON_0.PT := config.delayTime;
  this.TON_0.IN := TRUE;
  this.TON_0();
  ```

### Edge Detection
- In indexed modules, do not use implicit `EDGEPOS`/`EDGENEG`. Explicitly track old values.

### Alarm Handling
- Use `BrdkAlarmControl` function blocks within an `alarm` action.
- Check reactions using `MpAlarmXCheckReaction`.

## Examples

### Type Declaration (.typ)
```iec-st
TYPE
    (* Main local structure *)
    localType : STRUCT
        em : EquipmentModule;
        hw : localHwType;
        hmi : hmiType;
        state : INT;
    END_STRUCT;

    (* Enumeration *)
    myStateEnum :
    (
        MYSTATE_IDLE := 0,
        MYSTATE_RUNNING := 10,
        MYSTATE_ERROR := 99
    );
END_TYPE
```

### Variable Declaration (.var)
```iec-st
VAR
    local : localType;
    this : REFERENCE TO localType;
    hmi : REFERENCE TO hmiType;
    TON_Watchdog : TON;
END_VAR

VAR CONSTANT
    MAX_ITEMS : USINT := 10;
END_VAR
```

### Indexed Module Cyclic Code (.st)
```iec-st
PROGRAM _CYCLIC
    
    localNonIndexed; // Run non-indexed code first

    FOR i := 0 TO NUM_INSTANCES DO
        (* Update References *)
        this      ACCESS ADR(local[i]);
        em        ACCESS ADR(local[i].em);
        hmi       ACCESS ADR(local[i].hmi);
        interface ACCESS ADR(gModuleNameInterface[i]);
        config    ACCESS ADR(localConfig[i]);
        recipe    ACCESS ADR(localRecipe[i]);

        (* State Machine *)
        CASE em.ModeID OF
            MODE_PRODUCTION:   production;
            MODE_MAINTENANCE:  maintenance;
            MODE_MANUAL:       manual;
        ELSE
            em.Command.StateComplete := TRUE;
        END_CASE

        (* FB Calls *)
        em();
        alarm;
    END_FOR;

END_PROGRAM
```

### Alarm Action (alarm.st)
```iec-st
ACTION alarm:
    (* Configure Alarm *)
    this.alarm.BrdkAlarmControl_[0].Enable    := TRUE;
    this.alarm.BrdkAlarmControl_[0].MpLink    := ADR(mpAlarmXCore);
    this.alarm.BrdkAlarmControl_[0].Name      := 'MyAlarmName';
    this.alarm.BrdkAlarmControl_[0].Condition := NOT interface.status.isReady;
    this.alarm.BrdkAlarmControl_[0]();

    (* Handle Reactions *)
    IF MpAlarmXCheckReaction(mpAlarmXCore, 'stop') THEN
        em.Command.Stop := TRUE;
    END_IF;
END_ACTION
```
