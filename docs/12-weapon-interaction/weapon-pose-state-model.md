---
id: weapon-pose-state-model
title: Weapon Pose State Model
status: draft
version: 26.602.2053
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
hip fire / NonADSFire pose
point aim
aim down sights
sprint-compatible weapon hold pose driven by external locomotion state
cover-compatible weapon hold variants driven by external cover state
reload from current pose
mechanism manipulation from current pose
```

Reload is a temporary disruption of the current pose state, not a completely separate system.

---

## Core Principle

```text
Weapon pose state defines how the weapon is currently carried, visually aimed, and stabilized by interaction contacts.
Reload and mechanism manipulation temporarily modify that pose state and then restore or transition to a valid pose.
```

The system must not assume that every weapon is always held in ADS or always literally held at the hip.

Weapon pose state does not own locomotion speed, body yaw, turn-in-place, camera behavior, projectile behavior, damage, or inventory state.

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

`SprintingWithWeapon`, `InCoverReady`, and `InCoverAim` are weapon hold/presentation variants driven by external systems. Weapon interaction does not own sprinting or cover logic.

---

## Pose State Data

```text
WeaponPoseState:
  StateId
  PresentationSide / MirrorState
  CanonicalMainHandRole
  CanonicalSupportHandRole
  RequiredContacts
  PreferredContacts
  ContactQualityThresholds
  AimConstraint
  MuzzlePolicy
  ExternalMotionConstraintRequest optional
  InteractionFireReadinessPolicy
  ReloadEntryPolicy
  ReloadExitPolicy
  AnimationProfileId
```

Avoid naming fields as if weapon interaction owns movement speed or body rotation. If the pose needs external motion/body support, output a request/constraint for the external HLS layers.

---

## Pose State Meanings

### RelaxedCarry

Purpose:

```text
weapon is held but not immediately ready to fire visually
```

Typical properties:

```text
muzzle lowered or safe
hands may remain on main/support grips
shoulder contact optional
interaction fire readiness usually not ready
```

### LowReady

Purpose:

```text
weapon is ready but muzzle is lowered below target line
```

Typical properties:

```text
main grip active
support grip preferred for long guns
hand-to-hand support optional for handguns
shoulder contact optional or partial
muzzle safe/down
can transition quickly to NonADSFire, PointAim, or ADS
```

### HighReady

Purpose:

```text
weapon is ready but muzzle is raised or held near line of sight without full ADS
```

Typical properties:

```text
main grip active
support grip active for long guns
hand-to-hand support active for two-handed handgun if used
shoulder contact preferred for stocked weapons
muzzle near target line
can transition quickly to ADS
```

### HipFire / NonADSFire

Purpose:

```text
weapon can support a fire visual/request without strict sight alignment
```

Important:

```text
HipFire is a game-facing label for a non-sight-aligned fire-ready weapon pose.
It does not necessarily mean the weapon is literally held at the hip.
```

Typical properties:

```text
main grip active
support grip active for long guns if required
hand-to-hand support active/preferred for two-handed handguns
shoulder contact optional but often useful for stocked weapons
camera/sight alignment not exact
muzzle follows external aim intent approximately
interaction fire readiness may be ready if contacts and external backend allow
```

### PointAim

Purpose:

```text
weapon is aimed more deliberately than NonADSFire but without strict sight/eye alignment
```

Typical properties:

```text
weapon closer to sight line
muzzle follows external aim intent more tightly
support/hand-to-hand contact quality higher than relaxed poses
shoulder contact preferred for stocked weapons
```

### AimDownSights

Purpose:

```text
weapon sight is visually aligned for precise aiming presentation
```

Typical properties:

```text
main grip active
support grip active for long guns
hand-to-hand support active/preferred for handguns
shoulder contact preferred or required for stocked weapons
cheek contact optional/preferred for stocked ADS presentation
sight-eye alignment quality required for ADS-ready presentation
strict weapon orientation relative to external aim intent
```

ADS does not own camera zoom/FOV, body yaw, turn-in-place, or locomotion speed.

### SprintingWithWeapon

Purpose:

```text
weapon uses a sprint-compatible hold pose because external locomotion state says sprinting is active
```

Typical properties:

```text
muzzle lowered or angled safe
interaction fire readiness usually blocked/not-ready
reload usually blocked or interrupted by interaction policy
hands keep minimum safe hold
ADS not allowed unless project-specific policy says otherwise
```

Weapon interaction does not decide sprint speed or sprint movement.

### InCoverReady / InCoverAim

Purpose:

```text
optional weapon hold variants used when an external cover system says cover interaction is active
```

Typical properties:

```text
weapon pose may be compact or edge-aware
muzzle may be constrained by external cover/obstruction data
ADS/aim readiness depends on external cover state and interaction contacts
```

Weapon interaction does not own cover detection, peeking, exposure, or body placement.

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
LowReady → HipFire / NonADSFire
HipFire / NonADSFire → AimDownSights
AimDownSights → HipFire / NonADSFire
HipFire / NonADSFire → Reloading
AimDownSights → Reloading
Reloading → PreviousPoseState
Reloading → LowReady fallback
HipFire / NonADSFire → SprintingWithWeapon if external locomotion state requires it
SprintingWithWeapon → LowReady when external locomotion state exits sprint
LowReady → Holstered
```

Reload should restore the previous pose if valid:

```text
EntryPose = AimDownSights
Reload starts
Pose = Reloading with EntryPose remembered
Reload completes
Pose = AimDownSights if still valid, otherwise HipFire/NonADSFire or LowReady
```

---

## Interaction Fire Readiness by Pose

```text
RelaxedCarry:
  usually not ready or delayed by raise transition

LowReady:
  not ready or ready only after raise transition

HighReady:
  may be ready with degraded visual stability

HipFire / NonADSFire:
  ready if interaction contacts are stable and external fire backend allows

PointAim:
  ready with better visual stability than NonADSFire

AimDownSights:
  ready when contact quality and sight-eye alignment pass thresholds and external fire backend allows

SprintingWithWeapon:
  usually blocked/not-ready unless project-specific weapon policy allows

Reloading:
  depends on interaction commit state and external fire backend response
```

Weapon pose state does not decide projectile/hit/damage validity.

---

## Muzzle Policy by Pose

```text
RelaxedCarry:
  lowered safe

LowReady:
  lowered but controllable

HighReady:
  near aim line

HipFire / NonADSFire:
  approximate external aim intent

PointAim:
  tightened external aim intent

AimDownSights:
  sight/eye/external aim intent alignment

Reloading:
  inherited from reload policy, often KeepAimApproximate or LoweredSafe

SprintingWithWeapon:
  lowered or angled safe
```

---

## Stability Requirements by Pose

```text
RelaxedCarry:
  one or two hands depending on weapon archetype

LowReady:
  main hand plus optional support or hand-to-hand support

HipFire / NonADSFire:
  main hand plus support grip for long guns or hand-to-hand support for two-handed handguns

AimDownSights:
  main hand + support/hand-to-hand support + shoulder/cheek/sight quality as required by archetype

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
ADS → HipFire/NonADSFire → LowReady → RelaxedCarry
```

---

## Pose State and External Locomotion/Cover

External locomotion and cover systems may affect pose availability.

Examples:

```text
external standing/walking state may allow ADS
external sprint state may request SprintingWithWeapon or LowReady
external falling state may block ADS/reload
external crouch/prone state may restrict some reload variants
external cover state may enable InCoverReady and InCoverAim
```

Weapon interaction consumes these as external state or constraints. It does not own locomotion speed, body orientation, turn-in-place, cover detection, peeking, or exposure.

---

## Networking Rule

Replicate pose state and timestamps, not per-frame IK.

Conceptual replicated fields:

```text
CurrentPoseState
DesiredPoseState
PreviousPoseState optional
PresentationMirrorState / PresentationSide
PoseStartServerTime
PoseBlendDuration
PoseRevision
bHasAimIntent
bIsFiringVisual
```

Clients reconstruct visual pose locally.

---

## Final Formula

```text
Weapon pose state =
  how the weapon is held
  + how it is visually aimed
  + what contacts stabilize it
  + what interaction readiness it reports
  + how it returns after temporary actions
  + what external state may enable/block it.
```
