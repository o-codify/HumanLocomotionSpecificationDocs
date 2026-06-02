---
id: weapon-interaction-constraint-priority
title: Weapon Interaction Constraint Priority
status: draft
version: 26.602.1529
tags: [ weapon, interaction, constraints, priority, ik, planning ]
---

# Weapon Interaction Constraint Priority

## Purpose

This document defines how weapon interaction constraints are prioritized when they conflict.

Weapon interaction often has several valid goals at the same time:

```text
keep the weapon supported
keep main grip contact
keep support grip contact
align sight/muzzle visually
reach a magazine or mechanism point
move an object along an authored axis
avoid impossible wrist/elbow poses
restore the previous hold pose
avoid snapping during network correction
```

Not all of these can always be satisfied simultaneously. This document defines which constraints are hard, which are soft, and which may be temporarily broken.

---

## Scope Boundary

This document only prioritizes constraints owned by weapon interaction:

```text
hand contacts
weapon support contacts
interaction points
object visual attachment
moving part following
pose restoration
animation target validity
```

It does not prioritize locomotion speed, body yaw, camera behavior, projectile behavior, damage, inventory, or AI decisions.

---

## Constraint Classes

### Never-Break Constraints

These must not be violated during a valid interaction.

```text
no unsupported weapon if the weapon requires support
no impossible hand role state
no duplicate visual object ownership
no gameplay/visual state contradiction after authoritative correction
no use of missing required socket/interaction point
no semantic axis inversion
```

If a never-break constraint cannot be satisfied, the action must fail, recover, or fall back.

### Hard Constraints

These are required for the current interaction step to be valid.

```text
main hand remains attached to main grip unless the current action explicitly releases it
required manipulation hand reaches the required interaction point
object insert/extract follows authored local axis
moving part follows its authored axis/path
required commit point only fires at the authored phase
```

A hard constraint may be temporarily replaced only by an explicit recovery plan.

### Temporary-Break Constraints

These may be broken for a short, authored interval.

```text
support hand leaves support grip during reload
shoulder contact becomes partial during manipulation
muzzle visual alignment degrades during object insertion
ADS pose degrades during reload overlay
finger grip relaxes during hand transition to object
```

Temporary breaks must have:

```text
start phase
maximum duration or step range
recovery target
fallback if recovery fails
```

### Soft Constraints

These improve quality but should not block the interaction.

```text
perfect elbow pole direction
exact wrist comfort
exact finger contact
perfect muzzle visual alignment during non-fire manipulation
minor weapon pose offset preference
cosmetic object arc preference
```

Soft constraints blend out or lose priority when they conflict with hard constraints.

---

## Default Priority Order

Recommended priority order:

```text
1. validity of authored data
2. no unsupported weapon
3. no duplicate/conflicting visual object ownership
4. main grip or required primary support contact
5. current step hard target
6. authored axis/path semantics
7. required commit timing
8. hand reachability and joint limits
9. restoration of previous hold pose
10. sight/muzzle visual alignment
11. elbow/wrist comfort
12. fingers/cosmetic contact
```

Projects may tune this, but the rule must be explicit.

---

## Conflict Examples

### ADS vs Reload

Conflict:

```text
ADS wants strict sight alignment.
Reload wants support hand to leave the weapon.
```

Resolution:

```text
reload hard step wins
ADS becomes degraded overlay
restore ADS after reload if still valid
fallback to HipFire or LowReady if ADS cannot be restored
```

### Support Grip vs Magazine Reach

Conflict:

```text
support hand stabilizes weapon
same hand must reach magazine
```

Resolution:

```text
if remaining contacts satisfy stability, release support hand temporarily
if not stable, use alternate hand, pose adjustment, or reject/recover
```

### Wrist Comfort vs Insert Axis

Conflict:

```text
wrist comfort prefers a different hand rotation
magazine insert axis requires exact local-axis alignment
```

Resolution:

```text
insert axis wins during insertion
wrist comfort becomes soft and blends back during settle
```

### Mirroring vs Axis Semantics

Conflict:

```text
mirrored presentation changes visual side
local weapon axis must remain semantically correct
```

Resolution:

```text
canonical axis is transformed by the mirror stage
axis is not recomputed from arbitrary positions
```

---

## Constraint Result

Every solver should output a constraint result:

```text
ConstraintResult:
  bValid
  BlockingConstraint
  BrokenTemporaryConstraints
  SoftConstraintLoss
  RecoveryRequired
  SuggestedFallbackPose
  DebugReason
```

---

## Recovery Rules

If a hard constraint fails:

```text
try alternate hand if authored and valid
try compact weapon pose if allowed
try previous stable hold pose
try safe cancel
try visual-only recovery
fail action cleanly
```

If a never-break constraint fails:

```text
stop the interaction plan
clear invalid targets
restore last stable state if possible
emit validation/debug error
```

---

## Debug Requirements

Debug should show:

```text
active constraints
constraint class
priority value
current winner
current loser
temporary-break timer
recovery target
fallback selected
blocking reason
```

---

## Final Formula

```text
Constraint priority =
  never-break validity
  + hard interaction requirements
  + temporary authored breaks
  + soft quality goals
  + explicit recovery when conflict cannot be solved.
```
