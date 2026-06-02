---
id: weapon-draw-and-holster-interaction
title: Weapon Draw and Holster Interaction
status: draft
version: 26.602.2053
tags: [ weapon, interaction, draw, holster, animation, contacts ]
---

# Weapon Draw and Holster Interaction

## Purpose

This document defines draw and holster as weapon interaction actions.

This is not an inventory/equipment system. It only describes the procedural physical interaction of moving a weapon between a body-attached visual state and an in-hands hold pose.

---

## Scope Boundary

Owned here:

```text
hand reaches weapon grip
weapon visual detaches from body presentation socket
weapon visual enters hand-held pose
support hand joins if required
hold pose stabilizes
weapon visual returns to body presentation socket during holster
IK targets activate/clear
contacts activate/clear
```

Not owned here:

```text
which weapon is selected
inventory slot rules
loot/pickup system
weapon persistence
ammo state
camera side
locomotion speed
```

---

## Draw Phases

Recommended draw phases:

```text
Inactive
ReachBodyWeapon
GripBodyWeapon
DetachFromBody
BringToReadyPose
JoinSupportHand
StabilizeHold
Complete
```

---

## Holster Phases

Recommended holster phases:

```text
PrepareHolster
ReleaseSupportHand
MoveWeaponToBodySocket
AttachToBody
ReleaseMainHand
ClearHoldPose
Complete
```

---

## Visual Body Attachment

A body-attached weapon is a visual presentation state.

Example body visual sockets:

```text
BackWeaponSocket
ChestWeaponSocket
HipWeaponSocket
HolsterSocket
SlingSocket
```

Weapon interaction may consume a body attachment transform from an external equipment/presentation system.

It does not own the equipment inventory decision.

---

## Draw Interaction Flow

```text
1. external system marks weapon as selected/available
2. weapon interaction receives body visual attachment transform
3. main hand reaches canonical grip target
4. main hand contact becomes PreGrip then Active
5. weapon visual detaches from body presentation socket
6. weapon follows main hand toward ready pose
7. support hand joins if required
8. hold pose enters Stabilizing
9. hold pose becomes Stable
```

---

## Holster Interaction Flow

```text
1. external system requests holster interaction
2. weapon interaction chooses holster path target
3. support hand releases if active
4. main hand carries weapon toward body presentation socket
5. weapon visual attaches to body socket
6. main hand releases
7. hand IK targets clear
8. hold pose exits
```

---

## Contact Rules

Draw requires:

```text
main hand reaches main grip
weapon does not become unsupported during detach
support hand joins before entering two-handed stable pose if required
```

Holster requires:

```text
main hand keeps control until weapon visual is attached to body
support hand releases cleanly
hold pose exits after visual attachment is stable
```

---

## Mirroring

Draw/holster uses canonical interaction state and mirrored presentation output.

Rules:

```text
body attachment transform may be mirrored by presentation layer
hand targets are generated canonically
presentation mirror transforms final draw/holster path
no separate left-handed draw plan is required
```

---

## Interruption

Draw can be interrupted:

```text
before grip
after grip before detach
after detach before ready pose
while support hand joins
```

Holster can be interrupted:

```text
before body attach
at body attach
after body attach before hand release
```

Commit boundary:

```text
DetachFromBody
AttachToBody
```

Before detach, draw can usually return to body-attached visual state.
After detach, recovery should keep the weapon controlled by hand or move to a safe ready pose.

Before attach, holster can return to a hold pose.
After attach, holster should finish or restore from body-attached state.

---

## Network Reconstruction

Replicate interaction phase/time, not per-frame hand transforms.

Useful replicated concepts:

```text
DrawHolsterActionId
CurrentPhase
PhaseStartServerTime
PhaseDuration
BodyAttachmentId optional
PoseState
ObjectVisualState
Revision
```

Remote clients reconstruct draw/holster animation using authored paths and presentation mirror state.

---

## Editor Preview

Preview should support:

```text
draw from body socket to LowReady
holster from LowReady to body socket
canonical and mirrored presentation
main hand path
support hand join timing
body attach/detach commit point
failure if body socket missing
failure if main grip socket missing
```

---

## Debug Requirements

Debug should show:

```text
action type: draw or holster
phase
phase alpha
body attachment socket/transform
main hand target
support hand target
attach/detach commit status
current hold pose lifecycle phase
interruption recovery target
```

---

## Final Formula

```text
Draw/holster interaction =
  visual body attachment
  + hand reach/grip
  + attach/detach commit
  + support hand join/release
  + hold pose lifecycle transition
  + network phase reconstruction.
```
