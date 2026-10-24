# Headless planner CLI

The `planner` command discovers catalog IDs, calculates builds and edits build
documents without starting Fyne. It requires no CGO or game installation. It
can also inspect and edit the active build in a running desktop through the
`session` commands. Library commands use either an explicit file or opt-in access
to the desktop's shared library. Current desktop builds use the same transaction
locks; older desktop writers must be closed.

**In a release** (G367), `planner.exe` sits beside `character-planner.exe` and
finds the bundled pack itself, so `--pack` is not needed:

```text
planner.exe --json catalog list items --query sawyer
planner.exe --json session inspect
```

Scripts and agents should use these commands rather than the desktop's private
session transport: the CLI handles session discovery, framing and the session
token, and its JSON envelope, edit operations and exit codes are the supported
interface. `planner.exe --json version` reports the release version.

**From a source checkout**, build from `desktop/` with
`CGO_ENABLED=0 go build ./cmd/planner` (Windows PowerShell: set
`$env:CGO_ENABLED = '0'` first). Without `--pack`, the command looks for
`data/packs/division-2.json` beside its executable, then under the current
directory, then under its parent. A development build in `desktop/` has no pack
beside it, so the examples below pass `--pack ../data/packs/division-2.json`, the
path from `desktop/`. In a release, leave `--pack` out of every example.

## Commands and machine output

Global flags go before the command:

```text
planner --help
planner --json version
planner --json describe
planner --json --pack ../data/packs/division-2.json catalog list items --query doctor --slot primary_weapon
planner --json --pack ../data/packs/division-2.json catalog show items primary_weapon_doctor_home
planner --json --pack ../data/packs/division-2.json build new --name Baseline --output build.json
planner --json --pack ../data/packs/division-2.json build resolve --input build.json
planner --json session list
planner --json session inspect
planner --json session undo --expected-revision 7
```

`describe` (G370) returns the interface as JSON: the command synopses, the
session commands, every edit operation with its required fields (`fields`, plus
exactly one of `oneOf` when present, or `clearFields` with `"clear": true`),
the catalog kinds and the exit codes. It reads the same tables the commands
validate against. Errors for an unknown command, operation kind, field or
catalog kind list the valid values.

Catalog kinds are `items`, `slots`, `attributes`, `mods`, `talents`, `skills`
(active skill variants), `specializations` and `upgrades` (account ranks).
`list` supports a case-insensitive ID/name query. `--slot` filters items by
equipment compatibility or active skills by their loadout slot; it is rejected
for other kinds. `show` returns the full authored definition, including bounds
and eligible choices. Since G371, `--hosts ID` lists only the items whose
attribute, mod or talent slots offer that ID, such as every item that can carry
a talent, and item entries list the `sets` (sets and brands) they belong to.

`build inspect`, `validate`, `resolve`, `share` and `export` take exactly one
`--input PATH` or `--share CODE`. Use `--input -` for stdin. `build import`
requires `--share CODE`. A share code may begin with `#b=`; a full URL is not
accepted. `--output PATH` saves the versioned build document. Existing files are
refused unless `--force` is explicit; replacement uses a same-directory temporary
file and atomic publication. Failed validation writes no output document.

Machine output is one JSON envelope with `apiVersion: 1`, `command`, `ok`,
`diagnostics`, and applicable `pack`, `document`, `context`, `result` or `error`
fields. `build resolve` returns stats, skill previews, point budgets and modifier
breakdowns in `result`; `build share` returns `result.code`. Errors use the same
envelope on stdout. Incidental failures writing stdout go to stderr. Without
`--json`, successful documents/results are indented JSON and errors go to stderr.

| Exit | Meaning |
| --- | --- |
| 0 | Success; warning diagnostics may still be present |
| 2 | Invalid command, flags, catalog kind or catalog ID |
| 3 | Invalid pack/build/context, share code or edit operations |
| 4 | Unavailable input/output, failed persistence or output stream |
| 5 | Internal invariant or serialization failure |
| 6 | Stale live-session revision; the result may include the authoritative current snapshot |

## Live desktop sessions

```text
planner --json session list [--session-dir PATH]
planner --json session inspect [--session ID] [--session-dir PATH]
planner --json session resolve [--session ID] [--session-dir PATH]
planner --json session apply [--session ID] --expected-revision N --operations PATH|- [--session-dir PATH]
planner --json session undo|redo [--session ID] --expected-revision N [--session-dir PATH]
planner --json session history [--session ID] [--session-dir PATH]
planner --json session inspect --wait DURATION [--since-revision N] [--session ID] [--session-dir PATH]
```

Global flags such as `--json` go before `session`. Session commands reject
`--pack`: the running desktop session supplies the pack and build. `--operations`
uses the same non-empty ordered JSON edits as `build apply`; `-` reads it from
stdin. `--expected-revision` is required for apply, undo and redo, and zero is a
valid explicit revision.

By default, discovery reads
`os.UserConfigDir()/character-planner/sessions`. Set
`CHARACTER_PLANNER_SESSION_DIR` to an absolute path to select another registry,
or pass `--session-dir PATH` on an individual command. A missing registry lists
as an empty array and is not created by listing. The desktop publishes a
per-user private registry entry and a local authenticated endpoint; session
tokens and endpoint addresses are not included in CLI output.

Set the same override before launching the desktop and CLI when using a custom
registry. Unix socket paths have a short platform limit; use a short private
absolute directory if the default path is too long. The registry filesystem must
support hard links for atomic descriptor publication. Closing a window removes
its own entry. After a crash, stale entries remain listed as unavailable and are
not removed automatically. If hosting fails, the desktop continues without live
session control and reports the failure on stderr.
Discovery accepts at most 128 registrations. If old crash records fill the
registry, close all planner windows and remove the stale `.json` entries from
that directory before restarting the desktop.

`session list` authenticates an inspect request to every registered session and
returns summaries with ID, process ID, pack ID/version, start time and
availability, plus each available window's `buildName` and `revision` (G371). `inspect`, `apply`, `undo` and `redo` accept an explicit `--session
ID`. If omitted, the CLI selects the session only when exactly one available
session responds; zero or multiple available sessions return exit 4. Listing
reports unavailable registrations without exposing transport details.

**Assumptions (G368).** A `context` operation sets a view assumption, such as
stacks, a buff condition or the enemy target, instead of the build. A numeric
input takes `value`, which must be within its range, on its step and, for a
dropdown input, one of its options; a condition takes `text`, one of its values
(`true`/`false` gates accept either). `clear` restores the default. Later
operations in the same batch resolve with the new assumptions. In a session,
build and assumption edits in one `apply` are one undo step. `build apply`
returns the updated assumptions in the envelope's `context`; save them and pass
them back with `--context`, because build documents never store assumptions.

Session snapshots and `build resolve`/`apply` results list `assumptions`: every
input and condition that applies to the build (its equipped effects, talents,
sets, skills and specialization, every global input and the weapon selector),
with `kind` (`numeric` or `condition`), the current `value` (empty when the
default applies), the numeric `default`, `min`, `max`, `step` and `options`, a
condition's `values`, and the desktop Assumptions tab `section`. Since G380 each
also lists its `sources`: the parts of the build that read it, each with `kind`
(`item`, `talent`, `mod`, `set`, `skill`, `specialization`, `specializationNode`,
`upgrade` or `passive`), `id`, `name` and, for equipped and skill sources, `slot`.
A global input or the weapon selector that nothing equipped reads has none.

Resolve results are readable without the pack (G371): `names` maps every catalog
ID the build uses, skill mod choices included (G376), to its display name, and `sets` reports each set and brand
with active pieces: `pieces` (including talent `bonusPieces`), every bonus
`thresholds` and the `activeBonuses` reached. Diagnostics about an item's
attribute, mod or talent slot give the equipment `subject` and the slot as
`target`, matching an edit operation's `slot` and `target`.

Resolve results also list `weapons` (G375): one entry per equipped weapon slot
with its `slot`, `itemId`, `name`, whether it is the `active` slot in the
context, and its `equipped_weapon_*` `stats` resolved with that slot active and
every other assumption unchanged. The top-level `stats` still describe the
context's selected weapon slot.

When a build assumption is set away from its default, results also include
`unconditional` (G381): the build resolved with those assumptions back at their
defaults, so a conditional amplifier's effect reads next to the plain value.
`reset` lists the assumptions it returned to their defaults; `stats` lists only
the stats whose value differs from the top-level `stats`; `buildOutputs` lists
every build output. Global inputs such as the enemy target, and the weapon
selector, keep their values.

**Live calculation (G369).** `session resolve` returns the running window's
own calculation for its current build and assumptions in `result.resolution`:
the same stats, breakdowns, skill previews, build outputs, point budgets and
assumptions as `build resolve`. It takes no revision and changes nothing; the
envelope's `revision` says which state was calculated.

**Waiting and history (G382).** `session inspect --wait 30s` returns as soon as
the window's revision changes, whether from the desktop or another client, or
when the wait runs out. It waits past the revision found when the command
starts, or past `--since-revision N`, which also returns at once if the session
has already moved on. `result` adds `sinceRevision` and `changed`; running out
of time is not an error. Each request waits at most four seconds, below the
five-second call timeout, and the CLI repeats it for longer waits.

`session history` lists the edit history, oldest first, as `result.history`:
each entry's `revision` when recorded, its `origin` (`initial`, `desktop`, or
`session` for an edit through the CLI or another session client), a `label`
naming a session edit's operations, and `current` on the state shown now.
Entries after the current one are the redo branch. Undo and redo move
`current` without adding entries, and the desktop keeps the latest 100 states.

Live results use the same versioned envelope and include the authoritative
document, context, revision and diagnostics. `result` includes `sessionId`,
`canUndo` and `canRedo`. A stale revision returns exit 6 and includes the newer
snapshot when available. Session calls have a five-second timeout, and the CLI
does not retry mutations automatically. If a transport error says the outcome
may be unknown, inspect the session's current revision before deciding whether
to retry. A missing session never falls back to editing a build file.

## Comparing builds

```text
planner --json --pack PACK build compare --left before.json --right after.json
planner --json --pack PACK build compare --left before.json --right after.json --context context.json
```

Both documents must match the selected pack ID and version and pass validation.
The same context is used for both resolutions. At most one of `--left`, `--right`
and `--context` can be `-` for stdin. Comparisons write no build or library files.
Share codes can first be imported into documents with `build import`.

`result.left` and `result.right` contain the complete calculation results;
`leftDiagnostics` and `rightDiagnostics` retain each side's warnings. Invalid
builds fail with exit 3 and preserve side-specific error diagnostics in the
result. `statDeltas` is sorted by stat ID and reports right minus left.
`skillPreviewDeltas` is sorted by slot, skill ID and output ID: different variants
remain separate, and a numeric delta requires matching formats on both sides.
Each side has `present` and a nullable `value`, distinguishing absence from zero.
`delta` is omitted when a comparable pair is missing. This compares calculated
effects, not every configuration field or unmodeled game behavior.

## Build library and desktop sharing

```text
planner --json --pack PACK library list --library library.json
planner --json --pack PACK library save --library library.json --id baseline --input build.json
planner --json --pack PACK library save --library library.json --id baseline --input revised.json --replace
planner --json --pack PACK library load --library library.json --id baseline --output loaded.json
planner --json --pack PACK library delete --library library.json --id baseline
planner --json --pack PACK library list --desktop-library
planner --json --pack PACK library save --desktop-library --id baseline --input build.json
```

Choose `--library PATH` or explicitly opt into `--desktop-library`. The latter
uses the same path resolver as the current desktop app:
`os.UserConfigDir()/character-planner/builds.json`. Omitting both is an error;
they cannot be selected together. Resolving or listing a missing library does not
create it. The normal user config directory is platform-specific; use an explicit
path when operating on another user's data or a copied library.
`save` accepts exactly one `--input PATH|-` or `--share CODE`. Records are keyed
by explicit IDs; saving an existing ID requires `--replace`. Replacement may
insert a missing ID, and may replace a record saved under an earlier version of
the same pack, but cannot change a record's pack ID.
`load` uses `--force` only to replace an existing output document. Its output
must not alias the library file. `delete` is explicit and rejects missing IDs.

The store uses the desktop's array-of-build-records format, while load/save
responses contain the versioned CLI document. `list` returns summaries sorted
by ID for the current pack ID. Since G391 that includes builds saved under
another version of the pack, marked `otherVersion`: `load` returns them relabeled
to the current version, with a `pack-version-relabeled` warning, after the usual
validation, and `delete` removes them. Other-pack records remain in the file;
loading, replacing or deleting them through the selected pack fails.
Current-pack builds are validated before save and load. Context is not persisted.
Missing files read as empty; malformed, ambiguous, null or oversized libraries
fail instead of being overwritten as empty. Library input/output is bounded to
8 MiB. IDs and saved UTC timestamps are validated by the strict codec.

Cooperating CLI writers hold `PATH.lock` for the entire read-modify-write and
publish through a synced same-directory temporary file and atomic replacement.
A busy lock returns exit 4 without mutation. Locks are never broken automatically;
after a crash, confirm no writer is active before removing its stale lock.
Readers observe an atomic file snapshot. The desktop shipped from G269 onward
uses this same repository and transaction lock for build-library mutations.
Older desktop versions and external editors do not cooperate with these locks;
close them before shared writes.

In the desktop Build Library dialog, **Refresh** reloads CLI-written records.
Load re-reads the selected ID and validates it before replacing the active build;
deleted or invalid records cannot be loaded from an old row snapshot. Read/write
errors remain visible and failed operations preserve state and stored data.
The Compare dialog rejects hard-invalid builds rather than rendering misleading
deltas. Library writes do not change the active desktop build until the user
loads a record. There is no background file watcher; use the live `session`
commands to inspect or change the active desktop build.

## Documents and assumptions

Saved CLI documents use `{ "schemaVersion": 1, "build": { ... } }` with explicit
lower-camel build fields. They preserve configured equipment, rolls, selections,
upgrades, metadata and reference-pack scenario/passive state. This is a separate
document format from the desktop library's array of records and from the
compressed share code. Do not pass a library file as a CLI document.

The document's pack ID and content version must exactly match the selected pack.
Older builds require an explicit reviewed migration; the CLI does not relabel
them silently. Unknown/miscased fields, duplicate JSON keys, trailing values and
non-finite numbers are rejected. Inputs and decoded share payloads are bounded
to 8 MiB. Missing optional collections are normalized to empty collections.

Since G390 a build can mark equipped items as Prototype quality in an optional
`prototypes` map, keyed by slot: `{"itemId": ..., "augment": ..., "level": ...}`.
The record names the equipped item, so it no longer applies after a swap. An
empty `augment` means none has been chosen; otherwise `level` is 1 to 10.
Attribute rolls on a Prototype item use its Prototype range. Share codes with
Prototype items use codec version 12, which the web planner does not read yet.

Since G395 a build can list up to three party members in an optional `party`
array: `{"name": ..., "code": ..., "armor": ..., "effects": [{"id": ...,
"values": {stat: value}, "active": ...}]}`. `code` is the share code the
member's buffs were read from; a member without one was entered by hand, with
`armor` as the giver armor its armor-share buffs use. Each effect names a pack
ally effect, its values as evaluated for that member and whether it is ticked
on. Ticked effects apply to the build, the strongest per kind. Share codes
never carry the party.

Use `--context context.json` for a separate JSON string map, for example:

```json
{ "weapon_slot": "primary_weapon", "in_cover": "true" }
```

Keys and categorical values must occur in the selected pack's requirements or
numeric-input definitions; known boolean switches accept both `true` and `false`
strings. Invalid numeric bounds/steps are reported by the
engine. Context is never included in saved documents or share codes. Build and
context cannot both consume stdin in one command. These are explicit calculation
assumptions, not observed game state or a combat simulation.

## Upgrade ranking

`build rank --slot ID --stat ID` (G397) tries every legal attribute, mod and talent
choice on the item equipped in a slot, one at a time, and lists each with its
change to a derived stat, largest gain first. Attributes and mods use their
highest legal roll, so the current attribute or mod also appears, marked
`reroll`, when its roll is below that. Choices that add a build error, such as a
repeated attribute, are left out. A weapon slot is ranked as the active weapon.

```text
planner --json build rank --input build.json --slot chest --stat character_armor --context context.json
```

Each entry has `kind` (`attribute`, `mod` or `talent`), `target` and
`targetName` (the item's slot), `id` and `name`, the `value` rolled, `before`,
`after` and `delta`. Apply one with the matching `attribute`, `mod` or `talent`
edit operation.

## Combat simulation

`build simulate` (G405) plays the build through a fixed-length fight against a
stream of the selected enemy target, re-targeting after each kill, with skills at
their headline rates. See the [combat simulator plan](COMBAT_SIM_PLAN.md).

```text
planner --json build simulate --input build.json --duration 60 --iterations 500 --seed 1
planner --json build simulate --input build.json --average
```

`--duration` is the fight length in seconds (default 60, up to 600),
`--iterations` the number of random fights (default 500, up to 10,000) and
`--seed` makes them repeatable (default 1). `--average` runs one fight that deals
the expected shot every time. `--retarget-delay` is the pause before the next
target (default 1 s). The primary weapon is active unless `--context` names
another. The result reports `dps`, `weaponDps`, `skillDps`, `kills` and
`timeToKill` (each with mean, standard deviation, min, 5th, 50th and 95th
percentile and max), mean `shotsFired` and `reloads`, the mean damage in each
second (`timeline`), and the planner's own expected values for comparison.
Since G406, talents with a simulator trigger rule are switched on by the fight
itself (a crit kill for Killer, critical hits for Obliterate stacks), overriding
their assumption; `triggers` lists each driven key with its mean uptime or stacks
and how often it fired, and `states` counts the distinct states resolved.
Since G408, damaging skills with a rule deploy on their cooldown, dealing impact
damage and their rate for a window; `skills` lists each slot's skill with its
uses and DPS.
Since G409, `--swap empty` swaps to the other weapon instead of reloading (and back
when it empties) and `--swap kill` swaps after each kill, with `--swap-time` the
base seconds per swap (default 1); `swaps` reports the mean swaps per fight. Skill
headlines, and so Build DPS, average each damaging skill over its cooldown cycle.
Since G410, `--incoming` lets the enemies fire back: `incoming` reports the time to
the first armor break and to going down, armor breaks, the share of fights you went
down in, damage taken and armor repaired, with the planner's Time to Armor Break.
Since G412, `--distance M` sets the target distance in meters (up to 100; 0 sets
none) for Ranger, Close & Personal, Challenger and Pack Instincts. The result
repeats `distance` and sets `beyondOptimalRange` past the weapon's optimal range,
where falloff isn't modeled. A distance trigger's `value` is what its key was set
to.

## Transactional edits

`build apply` reads a nonempty ordered JSON operation array. Every operation must
be well-formed, reference authored choices and avoid introducing new build errors;
the final build must have no error diagnostics. Failure leaves the input and any
output path unchanged. Warnings about unconfigured choices do not reject a batch.

```json
[
  { "kind": "equipment", "slot": "primary_weapon", "id": "primary_weapon_acr" },
  { "kind": "attribute", "slot": "primary_weapon", "target": "core_1", "id": "assault_rifle_damage_weapon_core", "value": 0.15 },
  { "kind": "account_upgrade", "id": "watch_weapon_damage", "rank": 30 }
]
```

```text
planner --json --pack ../data/packs/division-2.json build apply --input build.json --operations edits.json --output equipped.json
planner --json --pack ../data/packs/division-2.json build resolve --input equipped.json --context context.json
planner --json --pack ../data/packs/division-2.json build share --input equipped.json
```

| Kind | Fields after `kind` |
| --- | --- |
| `equipment` | `slot`, `id`; or `slot`, `clear: true` |
| `attribute` | `slot`, `target` attribute slot, `id` attribute, optional `value`; or `slot`, `target`, `clear: true` |
| `mod` / `talent` | `slot`, `target` subslot, `id`; or `slot`, `target`, `clear: true` |
| `mod_roll` | `slot`, `target` mod slot, optional `value`; or `slot`, `target`, `clear: true` |
| `active_skill` | `slot`, `id`; or `slot`, `clear: true` |
| `skill_upgrade` | `slot`, `rank`; or `slot`, `clear: true` |
| `skill_mod` | `slot`, `target` skill-mod slot, `id` mod, optional `value`; or `slot`, `target`, `clear: true` |
| `specialization` | `id`; or `clear: true` |
| `specialization_node` | `id`, `enabled` boolean |
| `scenario` / `skill_node` | authored reference-pack `id`, `enabled` boolean |
| `account_upgrade` | `id`, `rank` (zero clears) |
| `equipment_upgrade` | `slot`, current item `id`, `rank` (zero clears) |
| `prototype` | `slot`; optional augment `id` and level `value` (1–10, default 10); `clear` returns the item to normal quality. Rolls move into the new range (G392) |
| `allocation` | primary stat `id`, `value` |
| `level` | integral `value` |
| `name` / `notes` | `text` |
| `party_member` | `target` name with share code `text`, or by hand with optional giver armor `value`; on an existing member, `value` sets a hand-entered member's armor and neither re-reads it; or `target`, `clear: true` (G395) |
| `party_buff` | `target` member, ally effect `id`, `enabled` boolean; a hand-entered member gains or loses the effect |
| `context` | `id` and `value` (numeric input) or `text` (condition); or `id`, `clear: true` |

**Maximum rolls and change lists (G373).** Leaving `value` out of an
`attribute`, `mod_roll` or `skill_mod` operation uses the highest legal roll, the
same maximum the desktop fills for fixed choices. `build apply` results and
session `apply` replies include `changes`: one entry per operation with its
1-based `operation` number, `kind`, and the paths it `set` or `cleared`, side
effects included. Paths name the build field, slot and target, such as
`equipment.chest`, `talentSelections.chest.talent` or
`attributeRolls.chest.core_1`; assumptions appear as `context.<id>`. An
operation that changes nothing, such as re-rolling a value the item already
filled, reports empty lists.

Equipment changes share the desktop's fixed-choice defaults: single-choice
attributes/mod rolls start at authored maxima, while configurable slots remain
explicit choices. Item and skill changes clear stale instance-bound state.
Removing a specialization prerequisite also removes its selected descendants.
Passive-node removal likewise prunes selected descendants.
Irrelevant fields and unsupported operation kinds are errors; the CLI does not
guess a selection or roll bound from an item name.

## Verification and remaining work

Subprocess tests build and drive the real CGO-free executable from an unrelated
working directory. Shared-operation tests cover rollback, defaults, legal rolls,
specialization cascades and both skill slots. Native state/interaction tests
continue to verify the desktop defaults through its existing API.

Build comparisons, coordinated library writes and live desktop session control
are implemented milestones in [the CLI plan](CLI_PLAN.md). Calculation coverage
and evidence limits remain those of the selected pack.
