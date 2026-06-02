---
id: procedural-weapon-reloading
title: Procedural Weapon Reloading
status: draft
version: 26.602.2053
tags: [ weapon, reload, upper-body, ik, procedural-animation, multiplayer ]
---

# Procedural Weapon Reloading

## Purpose

This document defines procedural weapon reloading as an engine-agnostic interaction sequence built on top of weapon holding, weapon interaction data, solvers, animation execution, and networking.

It describes reload scenarios and reload-specific action semantics.

It does not own:

- the full data model, which is defined in [Weapon Interaction Data Model](./weapon-interaction-data-model.md)
- hand/stability/reachability planning, which is defined in [Weapon Solvers and Planning](./weapon-solvers-and-planning.md)
- animation execution phases, which are defined in [Weapon Animation Execution](./weapon-animation-execution.md)
- multiplayer authority and replication rules, which are defined in [Weapon Interaction Networking](./weapon-networking.md)
- UE runtime implementation, which is defined in [Weapon Runtime Implementation for Unreal Engine](./weapon-runtime-implementation-ue.md)

---

## Core Idea

```text
Reloading = temporary disruption of a valid weapon hold pose
          + object transfer
          + directed insertion/extraction/operation
          + mechanical state commit
          + restoration of a valid weapon hold pose
```

Reloading is not a single animation. It is a sequence of validated manipulation steps.

---

## Scope

The reload model must support:

```text
magazine reloads
tactical reloads
emergency reloads
single-round insertion
shotgun shell loading loops
bolt cycling
slide racking
pump action
break-action open/close
bullpup magazine positions
side/top/bottom magazine wells
angled magazines
weapons with and without stock
right-shoulder and left-shoulder stances
interruption and recovery
```

---

## Required Inputs

A reload sequence is built from:

```text
ReloadRequest
WeaponHoldState
WeaponInteractionData
WeaponMechanicalState
Inventory / BodySlots
CharacterState
```

The structure and meaning of these inputs are defined in:

- [Weapon Interaction Data Model](./weapon-interaction-data-model.md)
- [Weapon Holding and Stabilization](./weapon-holding.md)
- [Weapon Solvers and Planning](./weapon-solvers-and-planning.md)

---

## Reload Action Sequence

A reload sequence is composed of reusable actions:

```text
PrepareWeaponPose
ResolveHandAssignment
Regrip
ReleaseGrip
ReachPoint
GripObject
DetachObject
ExtractObject
MoveObjectToBodySlot
DropObject
FetchObjectFromBodySlot
AlignObject
InsertObject
LockObject
ReleaseObject
OperateMechanism
CommitMechanicalState
ReturnHandToGrip
RestoreWeaponPose
```

The planner decides which actions are required. This document describes reload-specific usage of those actions.

---

## Reload Intents

Supported reload intents:

```text
FullReload
TacticalReload
EmergencyReload
LoadOne
CycleOnly
Unload
ClearMalfunction optional
```

### Emergency Reload

Purpose:

```text
restore firing capability as quickly as possible
```

Typical sequence:

```text
1. Move weapon to reload pose if needed.
2. Stabilize weapon.
3. Release manipulation hand.
4. Remove old magazine if present.
5. Drop old magazine.
6. Fetch new magazine.
7. Align magazine.
8. Insert and lock magazine.
9. Cycle bolt/slide if required.
10. Return hand to valid hold.
```

### Tactical Reload

Purpose:

```text
replace magazine while preserving the old magazine
```

Typical sequence:

```text
1. Move weapon to reload pose if needed.
2. Stabilize weapon.
3. Remove old magazine.
4. Stow old magazine in body slot.
5. Fetch new magazine.
6. Insert and lock new magazine.
7. Cycle mechanism if required.
8. Return hand to valid hold.
```

### Load One

Purpose:

```text
insert one round or shell without necessarily completing a full reload
```

Typical sequence:

```text
1. Stabilize weapon.
2. Fetch one round/shell.
3. Align round/shell insert tip.
4. Insert along defined axis/path.
5. Commit loaded round/shell.
6. Continue loop or return hand to grip.
```

### Cycle Only

Purpose:

```text
operate bolt, slide, pump, or charging handle without changing magazine/ammo object
```

Typical sequence:

```text
1. Stabilize weapon.
2. Assign hand to mechanism point.
3. Reach and grip mechanism.
4. Move mechanism along operate path.
5. Commit mechanical state.
6. Return hand to grip.
```

---

## Magazine Insertion Model

A detachable magazine is a reload object.

Required magazine object points:

```text
HandGripPoint
InsertTipPoint
LockPoint optional
```

Required weapon points:

```text
MagazineWell
MagazinePreInsert optional
MagazineRelease optional
```

Final alignment rule:

```text
DesiredMagazineWorldTransform =
  WeaponMagazineWellWorldTransform
  * Inverse(MagazineInsertTipLocalTransform)
```

This allows the magazine root and shape to be arbitrary.

---

## Direction Is Socket Axis, Not Point-To-Point Line

Insertion direction must come from authored local axes.

Bad approach:

```text
direction = normalize(WeaponSocketPosition - MagazineTipPosition)
```

Correct approach:

```text
InsertDirection = WeaponMagazineWellTransform.TransformAxis(LocalInsertAxis)
```

This supports:

```text
angled magazines
curved magazines
side-mounted magazines
top-mounted magazines
bottom magazines
bullpup magazines
custom sci-fi magazine wells
```

The axis convention is defined in [Weapon Interaction Data Model](./weapon-interaction-data-model.md).

---

## Pre-Insert and Final Insert

Insertion should use at least two poses:

```text
PreInsertPose = MagazineWellTransform - InsertDirection * InsertDistance
FinalInsertPose = MagazineWellTransform
```

Sequence:

```text
Approach → Align → Insert → Lock
```

The insert motion should not be a direct hand teleport to the final socket.

---

## Magazine Extraction

Extraction may use a different axis from insertion.

```text
ExtractDirection = MagazineWellTransform.TransformAxis(LocalExtractAxis)
```

For simple weapons:

```text
ExtractDirection = -InsertDirection
```

For custom weapons:

```text
extract down/back
insert up/forward
rock-in path
curved path
```

Insert and extract axes should be separately configurable.

---

## Rock-In and Curved Magazine Paths

Some magazines rotate or move through multiple path segments.

Example rock-in path:

```text
1. Hook front/rear point.
2. Rotate magazine around pivot.
3. Lock magazine.
```

Required data:

```text
RockPivotPoint
RockStartAngle
RockEndAngle
LockMotionDistance
```

This is still the same reload system. Only the insertion path changes.

---

## Bottom Magazine Example

Right shoulder initial state:

```text
RightHand = MainGrip
LeftHand = SupportGrip
RightShoulderContact = active
```

Typical sequence:

```text
1. PrepareWeaponPose: lower/roll weapon.
2. RightHand + shoulder stabilize weapon.
3. LeftHand releases SupportGrip.
4. LeftHand reaches MagazineWell.
5. LeftHand grips old magazine.
6. LeftHand extracts magazine.
7. Old magazine is dropped or stowed.
8. LeftHand fetches new magazine.
9. New magazine aligns to PreInsertPose.
10. LeftHand inserts magazine along InsertDirection.
11. Magazine locks.
12. LeftHand releases magazine.
13. LeftHand returns to SupportGrip.
14. Weapon restores previous or updated hold pose.
```

Left shoulder mirrors the hand roles through the solver, not through a separate hard-coded reload.

---

## Right-Side Magazine Example

For a right-side magazine:

```text
AccessRegion = Right
HandPolicy = SameSide
```

If weapon is on left shoulder:

```text
LeftHand + LeftShoulder stabilize.
RightHand changes magazine.
```

If weapon is on right shoulder and right hand is on main grip:

```text
1. LeftHand strengthens support contact.
2. RightHand releases MainGrip.
3. RightHand changes magazine.
4. RightHand returns to MainGrip.
```

The sequence is chosen by stability and hand assignment solvers.

---

## Bullpup / Rear Magazine Example

Bullpup weapons do not require a separate system.

Data example:

```text
MagazineWell = behind MainGrip
AccessRegion = BottomRear or Custom
InsertAxis = authored local axis
ExtractAxis = authored local axis
HandPolicy = OppositeShoulderSide or Custom
```

The same extract/fetch/align/insert/lock sequence applies.

---

## Single-Round Insertion

Single-round reload uses the same object manipulation framework.

Objects:

```text
RoundObject
RoundHandGripPoint
RoundInsertTipPoint
```

Weapon points:

```text
Chamber
ShellInsert
EjectionPort
```

Sequence:

```text
1. Stabilize weapon.
2. Fetch round from body slot.
3. Align round insert tip.
4. Move round along insert path.
5. Commit round inserted/chambered state.
6. Release or continue to next round.
7. Return to valid hold.
```

---

## Shotgun Shell Loading

Tube-fed shotgun loading is a repeated single-round insertion loop.

Typical loop:

```text
FetchShell → AlignShell → InsertShell → CommitShell → FetchNextShell
```

The loop can be interrupted after each committed shell.

Example bottom loading port:

```text
AccessRegion = Bottom
HandPolicy = OppositeShoulderSide
```

Right shoulder usually resolves to left-hand insertion. Left shoulder usually resolves to right-hand insertion.

---

## Bolt, Slide, and Charging Handle Operation

Mechanism operation is a directed manipulation action.

Required data:

```text
OperatePoint
AccessRegion
OperateAxis
OperateDistance
ReturnAxis
ReturnDistance
RequiredStability
LinkedMovingPart optional
```

For moving parts:

```text
Action drives moving part.
Hand follows follow point on moving part.
```

The animation and following behavior are defined in [Weapon Animation Execution](./weapon-animation-execution.md).

---

## Pump Action

Pump action uses a moving fore-end.

Typical sequence:

```text
1. Stabilizing hand remains on main grip and/or shoulder contact.
2. Pump hand remains attached to pump grip point.
3. Pump moves backward along pump-back axis.
4. Pump moves forward along pump-forward axis.
5. Mechanical state commits at the required point.
```

Pump action is not a detached reach action. The hand is constrained to a moving part.

---

## Object Attachment and Commit Points

Reload object attachment states are defined in [Weapon Interaction Data Model](./weapon-interaction-data-model.md).

Gameplay commit authority and replication are defined in [Weapon Interaction Networking](./weapon-networking.md).

Reload-specific commit examples:

```text
MagazineDetached
MagazineLocked
RoundInserted
RoundChambered
ShellInserted
BoltOpened
BoltClosed
PumpBack
PumpForward
ReloadCompleted
```

Visual attachment may happen before gameplay commit, but gameplay state must commit only at the authoritative commit point.

---

## Interruption and Recovery

Reload can be interrupted.

Common partial states:

```text
old magazine removed
new magazine in hand
round/shell in hand
bolt open
pump partially moved
main hand away from grip
support hand away from grip
```

Recovery should restore a valid hold pose, not necessarily complete the reload.

Recovery planning is defined in [Weapon Solvers and Planning](./weapon-solvers-and-planning.md). Recovery animation is defined in [Weapon Animation Execution](./weapon-animation-execution.md).

---

## Multiplayer Rule

Reload is multiplayer-safe only if gameplay state is server-authoritative.

This document defines reload semantics. The multiplayer model is defined in [Weapon Interaction Networking](./weapon-networking.md).

The UE runtime implementation is defined in [Weapon Runtime Implementation for Unreal Engine](./weapon-runtime-implementation-ue.md).

---

## MVP Reload Scope

Minimum reload support:

```text
1. Detachable magazine reload.
2. Bottom magazine well.
3. Weapon with main grip, support grip, optional stock.
4. Magazine object with hand grip and insert tip.
5. Straight-axis extract/insert using authored socket axis.
6. MagazineDetached and MagazineLocked commit points.
7. Return to valid weapon hold.
8. Multiplayer phase/state support through networking layer.
```

---

## Final Formula

```text
Procedural reload =
  reload intent
  + valid hold state
  + authored interaction data
  + planned hand/object/mechanism steps
  + explicit animation phases
  + mechanical commit points
  + recovery path.
```

Reload works for conventional and unusual weapons because the weapon describes geometry and semantics; the code does not need a separate class for every weapon type.
