---
id: weapon-transform-space-contract
title: Weapon Transform Space Contract
status: draft
version: 26.602.1610
tags: [ weapon, interaction, transforms, sockets, mirroring, unreal-engine ]
---

# Weapon Transform Space Contract

## Purpose

This document defines the transform-space contract for weapon interaction.

Most weapon interaction bugs come from mixing spaces silently:

```text
weapon local space
object local space
hand socket space
character component space
world space
canonical interaction space
mirrored/presentation space
network replicated state space
```

This document defines what each space means and when conversion happens.

---

## Scope Boundary

This document covers transform handling for weapon interaction only:

```text
weapon sockets
object sockets
hand IK targets
moving part axes
reload paths
mirroring
network reconstruction
Control Rig inputs
```

It does not own camera transforms, body orientation solving, locomotion root motion, projectile trajectories, or damage traces.

---

## Core Rule

```text
Always know the source space and target space of every transform.
Do not infer semantic axes from arbitrary world-space positions.
```

A transform should carry or imply:

```text
Space
Owner
SemanticMeaning
MirrorState
TimeSource if replicated/reconstructed
```

---

## Weapon Local Space

Weapon local space is the authored source of truth for weapon interaction points.

Examples:

```text
MainGripSocket
SupportGripSocket
MagazineWellSocket
BoltHandleSocket
PumpSocket
MuzzleSocket
StockShoulderSocket
CheekReferenceSocket
SightReferenceSocket
```

Rules:

```text
interaction points are authored relative to the weapon mesh/root
local axes describe semantic movement directions
local axes remain meaningful even after the weapon moves in world space
```

---

## Object Local Space

Reload/manipulation objects have their own authored local space.

Examples:

```text
MagazineHandGripSocket
MagazineInsertTipSocket
ShellPinchSocket
BatteryInsertTipSocket
```

Object-to-weapon alignment should use socket transforms:

```text
DesiredObjectWorldTransform =
  TargetWeaponPointWorldTransform * Inverse(ObjectInsertTipLocalTransform)
```

---

## Hand Socket / Hand Rig Space

Hand socket or rig space describes how the hand should align to an object/contact.

Examples:

```text
PalmCenter
GripPoseReference
IndexFingerReference
SupportPalmReference
```

Rules:

```text
hand pose assets should state their expected contact transform
hand IK target should align palm/contact reference, not arbitrary wrist origin only
finger pose may be driven separately by GripPoseId and FingerGripAlpha
```

---

## Character Component Space

Character component space is useful for animation and Control Rig.

Rules:

```text
world-space runtime targets may be converted into component space before AnimGraph/Control Rig
Control Rig should convert once at graph start if it receives world-space targets
avoid converting the same transform multiple times
```

---

## World Space

World space is useful for runtime composition:

```text
weapon actor/component transform
character mesh transform
object actor transform
remote reconstruction target transforms
```

World space should not replace authored local semantic data.

Incorrect:

```text
InsertAxis = MagazineWellWorldPosition - MagazineStartWorldPosition
```

Correct:

```text
InsertAxisWorld = WeaponTransform.TransformVectorNoScale(LocalInsertAxis)
```

---

## Canonical Interaction Space

Canonical interaction space is the non-mirrored authored interaction logic.

Rules:

```text
hand roles are resolved canonically
interaction steps are stored canonically
object visual states are stored canonically
reload plans are built canonically
```

Presentation mirroring should not create a separate mirrored gameplay plan.

---

## Presentation / Mirrored Space

Presentation space is the final visual output space after optional global character mirroring.

Rules:

```text
canonical transforms may be mirrored for presentation
mirror in one defined stage only
object paths, hand targets, elbow poles, and weapon pose offsets must be mirrored consistently
```

Do not mix canonical and already-presented transforms in the same solver step.

---

## Network Replicated State Space

Network state should replicate semantic state, not per-frame transforms.

Replicate:

```text
pose state
interaction phase
step id
step start server time
step duration
object visual state
moving part alpha/state
presentation mirror state if needed
revision
```

Reconstruct locally:

```text
current step alpha
canonical target transforms
presented/mirrored transforms
Control Rig inputs
```

---

## Moving Part Axes

Moving part axes are semantic local axes.

Examples:

```text
LocalBoltAxis
LocalSlideAxis
LocalPumpAxis
LocalLeverAxis
```

Rules:

```text
transform local axis to world/presentation space
never recompute axis from two arbitrary corrected positions if that changes authored meaning
```

---

## Transform Naming Convention

Use names that expose space:

```text
MagazineWell_Local
MagazineWell_World
MagazineWell_CanonicalWorld
MagazineWell_PresentedWorld
LeftHandTarget_World
LeftHandTarget_Component
ObjectInsertTip_Local
InsertAxis_Local
InsertAxis_World
InsertAxis_PresentedWorld
```

Avoid ambiguous names:

```text
TargetTransform
SocketTransform
FinalTransform
```

unless the containing type explicitly declares the space.

---

## Debug Requirements

Debug should show:

```text
transform space
canonical transform
presented transform
mirror stage
local axis
world axis
presented axis
source socket
target socket
conversion chain
```

---

## Final Formula

```text
Transform-space correctness =
  authored local semantic data
  + explicit conversion chain
  + canonical interaction state
  + one presentation mirror stage
  + local reconstruction from semantic network state.
```
