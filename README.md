# ORB ARENA

A physics ball brawler that runs in the browser. Pick an orb, kick yourself around a
tight box, bounce off the walls, and fight an escalating series of opponents. Every orb
has its own ability — and **you never press a button to use it**.

Single self-contained `index.html`. No build step, no dependencies, no assets.

```
open index.html
```

That's the whole install.

---

## How it plays

You are one orb in a closed box. You have no engine — the only way you move is by
**kicking**, and then you keep going until a wall stops you. The walls are perfectly
elastic, so a bounce never costs you speed. Momentum is the whole game.

| Input | Action |
| --- | --- |
| Mouse | Aim |
| Click / `Space` | Kick (hold to keep kicking on cooldown) |
| `W` `A` `S` `D` | Kick in that direction |
| `P` / `Esc` | Pause |
| `M` | Mute |

There is no ability key. See below.

---

## Abilities fire themselves

Every archetype runs an automatic trigger, always gated by its own cooldown:

- **`near`** triggers fire once a fight is genuinely in range.
- **`far`** triggers fire when it is safe to commit, or when you are badly hurt.
- **`wall`** triggers fire on the bounce itself, and anchor the effect to the exact
  piece of wall you just hit.

That last one is the heart of the design. Your movement input and your ability input are
the *same input*: where you choose to bounce is where you choose to place things. The
arena becomes a weapon.

Because the AI orbs run the identical rule set, the player and the opponents sit on
exactly equal footing.

---

## The ten orbs

| Orb | Ability | Trigger | Identity |
| --- | --- | --- | --- |
| **BRUISER** | SLAM | near | Heavy. Slowest to kick, but its body is a weapon. |
| **SPLITTER** | DIVIDE | far | Divides into smaller copies — and again when one dies. |
| **LASER** | FENCE | **wall** | Two wall nodes link into a live beam. 112 dmg/sec to anything crossing. |
| **TRAPPER** | SPIKE | **wall** | Every wall you hit gets a spike. Anything near it detonates. |
| **TRANSMUTER** | TRANSFORM | near | 2.5× contact damage, +30% speed, +18% size, 30% damage resistance. |
| **PHANTOM** | PHASE DASH | near | Lightest orb. Takes a kick harder and flies further than anything. |
| **FROST** | FREEZE | near | Chills everything nearby — a chilled orb keeps 45% of its top speed. |
| **MAGNET** | GRAVITY WELL | near | Tears open a point that drags enemies in and crushes them while they are held. |
| **TESLA** | CHAIN ARC | near | Leaps from enemy to enemy, up to four, losing power with every jump. |
| **RICOCHET** | MOMENTUM | **wall** | Every bounce banks a charge: more damage, more speed, less damage taken. |

### The three wall archetypes

**LASER** plants a node into the wall on every bounce. Nodes pair off in the order they
were planted, and each pair strings a live beam between them. Two touches, one fence.
Up to 7 nodes — three fences and a spare. String them across the box and let the arena
do the killing.

**TRAPPER** drives a spike into the wall on every bounce, up to 7 at a time. Bounce
around the rim and the whole edge of the arena becomes a minefield.

**RICOCHET** banks a charge on every bounce instead of leaving something behind: +14%
contact damage, +7% top speed and −5% damage taken per stack, up to eight. Let the
charge lapse for four seconds and you lose the lot. It is the most fragile orb in the
game when it is caught standing still and one of the toughest once it is rolling — so
it rewards a player who never stops moving.

These are area-control characters: they are weaker in a straight brawl and much stronger
if you use the box instead of fighting in the middle of it.

### The four "second wave" orbs

The three control orbs and RICOCHET were added as a second wave, tuned against the
original six so that no archetype is strictly better than another. **FROST** and
**MAGNET** are force-multipliers — they do not kill much themselves, but they make
everything else easy to hit. **TESLA** is a crowd weapon that wants the enemy bunched
up, which makes it a natural partner for a gravity well. **RICOCHET** is a pure
momentum character with the highest ceiling and the lowest floor.

---

## The secret orb

There is an eleventh orb that does not appear in the roster. It is unlocked by entering
a password on the admin screen — the lock button in the top-right of the main menu.

It is deliberately, unfairly strong: roughly six times the health of a normal orb and an
ability that chills, hauls in and damages every enemy on screen at once. It is a toy,
not a balance point.

The password is not published here — it is a secret, and this is a public repository.

---

## Loadout

Coins earned in runs buy passive modules. Three slots.

| Module | Effect |
| --- | --- |
| VITALITY | +22% max HP |
| KICK POWER | +18% kick power and top speed |
| QUICK BOOTS | −28% kick cooldown |
| SPIKES | +30% contact damage |
| OVERCLOCK | −22% ability cooldown |
| REGEN | Regenerate 2 HP/sec |
| PLATING | Take 18% less damage |
| VAMPIRE | Heal 15% of damage dealt |
| AEGIS | Start each round with a one-hit shield |
| WAR CORE | +45% ability damage |
| SWARM | Start each round with 2 free minions |

Progress, coins and inventory persist in `localStorage`.

---

## Technical notes

- **Fixed-timestep simulation** at 1/120 s with an accumulator, decoupled from rendering.
- **Fixed world box** (820×520) letterboxed into the viewport with fully proportional
  padding, so the arena scales and stays centred at any window size.
- **Kick as impulse**, not thrust — a kick is a velocity change, so light orbs fly
  further than heavy ones. A speed floor guarantees the box never settles into a
  stalemate.
- **Status effects** are first-class: chill (a speed debuff), gravity wells (pull plus
  crush damage on a tick), chain arcs (segmented lightning that hops between targets)
  and momentum stacks (a stacking self-buff).
- **Contact damage is symmetric** — the same figure is what an orb deals *and* what it
  takes. That makes any "hit harder" buff a self-debuff, so stacking buffs like
  RICOCHET's momentum carry an explicit damage-resistance term to keep them one-way.
- **Hard cap of 30 simultaneous orbs**, enforced in three separate places.
- **All audio is synthesised at runtime** from `OscillatorNode`, `AudioBufferSourceNode`
  and `BiquadFilterNode`. Zero audio files. A master compressor, per-sound cooldowns and
  a hard voice cap keep a busy screen from turning to mush, and sounds are panned by
  world position.
- Canvas 2D, vanilla JS, no framework.
