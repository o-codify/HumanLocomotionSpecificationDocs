---
id: weapon-pose-state-model
title: Weapon Pose State Model
status: draft
version: 26.602.1429
tags: [ weapon, pose-state, holding, aiming, hip-fire, ads ]
---

# Weapon Pose State Model

## Purpose

This document defines the engine-agnostic weapon pose state model for weapons held in hands.

It extends weapon interaction beyond reloads into normal weapon-in-hands behavior:

```text
relaxed holding
low ready
high ready
hip fire
point aim
aim down sights
sprinting with weapon
cover aim
reload from current pose
mechanism manipulation from current pose
```

Reload is a temporary disruption of the current pose state, not a completely separate system.

---

## Core Principle

```text
Weapon pose state defines how the weapon is currently carried, aimed, stabilized, and allowed to fire.
Reload and mechanism manipulation temporarily modify that pose state and then restore or transition to a valid pose.
```

The system must not assume that every weapon is always held in ADS or always held at the hip.

---

## Pose State List

Recommended high-level states:

```text
Unarmed
Holstered
RelaxedCarry
LowReady
HighReady
HipFire
PointAim
AimDownSights
Reloading
ManipulatingMechanism
SprintingWithWeapon
InCoverReady
InCoverAim
Blocked
```

---

## Pose State Data

```text
WeaponPoseState:
  StateId
  ShoulderSide
  MainHand
  SupportHand
  RequiredContacts
  PreferredContacts
  AimConstraint
  MuzzlePolicy
  MovementFreedom
  RotationFreedom
  FirePermissionPolicy
  ReloadEntryPolicy
  ReloadExitPolicy
  AnimationProfileId
```

---

## Pose State Meanings

### RelaxedCarry

Purpose:

```text
weapon is held but not immediately ready to fire
```

Typical properties:

```text
muzzle lowered or safe
hands may remain on main/support grips
shoulder contact optional
high movement freedom
fire not allowed or delayed by raise transition
```

### LowReady

Purpose:

```text
weapon is ready but muzzle is lowered below target line
```

Typical properties:

```text
main grip active
support grip preferred
shoulder contact optional or partial
muzzle safe/down
can transition quickly to hip fire or ADS
```

### HighReady

Purpose:

```text
weapon is ready but muzzle is raised or held near line of sight without full ADS
```

Typical properties:

```text
main grip active
support grip active
shoulder contact preferred
muzzle near target line
can transition quickly to ADS
```

### HipFire

Purpose:

```text
weapon can fire without exact sight alignment
```

Typical properties:

```text
main grip active
support grip active for two-handed weapons
shoulder contact optional but preferred for rifles
camera/sight alignment not exact
muzzle follows aim direction approximately
higher movement freedom than ADS
lower precision than ADS
```

### PointAim

Purpose:

```text
weapon is aimed more deliberately than hip fire but without strict sight alignment
```

Typical properties:

```text
weapon closer to sight line
muzzle follows aim direction more tightly
shoulder contact preferred
movement freedom between HipFire and ADS
```

### AimDownSights

Purpose:

```text
weapon sight is aligned with eye/camera target for precise fire
```

Typical properties:

```text
main grip active
support grip active
shoulder contact preferred for long guns
cheek/contact optional if modeled
strict weapon orientation
lower movement freedom
higher precision
```

### SprintingWithWeapon

Purpose:

```text
character moves fast while weapon is not fully ready
```

Typical properties:

```text
muzzle lowered or angled safe
fire usually blocked
reload usually blocked or interrupted
hands keep minimum safe hold
ADS not allowed
```

### Reloading

Purpose:

```text
temporary pose state while reload action plan executes
```

Reloading remembers entry pose:

```text
PreviousPoseState
ReloadPoseVariant
DesiredExitPoseState
```

### ManipulatingMechanism

Purpose:

```text
temporary pose state while bolt, slide, charging handle, pump, lever, or other mechanism is operated
```

---

## Pose Transition Rules

Transitions should be explicit and validated.

Common transitions:

```text
RelaxedCarry → LowReady
LowReady → HipFire
HipFire → AimDownSights
AimDownSights → HipFire
HipFire → Reloading
AimDownSights → Reloading
Reloading → PreviousPoseState
Reloading → LowReady fallback
HipFire → SprintingWithWeapon
SprintingWithWeapon → LowReady
LowReady → Holstered
```

Reload should restore the previous pose if valid:

```text
EntryPose = AimDownSights
Reload starts
Pose = Reloading with EntryPose remembered
Reload completes
Pose = AimDownSights if still valid, otherwise HipFire or LowReady
```

---

## Fire Permission by Pose

```text
RelaxedCarry:
  fire blocked or delayed by raise transition

LowReady:
  fire blocked or allowed only after raise transition

HighReady:
  fire may be allowed with penalty

HipFire:
  fire allowed if weapon mechanical state allows

PointAim:
  fire allowed with better stability than hip fire

AimDownSights:
  fire allowed with best stability/precision

SprintingWithWeapon:
  fire blocked unless weapon policy allows special case

Reloading:
  fire depends on reload/fire policy and commit state
```

---

## Muzzle Policy by Pose

```text
RelaxedCarry:
  lowered safe

LowReady:
  lowered but controllable

HighReady:
  near aim line

HipFire:
  approximate aim direction

PointAim:
  tightened aim direction

AimDownSights:
  sight/camera alignment

Reloading:
  inherited from reload policy, often KeepAimApproximate or LoweredSafe

SprintingWithWeapon:
  lowered or angled safe
```

---

## Stability Requirements by Pose

```text
RelaxedCarry:
  one or two hands depending on weapon weight

LowReady:
  main hand plus optional support hand

HipFire:
  main hand plus support hand for long guns

AimDownSights:
  main hand + support hand + shoulder contact preferred for long guns

Reloading:
  remaining contacts must satisfy reload stability requirement
```

---

## Pose State and Reload

Reload does not replace holding logic.

Reload uses:

```text
CurrentPoseState
WeaponHoldState
WeaponInteractionData
ReloadIntent
```

and produces:

```text
ReloadEntryPose
ReloadPoseAdjustment
ReloadActionPlan
ReloadExitPose
```

Reload exit must be valid. If entry pose is no longer valid, fallback:

```text
ADS → HipFire → LowReady → RelaxedCarry
```

---

## Pose State and Locomotion

Locomotion affects pose availability.

Examples:

```text
standing allows ADS
walking allows ADS with sway/movement penalty
sprinting forces SprintingWithWeapon or LowReady
falling blocks ADS and reload
crouching may improve stability
cover enables InCoverReady and InCoverAim
```

---

## Networking Rule

Replicate pose state and timestamps, not per-frame IK.

Conceptual replicated fields:

```text
CurrentPoseState
DesiredPoseState
PreviousPoseState optional
ShoulderSide
PoseStartServerTime
PoseBlendDuration
PoseRevision
bIsAiming
bIsFiring
```

Clients reconstruct visual pose locally.

---

## Final Formula

```text
Weapon pose state =
  how the weapon is held
  + how it is aimed
  + what contacts stabilize it
  + what firing/reload transitions are allowed
  + how it returns after temporary actions.
```
