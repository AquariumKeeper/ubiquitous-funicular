# Attribution and limits

Character Planner is an unofficial fan tool. It is not affiliated with, endorsed by or
sponsored by Ubisoft or Massive Entertainment. Tom Clancy's The Division 2 is a trademark
of Ubisoft Entertainment; game content and artwork belong to their owners.

## Game data

Gear, weapon, attribute, set, talent, skill and weapon-mod compilations are adapted from
[Division 2 Community Data Hub game data](https://github.com/div2hub/game-data), licensed
under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/)
(`licenses/division2-data-CC-BY-4.0.txt`). That license covers the Hub's compilation, not
Ubisoft or Massive game content.

The planner changes the data: it renames source IDs, converts percentages to fractions,
maps slot compatibility to planner choices, turns set and mod entries into stat
modifiers, and applies reviewed corrections. Corrections cite Ubisoft's published patch
notes and research checked against in-game readings.

Other community references:

- Skill damage and cooldown baselines: the community Division 2 gear spreadsheet (Skill
  List, "Last Updated by IC3SO TU27").
- Armor and health baselines: the
  [level-40 armor table](https://www.f-space.de/mdwiki/spiele/division2.html) and the
  [Division 2 gear calculator](https://github.com/ImThatTeriyaki/Tom-Clancy-The-Division-2-Gear-calculator).

## Artwork

Game artwork belongs to Ubisoft / Massive Entertainment; no separate artwork license is
asserted. The images come from the community Division 2 gear spreadsheet, The Division
Wiki contributors (some weapon renders cleaned up with AI tools, so fine markings may
differ) and screenshots supplied by the project maintainer. Each image's source is
recorded in the pack's artwork registry (`data/packs/division-2.json`).

Three icons come from [mxswat/mx-division-builds](https://github.com/mxswat/mx-division-builds),
whose repository license is CC BY-NC-SA 4.0
(`licenses/artwork-mxswat-CC-BY-NC-SA-4.0.txt`).

## What the planner estimates

The planner estimates configured builds and the conditions you set in the Assumptions
tab, and its combat simulator plays them through timed fights. Known limits:

- **Weapon falloff:** damage between optimal and long range isn't modeled.
- **Enemy targets:** time to kill ignores headshots, falloff, cover and enemy
  resistances.
- **Skills:** some Expertise aggregation, Hive charge recovery, Reviver armor stacking
  and support-buff aggregation are unresolved, and only reviewed skill mods are offered.
- **Talents:** some set and talent interactions aren't modeled; an unmodeled effect keeps
  its description.

`docs/IN_GAME_CHECKLIST.md` lists the in-game readings that would settle the open values.
