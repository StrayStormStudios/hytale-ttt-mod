# Trouble in Trork Town (TTT) - Configuration Guide

This guide is auto generated and documents installation, runtime configuration, map templates, commands, and permissions
for the current
codebase (`v0.6.4`).

- Core config schema expanded with new round timing, karma, map pool, role, and death-remains settings.
- Weapons config schema changed to `WeaponTypes` + `LootTables` (with optional `MaxItems`).
- Map template requirements expanded to include `config.json`, `instance.bson`, and `preview.png`.
- Built-in map scaffold added under `src/main/resources/templates/map/`.
- Command and permission surface expanded (`/ttt info`, `/ttt map config spawn-point`, `/ttt map save`, debug toggles,
  new permission nodes).
- Command package structure was reorganized to `ecs/commands`.

## Requirements

- Hytale server/runtime
- Java 25
- Gradle wrapper included (`./gradlew`)

## Build From Source

```bash
git clone https://github.com/kuttz-dev/hytale-ttt-mod.git
cd hytale-ttt-mod
./gradlew build
```

Current relevant `gradle.properties` values:

```properties
version=0.6.4
java_version=25
patchline=release # or pre-release
load_user_mods=false
hytaleServerVersion=2026.03.05-9fdc5985d
```

## Running a Development Server

The mod includes a run configuration for testing:

```bash
# Start a local development server
./gradlew runServer
```

This creates a `run/` directory with server files.

## Runtime Config Files

Generated on first run under:

```text
<Server>/config/ncode/ttt/config.json
<Server>/config/ncode/ttt/weapons_config.json
```

## `config.json` (Global Gamemode Config)

### Keys and defaults

| Key                                      |                               Default | Notes                              |
|------------------------------------------|--------------------------------------:|------------------------------------|
| `RequiredPlayersToStartRound`            |                                   `3` | Minimum players to start           |
| `TimeBeforeRoundInSeconds`               |                                  `10` | Pre-round countdown                |
| `RoundDurationInSeconds`                 |                                 `600` | Active round duration              |
| `TimeAfterRoundInSeconds`                |                                   `5` | Post-round delay                   |
| `TimeToVoteMapInSeconds`                 |                                  `30` | Map voting window                  |
| `TimeBeforeChangingMapInSeconds`         |                                   `5` | Delay before transition            |
| `KarmaStartingValue`                     |                                `1000` | Initial karma                      |
| `KarmaForDisconnectingMiddleRound`       |                                `-100` | Disconnect penalty                 |
| `KaramPointsForKillingSameRoleGroup`     |                                 `-10` | Team-kill karma penalty            |
| `KaramPointsForKillingOppositeRoleGroup` |                                  `10` | Correct-kill karma gain            |
| `PlayerGraveId`                          |                      `"Player_Grave"` | Grave entity ID                    |
| `LootBoxBlockId`                         | `"Furniture_Human_Ruins_Chest_Small"` | Loot marker block                  |
| `RoundsPerMap`                           |                                   `8` | Rounds before map cycle            |
| `MapsInARowForVoting`                    |                                   `3` | Number of voting choices           |
| `WorldTemplatesFolder`                   |                `"universe/templates"` | Templates base path hint           |
| `Roles`                                  |                        built-in array | Role definitions (see below)       |
| `PlayersLeaveRemainsWhenDie`             |                                `true` | Enables corpse/gravestone remains  |
| `PlayersLeaveGravestonesWhenDie`         |                               `false` | Use gravestones instead of corpses |
| `GravestonesHaveNameplates`              |                                `true` | Floating nameplate on gravestones  |

### Roles (`Roles`)

Each role entry supports:

- `Id`
- `TranslationKey`
- `CustomBackgroundColor`
- `MinimumAssignedPlayersWithRole`
- `MaxAssignedPlayersWithRole`
- `RoleGroup` (`INNOCENT` or `TRAITOR`)
- `SecretRole`
- `Ratio`
- `StartingItems`
- `StoreItems`
- `StartingCredits`
- `PublicRoleMessagesPrefix`

Defaults include `detective`, `innocent`, and `traitor`.

### Item string format

For `StartingItems` and `StoreItems`:

- Single item: `ItemId:Amount`
- Bundle in one slot: `ItemA:1|ItemB:2`

## `weapons_config.json`

### Top-level keys

- `WeaponTypes`: array of weapon category rules
- `LootTables`: array of loot table definitions

### `WeaponTypes` entry

- `TypeId` (string)
- `ItemIds` (string array)
- `AllowedItemsOfSameType` (int, default `1`)

### `LootTables` entry

- `Id` (string)
- `Items` (array of loot items)
- `MaxItems` (optional int)

### Loot item entry

- `Probability` (int, default `100`)
- `Amount` (int, default `1`)
- `ItemId` (string)
- `Includes` (array of extra items with `ItemId` + `Amount`)

## Map Templates

Maps are loaded from plugin data directory:

```text
<Server>/config/ncode/ttt/maps/<map_name>/
```

Each map folder must contain:

- `chunks/`
- `config.json`
- `instance.bson`
- `preview.png`

Template scaffold in repo:

```text
src/main/resources/templates/map/
```

### Map `config.json` schema (`InstanceConfig`)

- `LootSpawnPoints` (array)
- `PlayerSpawnPoints` (array)
- `IsMapDestructibleByExplosions` (boolean, default `true`)

`SpawnPoint` shape:

- `Position` (`x`,`y`,`z` as `Vector3d`)
- `Rotation` (`x`,`y`,`z` as `Vector3f`)

`LootSpawnPoint` shape:

- `SpawnPoint`
- `LootTables` (string array of loot table IDs)
- `Probability` (int, default `100`)

Minimal example:

```json
{
	"LootSpawnPoints": [],
	"PlayerSpawnPoints": [],
	"IsMapDestructibleByExplosions": true
}
```

## Commands

Registered root commands:

- `/t <message>`

### `/ttt` subcommands

- `shop` (aliases: `store`, `buy`)
- `role set <role> [targetPlayer]`
- `credits set <credits> [targetPlayer]`
- `info <targetPlayer>`
- `spawn add`
- `spawn show`
- `loot spawn add [probability]`
- `loot spawn show`
- `loot spawn force`
- `map vote` (alias: `votemap`)
- `map finish` (alias: `end`)
- `map config spawn-point`
- `map save`
- `map create <name>` (alias: `add`)
- `map read [name]` (aliases: `list`, `ls`, `show`)
- `map update <oldName> <newName>` (alias: `rename`)
- `map delete <name> [confirm]` (aliases: `del`, `rm`)
- `debug toggle gamemode`
- `debug toggle blocks`
- `debug toggle gravestones`
- `debug toggle entities`
- `debug toggle pickup`
- `debug get-position`
- `debug memory`
- `debug info`

## Permissions

### Nodes

- `ttt.traitor.chat`
- `ttt.map.vote`
- `ttt.map.finish`
- `ttt.map.config.spawn-point`
- `ttt.map.save`
- `ttt.map.crud`
- `ttt.shop.open`
- `ttt.info.see`
- `ttt.role.set`
- `ttt.credits.set`

### Permission groups

- `ttt.groups.user`
- `ttt.groups.admin`

At startup, permissions are assigned as:

- `ttt.groups.user`: `ttt.map.vote`, `ttt.shop.open`, `ttt.traitor.chat`
- `ttt.groups.admin`: user permissions plus `ttt.credits.set`, `ttt.info.see`, `ttt.role.set`, `ttt.map.finish`,
  `ttt.map.crud`, `ttt.map.config.spawn-point`
- `ttt.map.save` exists as a node and is required by `/ttt map save`, but it is not included in default user/admin group
  assignments.

## Localization

Language files:

```text
src/main/resources/Server/Languages/en-US/ncodeTTT.lang
src/main/resources/Server/Languages/es-AR/ncodeTTT.lang
```

## Notes

- If commands/config are changed in code, this file should be updated from source (`CustomConfig`, `WeaponsConfig`,
  `InstanceConfig`, `CustomPermissions`, and `ecs/commands`).

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request+

### Adding a New Language

1. Copy `en-US/ncodeTTT.lang` to your language folder (e.g., `nl-NL/`)
2. Translate the values
3. Rebuild and install