# In-game checklist for testers

The planner's numbers come from sources that sometimes disagree.
One reading on the item card or stats screen usually
settles it.

## How to report a reading

For each check, send:

- the game version (title screen or patch notes);
- the item's name, and a screenshot of its card or the stats screen;
- every other bonus that could change the value: gear attributes, mods, talents,
  brand and set bonuses, SHD Watch and specialization. Removing them first, where
  the check says so, makes the reading much easier to use;
- for timing checks, a short video if you can. Frame-by-frame timing beats a
  stopwatch.

## Weapon talents and stats

Unless a check says otherwise, take off weapon mods and gear that add the same
stat, so the talent is the only bonus.

| Item | What to read | Research says | Community says |
| --- | --- | --- | --- |
| **Rock n' Roll** (Perfect Extra) | Magazine size on the weapon card. Base magazine is 20. | **30** (+50%) | **26** (+30%) |
| **New Reliable** (Perfect Optimized) | The weapon handling stat itself, not reload time or stability, which handling feeds through different formulas. | **+40%** | **+20%** |
| **The Send-Off** (Perfect Distance) | Optimal range on the weapon card. Base is 16 m. | **32 m** (+100%) | **20 m** (+25%) |
| **Safety Distance** | Time a reload with rounds still in the magazine (a tactical reload), with no reload speed bonuses. | **2.79 s** (and 2.09 s from empty) | no tactical value |
| **Sweet Dreams** | Optimal range with a total optimal-range bonus of +30% or +40%. Its fixed Stunner mod gives +25%, so add +5% or +15% more. With only Stunner, both sources show 13 m, so that card doesn't settle it. | **13.0 m** at +30%, **14.0 m** at +40% | **13.5 m** at +30%, **14.6 m** at +40% |
| **Busy Little Bee** | Optimal range with Bee's Focus. | **18 m** expected | – |

## Situational talents

These need the talent's condition to trigger, so a short video helps most.

- **Bober (Perfect Entrench):** with armor below 30%, land a headshot from cover.
  Read how much armor comes back, as a share of your maximum armor. Research says
  **30%**, community says **20%**. The 30% armor threshold is a separate value.
- **Achilles or Instigator (Perfect Soft Spot):** destroy a weak point and note the
  weapon damage bonus and how long it lasts. Research says **32% for 15**,
  community says **24% for 15 s**. Please also confirm that the 15 is seconds.
  Note your other weapon damage bonuses, since they add together.
- **Vigil (Perfect Versatile):** with an assault rifle or LMG, swap between two
  different weapons and shoot enemies 15–25 m away. Read the damage amplification.
  Research says **15%**, community says **20%**, for 10 s.
- **Sleight (Perfect Protected Reload):** needs a teammate. While they reload, read
  the bonus armor you give them, per blue (defensive) core attribute you have.
  Research says **5% per core**, community says **3% per core, up to 18%**. The
  research shows no cap, so six cores would give 30%.

## Prototype gear

The planner models Prototype gear from the official notes and community data; a
few values still need a reading.

- **Upgraded item maxima:** upgrading an item to Prototype rolls one attribute at
  the new maximum. If that attribute is one of these, read its value. Each
  settles how the game rounds 1.5 × the old maximum:

  | Attribute | Planner shows | Exactly 1.5 × |
  | --- | --- | --- |
  | Critical Hit Chance (weapon minor) | 14.5% | 14.25% |
  | Health Damage (weapon minor) | 14.5% | 15% |
  | Armor Regeneration (gear minor) | 7,388 | 7,387.5 |
  | Health (gear minor) | 28,403 | 28,402.5 |
  | Skill Haste (gear minor) | 15% | 18% |
  | Reload Speed on Sharpshooter's 93R | 14% | 13.95% |
  | Health Damage on Firestarter | 11.3% | 11.25% |

  Research lists Skill Haste at 18%, so its reading matters most.
- **Skill Tier:** with Prototype Skill Tier cores equipped, read the total Skill
  Tier. The planner counts 1.5 per item and rounds the total down: one item
  shows 1, two show 3, three show 4 and four show 6.
- **Augment values:** read a Quantum and an Anomaly augment's percentage at a
  known level. The planner uses Quantum 1.6% at level 1 rising 0.3% per level,
  and Anomaly 4% rising 0.5% per level. Older patch notes give different
  figures.

## Skill timing

These feed the planner's skill DPS. Note the skill tier, mods and Expertise grade,
and what you were shooting at.

- **Striker Drone:** shots per burst, how long it fires, and the pause between
  bursts, recorded separately. It may or may not match the Assault Turret.
- **Sniper Turret:** the time between shots you trigger, and its recharge time.
- **Incinerator Turret:** the interval between damage ticks. Keep direct flame
  hits, burn refreshes and burn ticks apart. The community suggests **0.5 s**, but
  nobody has measured it yet.

## Newly modeled talents

- **Perfect Empathic Resolve:** after repairing yourself, read your weapon damage
  bonus at a known Skill Tier. The planner gives 2.3% plus 2.333% per tier
  (16.3% at tier 6).
- **Alternating Current:** at 10 stacks, check whether both skills or only the
  stacked one deal more damage. The planner applies the stacks to both.
- **Sledgehammer:** with a grenade mark on a target, check whether the extra armor
  damage also applies to your other weapon. The planner applies it only while the
  Sledgehammer weapon is out.

- **Boomerang:** fire a full magazine at a target and count the rounds fired before
  reloading, with a known critical hit chance. The planner expects the magazine to
  last 1 / (1 - 0.5 x crit chance x hit rate) times as long.
- **First Blood:** with an 8x scope, check whether the first shot after any reload
  counts as a headshot, or only after reloading from empty as the text says.

- **Headhunter:** after a headshot kill, read the damage of your next hit and the
  killing blow's. The planner adds 125% of the killing blow, capped at 800% of
  weapon damage (1,250% above 150% headshot damage).

- **Busy Little Bee:** after swapping away with 10 stacks, read your other weapon's
  damage per bullet. The planner triples it (+20% amplified per stack).

- **Abridged:** with two non-named rifles of one type and a stat talent such as
  Near Sighted on the secondary, check the primary's stability on its card. The
  planner adds the secondary talent's effect while the primary is out.

## Party buffs

The Party tab applies teammates' talent buffs. Two readings would settle how they
combine:

- **The same buff twice:** with two teammates both running Overwatch (or both
  Vanguard), read your weapon damage (or bonus armor) while both buffs are up.
  The planner applies the same buff once, the stronger copy.
- **Vanguard's share:** with a Vanguard teammate whose armor you know, deploy their
  shield and read the bonus armor you receive. The planner gives 45% of their
  total armor.

## Bigger tests

- **Damage falloff:** for a few weapons, shoot the same target at measured
  distances from optimal range out past long range, and record the body-shot damage
  number at each distance. This settles how damage drops between optimal and long
  range, and where it bottoms out. The planner doesn't model falloff until then.
- **Enemy health:** pick one enemy type at a stated difficulty and count the body
  shots a known weapon needs to break its armor and then kill it. This confirms the
  absolute health and armor the planner uses for time to kill.

Planner developers: the reviewed status of each check is in
reviewed corrections and
data requests R7 and R15.
