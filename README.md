# Character Planner for The Division 2

A build planner for Tom Clancy's The Division 2. Equip and configure gear, weapons,
talents, specialization, skills and SHD Watch ranks, and see how armor, health, weapon
damage, DPS and time to kill respond, so you can plan a build before spending scarce
in-game resources on it.

> Unofficial fan tool. Not affiliated with, endorsed by or sponsored by Ubisoft or
> Massive Entertainment. Tom Clancy's The Division 2 is a trademark of Ubisoft
> Entertainment; game content and artwork belong to their owners.

## Features

- Configured equipment in an in-game-style layout: attribute, mod and talent rolls,
  brand and gear-set bonuses, named and exotic items.
- 1,022 equipment entries, 373 talents (207 modeled directly), six specializations and
  43 skill variants, all with skill previews.
- Headline estimates: practical weapon DPS, skill DPS, Build DPS, time to kill against
  selectable enemy targets, Effective HP and Time to Armor Break.
- Stat breakdowns that show every formula, input and bonus.
- Build library, share codes, side-by-side comparisons, equipment-preview deltas and
  undo/redo.
- A headless CLI (`planner`) for scripts and agents, which can also drive a running
  planner window. See [CLI usage](docs/CLI.md).

The Windows desktop app (Go and Fyne) is the main planner. A web planner (Vite, React
and TypeScript) shares the same game pack and calculation contracts.

## Download and run

Download the Windows ZIP and its `.sha256` checksum from a release, extract the whole
ZIP, and run `character-planner.exe`. Keep `data/` and `licenses/` beside the
executable. No Go, Node, Python or source checkout is needed, and Windows needs a
graphics driver with OpenGL 2.0 or newer. Builds are saved in your user configuration
directory under `character-planner`.

`planner.exe` in the same folder is the CLI: run `planner.exe --help`, or see
[CLI usage](docs/CLI.md).

## Accuracy

The planner estimates configured builds and the conditions you assert in its
Assumptions tab. It does not simulate combat events, timers or enemy behavior. Values
come from a pinned community catalog and from research checked against
in-game readings; 

Known gaps:

- **Weapon falloff:** long and maximum range are shown, but damage between optimal
  and long range is not modeled. Six catalog rows have no game ranges yet.
- **Enemy targets:** health and armor are source formula values, and named enemies
  are linked to missions. Time to kill ignores headshots,
  falloff, cover and enemy resistances.
- **Skills:** some Expertise aggregation, Hive charge recovery, Reviver armor stacking,
  support-buff aggregation and contested source values are unresolved. Only reviewed
  skill mods are offered.
- **Talents:** some set and talent interactions, and event or timer behavior, are not
  modeled. Unmodeled effects keep their source description.

Players can help settle the open questions: the [in-game checklist](docs/IN_GAME_CHECKLIST.md)
lists the item-card and timing readings that would decide them.


## Status and remaining work

Remaining priorities:

- Expand talent and skill coverage.
- Add the ability to simulate an entire party, so team member buffs are represented.
- Improve dps and ehp calculations to better reflect in game experience.
- **Verification:** resolve the remaining named and exotic discrepancies and
  level or version mismatches using game evidence. 
- **Weapon falloff:** measure the falloff (if any) in game.
- **Skills and talents:** accuracy questions, and
  more reviewed skill mods.
- **Maintenance:** keep simplifying the importer and duplicated test fixtures.
