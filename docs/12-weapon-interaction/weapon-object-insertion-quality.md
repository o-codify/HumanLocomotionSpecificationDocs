---
id: weapon-object-insertion-quality
title: Weapon Object Insertion Quality
status: draft
version: 26.602.1612
tags: [ weapon, interaction, object, insertion, reload ]
---

# Weapon Object Insertion Quality

## Purpose

This document defines quality and phases for inserting interaction objects into weapons.

Objects include:

```text
magazines
shells
rounds
clips
batteries
power cells
sci-fi cores
other authored reload/manipulation objects
```

This is visual/procedural object interaction, not inventory or ammo economy.

---

## Scope Boundary

This document owns:

```text
object alignment
insert tip alignment
axis alignment
insert travel
seat/lock visual quality
failed alignment recovery
object-to-weapon socket relation
```

It does not own:

```text
whether inventory has the item
ammo count
projectile behavior
damage
weapon balance
```

---

## Core Rule

```text
An object insertion is valid when the object's authored insert reference aligns to the weapon's authored insertion target and travels through the authored semantic path.
```

Do not decide insertion quality by final position alone.

---

## Common Insertion Phases

Recommended phases:

```text
PreAlign
Approach
AxisAlign
InsertTravel
Seat
LockCommit
PostLockSettle
Recover
Failed
```

---

## Phase Meanings

### PreAlign

The object is oriented roughly toward the target before precise insertion.

### Approach

The hand/object moves toward the insertion point.

### AxisAlign

The object insert axis aligns with the weapon insertion axis.

### InsertTravel

The object moves along the authored local insertion axis/path.

### Seat

The object reaches final seated position or near-seated alignment.

### LockCommit

The semantic interaction commit happens.

Examples:

```text
MagazineLocked
ShellInserted
BatterySeated
CoreLatched
```

### PostLockSettle

The hand/object/weapon visually settles after commit.

### Recover

The system corrects or backs out from a partial insertion.

### Failed

Insertion cannot continue.

---

## Quality Values

Recommended values:

```text
ObjectApproachQuality: 0..1
ObjectAxisAlignmentQuality: 0..1
ObjectInsertionDepthQuality: 0..1
ObjectRotationAlignmentQuality: 0..1
ObjectSeatQuality: 0..1
ObjectLockReadiness: 0..1
```

`ObjectLockReadiness` is procedural readiness for the commit point. It is not inventory confirmation.

---

## Straight Magazine Insertion

Typical phases:

```text
PreAlign → Approach → AxisAlign → InsertTravel → Seat → LockCommit → PostLockSettle
```

Rules:

```text
InsertTipSocket aligns to MagazineWellSocket
LocalInsertAxis controls travel direction
LockCommit occurs only after depth/rotation quality pass thresholds
```

---

## Rock-And-Lock Magazine

Some magazine interactions require rotation around a seated front/rear point.

Typical phases:

```text
PreAlign → FrontHookAlign → RotateIntoWell → Seat → LockCommit → PostLockSettle
```

Authored data may include:

```text
HookSocket
PivotAxis
RotationAngle
LockSocket
```

---

## Shell Loading

Shell loading may use pinch grip and shorter insertion travel.

Typical phases:

```text
Approach → AxisAlign → InsertTravel → Seat/Commit → Release → Recover
```

Rules:

```text
shell insert tip must align to port/tube
hand grip may be pinch, not full object grip
commit may happen before hand fully releases
```

---

## Battery / Core Insertion

Power cells or sci-fi cores may use custom paths.

Rules:

```text
still expose authored insert target
still expose semantic axis/path
still expose commit point
still expose post-lock settle
```

Do not make custom objects bypass the interaction model.

---

## Alignment Formula

Standard socket alignment:

```text
DesiredObjectWorldTransform =
  TargetWeaponPointWorldTransform * Inverse(ObjectInsertTipLocalTransform)
```

For mirrored presentation:

```text
CanonicalDesiredTransform → PresentationMirror → PresentedDesiredTransform
```

Do not recompute insert axis from mirrored start/end positions if that changes semantic direction.

---

## Failure And Recovery

Common failures:

```text
insert tip missing
weapon target socket missing
axis invalid
object rotation cannot align
insertion path blocked by external obstruction result
mirrored presentation breaks path consistency
network correction changes object state mid-insert
```

Recovery options:

```text
pause before commit
back object out along extract axis
re-align from PreAlign
blend to authority object state
fallback to failed interaction state
```

---

## Debug Requirements

Debug should show:

```text
object type
insert target
insert tip transform
insert axis local/world/presented
alignment quality
insertion depth
seat quality
lock readiness
commit fired or not
failure reason
```

---

## Final Formula

```text
Object insertion quality =
  insert tip alignment
  + semantic axis/path alignment
  + insertion depth
  + seat/lock readiness
  + commit-safe recovery.
```
