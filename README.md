# LibAbilityLogger

A developer-focused logging addon for **The Elder Scrolls Online**. It prints a readable log of combat and buff/debuff events to chat, keeps a short history and per-ability damage/healing totals, and exposes a small API so other addons can consume the same events.

## Features

- Combat log lines showing the source, ability, target, result, hit value, critical hits, and the cost of your own casts
- Buff and debuff logging (gained, updated, faded, stack counts)
- Filters for category, source/target scope, minimum hit value, and ignored abilities
- A history buffer you can review in chat with `/lab recent`
- Per-ability damage and healing totals with `/lab stats`
- A listener API so other addons can receive structured event data
- Filter settings saved between sessions

## Requirements

- The Elder Scrolls Online, API version 101046 (Update 46)
- The manifest declares `LibAddonMenu-2.0` as a dependency, but the current code does not use it

## Installation

1. Download or clone this repository.
2. Copy the `LibAbilityLogger` folder into your AddOns directory:
   - Windows: `Documents\Elder Scrolls Online\live\AddOns\`
3. Make sure the folder contains `LibAbilityLogger.addon` and `LibAbilityLogger.lua`.
4. Enable **LibAbilityLogger** in the in-game AddOns menu and reload the UI.

## Quick start

Type `/lab` in chat to turn logging on. Each combat or buff/debuff event is then printed as one line. Type `/lab` again to turn it off.

```
[LibAbilityLogger] [combat] You used Frost Staff (12345) on Target Dummy: damage 1234 (critical), cost 500 Magicka
[LibAbilityLogger] [effect] Effect 'Minor Courage' (5555) gained on Tester [player], slot 3
```

## Slash commands

| Command | Description |
|---|---|
| `/lab` | Toggle logging on or off |
| `/lab on` / `/lab off` | Turn logging on or off |
| `/lab status` | Show the current logging state and settings |
| `/lab combat on\|off` | Log or hide combat events |
| `/lab effects on\|off` | Log or hide buff and debuff events |
| `/lab scope all\|self\|incoming` | Limit combat events (see below) |
| `/lab minhit <value>` | Hide combat events whose hit value is below `<value>` |
| `/lab ignore <abilityId>` | Stop logging an ability |
| `/lab unignore <abilityId>` | Resume logging an ability |
| `/lab stats [count]` | Show damage and healing you dealt, per ability (default 10 rows) |
| `/lab recent [count]` | Show the most recent logged entries (default 10) |
| `/lab clear` | Clear the history and the stats |
| `/labhelp` | List the commands |

### Scopes

- **`all`** (default): every combat event the game reports.
- **`self`**: only events where you or your pet are the source.
- **`incoming`**: only events where you are the target.

Effect events do not include a source. For buffs and debuffs, any scope other than `all` only shows effects on your own character.

### Stats

`/lab stats` counts damage and healing whose source is you or your pet. Damage over time and healing over time ticks are included, recorded under the ability ID the game reports for each tick. Stats are per session and are cleared with `/lab clear` or when the UI reloads.

## Settings

Filter settings are saved in the `LibAbilityLogger_SavedVars` SavedVariable, so they persist between sessions:

| Setting | Default | Meaning |
|---|---|---|
| `combat` | `true` | Log combat events |
| `effects` | `true` | Log buff and debuff events |
| `scope` | `"all"` | `all`, `self`, or `incoming` |
| `minHitValue` | `0` | Minimum hit value to log |
| `historySize` | `200` | Number of entries kept in memory |
| `ignored` | `{}` | Set of ability IDs to skip |

Logging itself is not saved. It starts off each time the UI loads.

## Developer API

The addon is available as the global `LibAbilityLogger`.

### Printing your own messages

```lua
LibAbilityLogger:Print("Value is %d", someValue)
```

`Print` writes to chat only while debug logging is on. It uses `string.format` syntax.

### Debug state

```lua
LibAbilityLogger:EnableDebug()
LibAbilityLogger:DisableDebug()
LibAbilityLogger:ToggleDebug()
local on = LibAbilityLogger:IsDebugEnabled()
```

### Listening for events

```lua
local function OnLogEntry(entry)
    if entry.category == "combat" and entry.kind == "damage" then
        -- entry.value is the damage amount
    end
end

LibAbilityLogger:RegisterListener(OnLogEntry)
-- later:
LibAbilityLogger:UnregisterListener(OnLogEntry)
```

Listeners run for every captured entry. Entries are captured only while debug logging is on.

Each entry is a table with these fields:

| Field | Combat | Effect | Description |
|---|---|---|---|
| `time` | yes | yes | `GetGameTimeMilliseconds()` when captured |
| `category` | yes | yes | `"combat"` or `"effect"` |
| `kind` | yes | yes | `"damage"`, `"healing"`, `"effect"`, or `"other"` |
| `abilityId` | yes | yes | Ability ID (0 if unknown) |
| `abilityName` | yes | yes | Ability or effect name |
| `sourceName` / `sourceType` | yes | no | Combat source |
| `targetName` / `targetType` | yes | no | Combat target (effects use `targetName` only) |
| `value` | yes | no | Hit value |
| `result` | yes | yes | Human-readable result, such as `"damage"` or `"gained"` |
| `critical` | yes | no | `true` for critical results |
| `message` | yes | yes | The text that was printed to chat |

### Querying data

```lua
local recent = LibAbilityLogger:GetRecent(20)  -- array of the last 20 entries
local stats = LibAbilityLogger:GetStats()      -- array of per-ability records, highest total first
LibAbilityLogger:ResetSession()                -- clear history and stats
```

Each stats record has `abilityId`, `name`, `damage`, `damageHits`, `healing`, and `healingHits`. The records are live, so copy them before changing them.

## Known limitations

- Captures only happen while debug logging is on. The addon does not record events in the background.
- Ability names come from `GetAbilityName`. If that returns nothing, the name from the combat event is used.
- Result types that the addon doesn't have a label for show the game's label when one is available, or the numeric result ID otherwise.

## Files

| File | Purpose |
|---|---|
| `LibAbilityLogger.addon` | Addon manifest (title, API version, SavedVariables) |
| `LibAbilityLogger.lua` | Addon code |
| `README.md` | This file |

## Author

Vixen Hunny

## License

No license has been specified for this project yet.
