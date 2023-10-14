# Tyruswoo Battle Mechanics for RPG Maker MZ

Alter your game's battle calculations!

Make some skills more likely to hit than others!

Make some skills strike critical hits more often or more powerfully
than others!

## Compatibility

This plugin is designed to work well alone or with most other plugins in
any order. However, it may conflict with other plugins that heavily modify
the way damage, hit rate, or other battle calculations are made.

## Overview

This plugin opens up many ways to customize battle calculations. You can:
- Set a global minimum and maximum damage per hit.
- Alter the formulas that determine how hit, evasion, critical hit,
  critical damage, and luck effect rates are calculated.
- Use `<hit mod>`, `<crit mod>`, and `<crit boost>` notetags to customize the
  hit rate, critical hit rate, or critical damage of specific skills.
- Use Standard Damage Function and High Resist Damage Function to process
  all skills and items in a unified way, based on each skill's `<power stats>`
  and `<resist stats>` notetags.
- Log specific battle calculations to the console to see how they're
  working.

## Formula Writing Helps

This plugin exposes several formulas as parameters to let you customize
battle calculations. Any formula you leave at its default value will work
as RPG Maker MZ does, except that this plugin's notetags (if present) will
alter the result appropriately.

All formulas (except the Damage Functions) can use the following variables:

| Variable         | Description                                         |
|------------------|-----------------------------------------------------|
| `action`         | The current action (attack, skill use, or item use) |
| `subject`        | The battler performing the action                   |
| `target`         | The battler who may be affected by this action      |
| `a`              | An alias for subject                                |
| `b`              | An alias for target                                 |
| `v`              | The list of game variables.                         |

Actions and battlers are objects; to include them in the math you'll need
to use their properties or methods.

All battlers have the following types of properties:
- Parameters: `atk`, `def`, `mat`, `mdf`, `agi`, `luk`, `hp`, `mp`, `tp`, `mhp`, `mmp`
- Ex-Parameters: `hit`, `eva`, `cri`, `cev`, `mev`, `mrf`, `cnt`, `hrg`, `mrg`, `trg`
- Sp-Parameters: `tgr`, `grd`, `rec`, `pha`, `mcr`, `tcr`, `pdr`, `mdr`, `fdr`, `exr`
- Notetags: `critBoost`

This plugin grants the following properties to actions:
- `hitMod`: The `<hit mod>` value of this action's skill or item.
  Useful for making some skills more likely to hit than others.
- `critMod`: The `<crit mod>` value of this action's skill or item.
  This can make some skills more or less likely to land a critical hit.
- `critBoost`: The `<crit boost>` value of this action's skill or item.
  This lets some skills do higher or lower critical damage compared to
  their usual damage.

Below are examples of syntax for accessing properties and game variables
in formulas:

| Example                  | Note                                          |
|--------------------------|-----------------------------------------------|
| `subject.hit` or `a.hit` | The attacker's Hit Rate                       |
| `target.agi` or `b.agi`  | The target's Agility                          |
| `action.critMod`         | The active skill's `<crit mod>` notetag value |
| `v[15]`                  | The value stored in Game Variable #15         |

## Console Logging

When you playtest your RPG Maker MZ game, you can use the F12 key at any
time to open the debugging console. This plugin lets you choose which
types of messages about battle calculations show up in the debugging
console. For playtesting convenience, the parameter Console Log Categories
is at the top of the list of plugin parameters.

To pick a type of battle calculations to show on the console log,
double click on the Console Log Categories list.
You'll see another window pop up, where you can click the dropdown to
select a category. Then click OK. Repeat this step to add more categories
if you wish. To delete a category, select it and press the Delete key,
or right click it and pick Delete from the context menu.

The "all" category shows all Tyruswoo_BattleMechanics log messages of all
categories.

The "none" category turns off all Tyruswoo_BattleMechanics log messages
(except for errors and warnings), even if other categories are in the list.

## Minimum and Maximum Damage

The default Minimum Damage parameter is 0. If you'd like to give even the
weakest attack or heal at least a little effect, raise the Minimum Damage.

One exception to Minimum Damage: if the target is completely immune to
the element of the attack, the damage will be zero.

RPG Maker MZ has no upper limit to each hit's damage. This plugin starts off
with the Maximum Damage plugin parameter set to 9999, but you can change it
to whatever suits your game. This limit will apply to damage and healing
of HP and MP.

## Standard and High Resist Damage Functions

The purpose of the Damage Functions is to modify most skills in a uniform,
mass-modifiable way based on their `<power stats>` and `<resist stats>`.
If you'd rather use RMMZ's skill damage formulas as usual and tweak them
individually, you can freely ignore this feature for some or all of your
skills. If you know how strong you want damage skills to be relative to
each other, but you're not sure yet how you plan to balance them as a whole,
the Damage Functions, `<power stats>`, and `<resist stats>` feature may help you.

### Understanding Damage Calculations

This plugin applies the Standard or High Resist Damage Function directly
after the active skill or item's damage function is calculated,
but before other factors alter the damage. For context, below are all steps
RPG Maker MZ takes to calculate damage, with this plugin's additions marked:

 1. Evaluate the active skill or item's damage formula. This is `itemDamage`.
 2. THIS PLUGIN: Apply the Standard or High Resist Damage Function to the
    itemDamage.
 3. Multiply by the target's Element Rate for this action's element(s).
 4. Multiply by the target's `pdr` or `mdr`, depending on attack type.
 5. If it's healing, multiply by the targets `rec` rate.
 6. If it's a critical hit, apply the Critical Damage Function.
 7. Apply this skill or item's variance.
 8. If the target is guarding, divide damage by `(2 * target.grd)`.
 9. Round damage to the nearest whole number.
10. THIS PLUGIN: Make damage at least Minimum Damage,
    but no more than Maximum Damage.

The Damage Functions will apply to all actions using skills or items
that have notetags written in their Note field for `<power stats>`,
`<resist stats>`, or both.

Because the Damage Functions already take care of scaling damage based
on stats, a database entry for a skill or item with `<power stats>` and
`<resist stats>` can use a simple constant for its damage formula
(e.g. `5` for a weak skill, or `50` for a stronger skill), and the
globally defined Damage Functions will take care of the rest.

### `<power stats>` notetag
The higher the skill user's power stat(s), the more damage or healing this
skill will do. If a skill has multiple power stats listed, their arithmetic
average will be used as `powerStat` in the Damage Functions.

If the active skill or item has no `<power stats>` notetag, 
the Damage Function will use a `powerStat` of 0.

### `<resist stats>` notetag
The higher the target's resist stat(s), the less damage it will take from
a damage skill.

If the active skill or item has no `<resist stats>` notetag,
the Damage Function will use 0 as the `resistStat`.

Below are examples of valid power stat and resist stat notetags:

| Syntax Example              | Comment                                      |
|-----------------------------|----------------------------------------------|
| `<power stats: mat, luk>`   | This skill's damage depends on the average of the subject's Magic Attack and Luck. |
| `<resist stats: mdf>`       | The target's MDF reduces this skill's damage |
| `<power stat: atk>`         | This skill's damage depends on ATK alone.    |
| `<powerStat: ATK, AGI>`     | The notetag is case insensitive. The space and ending 's' are optional. |
| `<resistStats:def,agi,luk>` | Use many power or resist stats if you like   |

The average of the subject's `<power stats>` for this skill or item are used
as `powerStat`, and the average of the target's `<resist stats>` for this skill
or item are used as `resistStat`.

If `powerStat` >= `resistStat`, the Standard Damage Function is used.
If `powerStat` < `resistStat`, the High Resist Damage Function is used instead.

### Standard Damage Function

The default formula for the Standard Damage Function is as follows:
```
    itemDamage + powerStat - resistStat
```
Since the Standard Damage Function applies when `powerStat` >= `resistStat`,
the skill or item's damage will be increased by however many points
greater the `powerStat` is than the `resistStat`.

### High Resist Damage Function

The default formula for the High Resist Damage Function follows:
```
    itemDamage - Math.pow(resistStat - powerStat, 0.5)
```
This means that however much greater `resistStat` is than `powerStat`,
take the square root of that and subtract it from `itemDamage`. This makes
damage drop to the Minimum Damage more gradually than the Standard Damage
Function would have done.

### Example of Damage Functions at work

Let's say that Stab is a skill whose damage formula is simply `10`.
No calculations in the damage formula, just `10`
Stab also has the following notetags written in its note:
```
    <power stats: atk, agi>
    <resist stats: def, agi>
```
Now suppose Alice uses Stab on Bob and it hits. How much damage will it do
once we put it through its Damage Function, assuming default formulas?
1. We've already run Stab's damage formula at this point. Since the formula
   is `10`, itemDamage is 10.
2. Stab's Power Stats are ATK and AGI, so if Alice's ATK is 8 and
   her AGI is 6, then this action's `powerStat` is (8 + 6) / 2 = 14/2 = 7.
3. Stab's Resist Stats are DEF and AGI, so if Bob's DEF is 7 and
   his AGI is 3,then this action's `resistStat` is (7 + 3) / 2 = 10/2 = 5.
4. In this case, `powerStat` >= `resistStat`, so we use the Standard Damage
   Function.
5. The Standard Damage Function is `itemDamage + powerStat - resistStat`,
   so if we plug in our values, that's 10 + 7 - 5 = 12
6. The damage coming out of the Damage Function phase is 12, but this
   may change further if Alice strikes a critical hit, or if Bob is
   guarding, or if there's some other damage-altering circumstance found
   later on in the damage calculations.

### Rewriting the Damage Functions

When you alter a Damage Function, your changes will affect the damage
calculations of all skills and items that have notetags for `<power stats>`,
`<resist stats>`, or both.

The Damage Functions can use all variables available to other formulas,
plus the following:
- `itemDamage`: The result of evaluating the skill or item's damage formula.
- `powerStat`: The average of the item's `<power stats>` for this action's
  subject.
- `resistStat`: The average of the item's `<resist stats>` for the current
  target.

When you formulate Damage Functions, consider what you want to happen
in a variety of scenarios, such as the following:
* `powerStat` is much greater than `resistStat`
* `powerStat` is a little greater than `resistStat`
* `powerStat` and `resistStat` are equally matched
* `powerStat` is a litle less than `resistStat`
* `powerStat` is much less than `resistStat`
* `powerStat` is zero
* `resistStat` is zero
* both `powerStat` and `resistStat` are zero

A graphing calculator can help you visualize possible outcomes:
https://www.desmos.com/calculator

Refer to JavaScript's [Math object reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) for functions you can
call inside your formulas.

## Hit and Evasion Rates

RPG Maker MZ runs a hit calculation first, and then an evasion calculation.

### Hit Rate Formulas

The hit rate calculation interprets the skill or item's success rate as a
percent, and multiplies it by the subject's HIT trait (or by 1 in the case
of magical attacks).

Tyruswoo_BattleMechanics exposes Hit Rate Formulas to let you replace this
HIT trait (or 1 in the case of magical attacks) with whatever works best
for your game.

In case you don't want to change hit calculations, the default Hit Rate
Formulas keep it simple.

The default Physical Hit Rate Formula:
```
    subject.hit + action.hitMod
```
This lets you give some skills a `<hit mod>` notetag to give them a
higher (or lower) hit rate than other skills.

The default Magical Hit Rate Formula is simply 1, meaning that magic
doesn't have a chance to miss aside from Success-Rate-based failures.

If you want chance to hit to change based on a stat like Agility or Luck,
you can change the Hit Rate Formulas to do that! Bear in mind that, given
a Success Rate of 100%, 0.0 or less always misses, 0.5 misses half the time,
and 1.0 or greater never misses.

### `<hit mod>` notetag

Put a `<hit mod>` notetag in a skill or item's Note field to alter its
chance to hit. Hit mod is additive, so if a character with a HIT ex-param of
95% uses a skill with the notetag `<hit mod: 4>`, it has a 99% chance to hit.

Below are examples of valid `<hit mod>` notetags:

- `<hit mod: 5>` - This skill's hit rate is the user's HIT + 5%.
- `<hot mod: +2>` - This skill's hit rate is the subject's HIT + 2%.
                  The plus sign is optional.
- `<hit mod: 20%>` - This skill's hit rate is 20% more than the subject's HIT.
                   The percent sign is optional.
- `<hit mod: -10>` - This skill's hit rate is the subject's HIT trait - 10%.
- `<hit mod: 0>` - This skill has a typical chance to hit based on its user's
  HIT trait.

Any skill that has no `<hit mod>` notetag is assumed to have a `hitMod` of 0.

A hit mod of -10 differs from a 90% success rate because hit mods are
additive rather than multiplicative.
A character with a high HIT trait can overcome a negative `<hit mod>`,
while a character with a low HIT trait is even more adversely affected by
it.

Because Tyruswoo Battle Mechanics caps Hit Rate Formula results at 1.0,
a skill with a 90% success rate will always have at least 10%
chance of failing, regardless of how skilled its user is.

### Evasion Formulas

If the subject doesn't miss when attacking, the target might still evade.
This plugin lets you customize the formulas for the target's probability of
evading a physical or magical attack.

This plugin's default Physical Evasion Formula is the same as RMMZ's:
```
    target.eva
```
And its default Magical Evasion Formula is also the same as RMMZ's:
```
    target.mev
```
A battler's Ex-Params EVA or MEV are the sum of all EVA or MEV traits
applied to that battler, directly through its Actor or Enemy database entry,
and through its class, weapons, armor, and states currently in effect.

If you want physical or magical evasion to get a bonus from Agility or Luck,
or be calculated any way of your choice, you can alter its formula to do
that!

## Critical Hit Rate

This plugin exposes a Critical Hit Rate Formula that you can customize
freely. The following is its default value:
```
    (subject.cri + (action.critMod/100)) * (1 - target.cev)
```
Like RMMZ's hard-coded crit rate formula, it starts with the subject's
CRI Ex-Param summed from the subject's own database entry and that of its
class, equipment, and states. Then it scales it down based on the target's
CEV (Critical Evasion) Ex-Param, if any.

What's different is that this plugin's default crit rate formula adds the
active skill or item's `<crit mod>` notetag value (interpreted as a percent)
to the subject's CRI when determining the crit rate.

### `<crit mod>` notetag

A `<crit mod>` notetag can be placed in the Note field of a Skill or an Item
to make it more (or less) likely to score a critical hit when used.
Crit mod is additive, so a battler with a CRI of 4% using a skill with the
notetag `<crit mod: 5>` will have a 9% chance of scoring a critical hit with
that skill.

Below are examples of valid `<crit mod>` notetags:

- `<crit mod: 3>` - This skill's crit rate is its users CRI + 3%.
- `<crit mod: 10>` - This skill's crit rate the subject's CRI + 10%.
- `<crit mod: -2>` - This skill's crit rate is the subject's CRI - 2%.
- `<crit mod: 0>` - This skill has a typical chance to hit based on its user's
  CRI trait.

Any skill that has no `<crit mod>` notetag is assumed to have a `critMod` of 0.

## Critical Damage

Without this plugin, a critical hit always deals the action's regular damage
multiplied by 3. This plugin exposes the Critical Damage Formula as a
plugin parameter so that you can make a critical hit alter the damage in
any way you want.

The default Critical Damage Formula follows:
```
    damage * (3 + (subject.critBoost + action.critBoost) / 100)
```
This formula makes a multiplier for the damage, starting with 3, then
adding all the subject's and skill or item's `<crit boost>` notetags,
interpreting their sum as a percent. If no `<crit boost>` notetags apply,
the damage will simply be modified by 3, like RMMZ does by default.

Each variable in the formula, explained:
- `damage`: This action's damage before it is made into critical damage.
- `subject.critBoost`: The sum of all values of `<crit boost>` notetags applied
  to the acting battler. This includes what's on its own Actor or Enemy
  entry, as well as its class, equipment, and applied states. If no
  `<crit boost>` notetags apply to the subject, subject.critBoost is 0.
- `action.critBoost`: The value of the `<crit boost>` notetag, if any, on the
  Skill or Item being used in the current action, or 0 if there is none.

### `<crit boost>` notetags

A crit boost notetag can be placed in the Note field of an Actor, Class,
Skill, Item, Weapon, Armor, Enemy, or State. Below are examples of
correctly formed crit boost notetags, and their meanings under the default
Critical Damage Formula:

- `<crit boost: 5>` - Add 5% to the critical damage multiplier.
- `<crit boost: +10>` - Add 10% to the multiplier. The plus sign is optional.
- `<CritBoost: +2>` - Add 2% to the multiplier. Notice that it's case insensitive and the space in crit boost is optional.
- `<crit boost: -5>` - Subtract 5% from the critical damage multiplier.
- `<crit boost: -25%>` - Subtract 25% from the critical damage multiplier. The percent sign is optional.                       |

## Luck Effect Rate

The Luck Effect Rate affects a battler's chance of inflicting a state or
debuff on an opposing battler.

The plugin parameter Luck Effect Rate Formula is pre-loaded with RPG Maker
MZ's formula: 
```
     1.0 + (subject.luk - target.luk) * 0.001
```
This formula gives a luckier subject a modest bonus to state effect success,
and a luckier target a modest bonus to state effect resistance.
You may change this formula to base state and debuff success on anything
you wish, with any math you want to use.

If you want luck to affect other types of battle calculations aside from
states and debuffs, you can include a.luk, b.luk, or action.lukEffectRate 
in other formulas using this plugin.

### For more help using this plugin, contact us at [Tyruswoo.com](https://www.tyruswoo.com).

## Version History

**v0.1.0**  10/13/2023
- Tyruswoo Battle Mechanics development version released!
- Minimum and maximum damage limits
- Customizable formulas for hit and evasion rates, critical hit rate,
  critical damage, and luck effect rate
- `<hit mod>`, `<crit mod>`, and `<crit boost>` notetags
- Global damage functions that work with `<power stats>` and
  `<resist stats>` notetags
- Console logging by battle calculation category
