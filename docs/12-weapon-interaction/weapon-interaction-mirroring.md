---
id: weapon-interaction-mirroring
title: Weapon Interaction Mirroring
status: draft
version: 26.602.1507
tags: [ weapon, interaction, mirroring, animation, presentation, control-rig ]
---

# Weapon Interaction Mirroring

## Purpose

This document defines how weapon interaction remains compatible with global full-body character mirroring.

This is not a left-handed character system.
This is not dynamic hand transfer.
This is not a real gameplay swap of main hand and support hand.

The goal is:

```text
one canonical authored interaction
+ optional global character presentation mirror
= visually mirrored stance, shoulder side, camera side, weapon pose, and interaction animation
```

---

## Core Principle

```text
Weapon interaction authors one canonical side.
Global character presentation may mirror the final pose.
Weapon interaction must produce mirror-safe targets.
```

The canonical authored side is usually:

```text
BaseMainHand = Right
BaseSupportHand = Left
BaseShoulder = RightShoulder
```

When the player switches presentation side, the system does not create a true left-handed interaction plan.

Instead:

```text
canonical interaction state → mirror transform → mirrored presentation pose
```

---

## Not Handedness

Do not model this as:

```text
DominantHand = Left
LeftHandedCharacter
AmbidextrousCharacter
MainHand changes from Right to Left in gameplay logic
SupportHand changes from Left to Right in gameplay logic
```

The base interaction logic remains canonical.

Mirroring is a presentation transform applied by the broader character presentation/animation system.

---

## Owned By Weapon Interaction

Weapon interaction owns compatibility with mirroring:

```text
producing mirror-safe interaction targets
marking which transforms are canonical
marking which transforms may be mirrored
keeping hand/contact roles stable in canonical space
ensuring reload/manipulation paths remain valid after presentation mirroring
providing debug data for canonical vs mirrored output
supporting editor preview of mirrored presentation
```

Weapon interaction does not own the global decision to mirror the whole character.

External systems may own:

```text
camera side
character presentation mirror flag
full-body animation mirroring
locomotion mirroring
body orientation mirroring
```

---

## Mirror State Input

Weapon interaction may read a small external state:

```text
FCharacterPresentationMirrorState:
  bMirroredPresentation
  MirrorPlane
  MirrorOrigin
  MirrorSpace
  MirrorRevision
```

Common simplified state:

```text
PresentationSide = RightShoulderView | LeftShoulderView
```

Weapon interaction should not decide why this state changed.

---

## Canonical Space vs Presentation Space

Use two conceptual spaces:

```text
CanonicalInteractionSpace:
  authored weapon interaction logic lives here.

PresentationSpace:
  final character/weapon animation may be mirrored here.
```

Rules:

```text
solvers build canonical hand/object/weapon targets
interaction state stores canonical hand roles
replication sends canonical interaction state plus mirror/presentation side if needed
animation presentation applies mirror transform after or during pose construction
```

---

## What Is Mirrored

Presentation mirroring may affect:

```text
full-body stance
weapon hold pose
weapon shoulder presentation
hand IK target presentation
elbow pole presentation
weapon pose offset presentation
reload hand path presentation
magazine/object visual path presentation
muzzle/sight visual alignment presentation
fire visual pose impulse presentation
```

---

## What Is Not Mirrored As Gameplay Logic

Do not mirror these as separate gameplay logic:

```text
canonical main hand role
canonical support hand role
canonical reload plan identity
canonical interaction step ids
canonical commit points
canonical mechanical interaction state
external inventory/ammo logic
external fire backend state
```

The mirrored presentation may show the weapon on the other shoulder, but the interaction plan remains the same canonical plan.

---

## Weapon-Side Interaction Points

Weapon-side interaction points remain authored truth.

Examples:

```text
MagazineWellSocket
BoltHandleSocket
ChargingHandleSocket
PumpSocket
SafetyLeverSocket
EjectionPortSocket
```

Important rule:

```text
Do not invent mirrored weapon controls that do not exist on the weapon.
```

If the whole character and weapon mesh presentation is mirrored, the authored weapon points are transformed visually by the same mirror presentation.

If a weapon control is physically one-sided and the game does not mirror the weapon mesh itself, the interaction must use the real authored control location.

---

## Socket Axes Under Mirroring

Local socket axes must remain semantically valid.

For each authored axis:

```text
LocalInsertAxis
LocalExtractAxis
LocalSlideAxis
LocalPumpAxis
LocalBoltAxis
```

The presentation layer must transform directions with the mirror transform, not recompute them from positions.

Correct:

```text
CanonicalAxisWorld = WeaponTransform.TransformVectorNoScale(LocalAxis)
MirroredAxisWorld = MirrorDirection(CanonicalAxisWorld)
```

Incorrect:

```text
MirroredAxis = MirroredTargetPosition - MirroredStartPosition if that changes authored semantic direction
```

---

## Hand Targets

Canonical target data:

```text
CanonicalRightHandTarget
CanonicalLeftHandTarget
CanonicalRightElbowPole
CanonicalLeftElbowPole
```

Presentation output may mirror them:

```text
PresentedRightSideTarget
PresentedLeftSideTarget
```

The naming should remain clear:

```text
canonical hand role
presented mirrored transform
```

Avoid ambiguous names such as `RealLeftReloadHand` if the system is not actually changing gameplay hand roles.

---

## Reload Paths

Reload action plans remain canonical.

Example canonical plan:

```text
Right hand stabilizes main grip
Left hand reaches magazine
Left hand extracts magazine
Left hand inserts magazine
Left hand returns support grip
```

Mirrored presentation may display the same plan on the opposite side of the body.

The reload plan id, step ids, and commit points remain identical.

---

## Object Visual States

Object visual state remains canonical:

```text
Hidden
InWeapon
InLeftHand
InRightHand
InBodySlot
DroppedWorld
```

Presentation may mirror the transform of that object.

Do not create duplicate mirrored object states such as:

```text
InMirroredLeftHand
InMirroredRightHand
```

Mirroring is transform presentation, not state explosion.

---

## Networking

Replicate interaction state once.

Recommended replicated concept:

```text
CanonicalInteractionState
PresentationMirrorState or PresentationSide
StepStartServerTime
StepDuration
StepId
ObjectVisualState
PoseState
Revision
```

Do not replicate separate left-side and right-side interaction plans.

Remote clients reconstruct:

```text
canonical interaction phase
canonical targets
presentation mirror transform
final mirrored visual pose
```

---

## Control Rig Integration

Control Rig may receive either:

```text
canonical targets + mirror flag
```

or:

```text
already mirrored presentation targets
```

Preferred rule:

```text
Mirror in one place only.
```

Avoid double mirroring:

```text
AnimGraph mirrors pose
Control Rig mirrors targets again
weapon object path mirrors separately a third time
```

The implementation must define the mirror stage clearly.

---

## Editor Preview

Editor preview should support:

```text
Preview Canonical
Preview Mirrored Presentation
Toggle PresentationSide
Show canonical targets
Show mirrored targets
Show mirror plane
Show transformed socket axes
Show reload path before/after mirror
Show object visual path before/after mirror
```

Validation should warn when:

```text
axis changes semantic meaning after mirror
mirrored target becomes unreachable
Control Rig receives already-mirrored data but mirror flag is still enabled
object visual path is mirrored twice
weapon-side control is assumed mirrored when the mesh is not mirrored
```

---

## Debug Requirements

Debug should show:

```text
bMirroredPresentation
PresentationSide
MirrorPlane
CanonicalPoseState
CanonicalInteractionStep
CanonicalMainHandRole
CanonicalSupportHandRole
CanonicalHandTargets
PresentedHandTargets
CanonicalObjectTransform
PresentedObjectTransform
CanonicalAxisWorld
PresentedAxisWorld
MirrorStage
```

---

## Failure Cases

Common failures:

```text
double mirrored IK targets
mirrored weapon but non-mirrored object path
mirrored hands but non-mirrored elbow poles
socket axes recomputed incorrectly
canonical hand role renamed as if gameplay hand changed
remote client receives mirrored state but mirrors again
editor preview validates only canonical side
```

---

## Final Formula

```text
Weapon interaction mirroring =
  canonical authored interaction
  + external presentation mirror state
  + one clearly defined mirror stage
  + mirror-safe targets and axes
  + no gameplay hand-role swap.
```
