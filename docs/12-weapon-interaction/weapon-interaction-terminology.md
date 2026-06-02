---
id: weapon-interaction-terminology
title: Weapon Interaction Terminology
status: draft
version: 26.602.1415
tags: [ weapon, terminology, glossary, implementation ]
---

# Weapon Interaction Terminology

## Purpose

This document defines shared terminology for the weapon interaction section.

Use this document to avoid mismatched names such as reload step vs action step, visual attachment vs gameplay attachment, or interaction phase vs animation phase.

---

## Core Terms

### Weapon Interaction Data

Authored or runtime data that describes what exists on a weapon and how it can be used.

Defined by [Weapon Interaction Data Model](./weapon-interaction-data-model.md).

Examples:

```text
feature set
contact points
interaction points
local axes
moving parts
reload objects
body slots
mechanical state
```

---

### Contact Point

A point used to hold or stabilize the weapon.

Examples:

```text
MainGrip
SupportGrip
StockShoulder
SlingContact
BipodContact
SurfaceContact
```

A contact point is not automatically an interaction point.

---

### Interaction Point

A point used for manipulation.

Examples:

```text
MagazineWell
Bolt
ChargingHandle
Slide
Pump
ShellInsert
Chamber
MagazineRelease
```

Interaction points contain access regions, hand policies, axes, and required stability.

---

### Moving Part

A weapon part that moves relative to the weapon body.

Examples:

```text
BoltBone
SlideBone
PumpBone
LeverBone
BreakActionBone
```

Rule:

```text
Action drives the moving part.
Hand target follows the moving part follow point.
```

---

## Planning Terms

### Reload Intent

The player's or AI's high-level reload request.

Examples:

```text
FullReload
TacticalReload
EmergencyReload
LoadOne
CycleOnly
Unload
```

---

### Action Plan

A validated list of steps that can be executed over time.

UE runtime name:

```text
FReloadActionPlan
```

An action plan is built by the planner and executed by the runtime component.

---

### Action Step

One discrete step in an action plan.

UE runtime name:

```text
FReloadActionStep
```

Examples:

```text
ReachSocket
GripObject
MoveAlongAxis
InsertObject
OperateMovingPart
CommitMechanicalState
ReturnHand
```

---

### Step Definition

Authored template data that describes a step before runtime hand/object resolution.

UE DataAsset name:

```text
FReloadActionStepDefinition
```

Defined in [Weapon Reload Sequence Profile for Unreal Engine](./weapon-reload-sequence-profile-ue.md).

---

### Interaction Phase

The current high-level phase of an interaction step.

UE runtime name:

```text
EWeaponInteractionPhase
```

Values:

```text
None
PreparePose
Reach
PreGrip
GripContact
VisualAttach
Manipulate
GameplayCommit
Release
Return
Settle
Recovery
```

Do not use a separate `ReloadPhase` term unless it is an alias for `InteractionPhase`.

---

### Step Alpha

Normalized time inside the current action step.

```text
StepAlpha = (ServerTimeNow - StepStartServerTime) / StepDuration
```

Step alpha is used for visual reconstruction and animation execution.

---

## Object Terms

### Reload Object

An object manipulated during reload or weapon operation.

Examples:

```text
Magazine
Round
ShotgunShell
Battery
EnergyCell
Clip
SpeedLoader
```

---

### Gameplay Object

Server-authoritative inventory, ammo, or loot object state.

Examples:

```text
inventory magazine entry
ammo stack
lootable dropped magazine actor
```

Gameplay objects affect ammo, inventory, pickup, and firing state.

---

### Visual Object

Client-side or replicated visual representation used for animation.

Examples:

```text
magazine mesh attached to hand
shell mesh moving into tube
battery visual attached to weapon
cosmetic dropped magazine
```

Visual objects do not by themselves change gameplay state.

---

### Visual Attachment

A visual object becomes attached to a hand, weapon socket, body slot, or world transform.

Visual attachment can be predicted locally.

---

### Gameplay Attachment

Server-authoritative attachment or ownership change.

Examples:

```text
magazine becomes locked into weapon
round becomes inserted
magazine becomes lootable dropped object
```

Gameplay attachment should occur only at commit points.

---

### Object Visual State

The current visual placement state of a reload object.

UE runtime name:

```text
EReloadObjectVisualState
```

Values:

```text
Hidden
InWeapon
InLeftHand
InRightHand
InBodySlot
DroppedWorld
```

---

## Networking Terms

### Commit Point

A server-authoritative gameplay state transition.

Examples:

```text
MagazineDetached
MagazineLocked
RoundInserted
BoltOpened
BoltClosed
PumpBack
PumpForward
ReloadCompleted
```

Visual animation may lead into the commit point, but gameplay state changes at the commit point.

---

### Mechanical State

Gameplay-relevant weapon internals.

Examples:

```text
MagazineInserted
MagazineLocked
RoundChambered
BoltOpen
NeedsCycle
AmmoInMagazine
AmmoInChamber
InternalAmmoCount
```

UE runtime name:

```text
FReplicatedWeaponMechanicalState
```

---

### Mechanical State Revision

A small replicated counter that increments when authoritative mechanical state changes.

Purpose:

```text
clients can detect state changes even if multiple booleans look similar during correction
```

---

### Reload Revision

A small replicated counter that increments when reload phase/step/interruption changes.

Purpose:

```text
clients can detect new reload state and reconcile prediction
```

---

### Prediction Id

A client-generated identifier for matching local predicted visuals to server confirmation/rejection.

Prediction id does not make client gameplay state authoritative.

---

## Animation Terms

### Hand IK Target

A transform and supporting data used by animation systems to place a hand.

UE animation name:

```text
FWeaponHandIKTarget
```

Contains:

```text
target transform
elbow pole
position alpha
rotation alpha
finger grip alpha
grip pose id
contact state
```

---

### PreGrip

A pose near the object or interaction point before contact/attachment.

Used to avoid snapping directly into final grip.

---

### GripContact

The moment where the hand visually contacts the object or mechanism.

Visual attachment may begin here depending on visual policy.

---

### Settle

Short final blend that restores stable contact, removes small offsets, and completes the visual action.

---

## UE Document Ownership

```text
weapon-interaction-profile-ue.md
  DataAssets, profile authoring, validation, preview-facing authored data.

weapon-reload-sequence-profile-ue.md
  authored reload sequence templates, timing, visual policy, commit policy.

weapon-reload-planner-ue.md
  converting request/context/profile/sequence data into FReloadActionPlan.

weapon-runtime-implementation-ue.md
  components, replicated state, RPCs, OnRep, server executor, prediction, tick order.

weapon-animation-control-rig-ue.md
  AnimInstance, Control Rig, IK targets, animation graph execution.

weapon-object-lifecycle-ue.md
  gameplay object vs visual object, attachment, drop, prediction reconciliation.

weapon-editor-preview-ue.md
  validation reports, preview actor/component, socket axis and reload preview.
```

---

## Terms To Avoid

Avoid ambiguous terms:

```text
ReloadPhase
```

Use:

```text
InteractionPhase
```

Avoid:

```text
Attachment
```

Use either:

```text
VisualAttachment
GameplayAttachment
```

Avoid:

```text
MagazineObject
```

Use one of:

```text
GameplayObject
VisualObject
ReloadObject
```

---

## Final Formula

```text
Consistent terminology =
  clear ownership
  + clear runtime state
  + clear visual/gameplay separation
  + fewer implementation mistakes.
```
