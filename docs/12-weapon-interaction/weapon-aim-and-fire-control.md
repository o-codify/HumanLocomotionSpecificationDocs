---
id: weapon-aim-and-fire-control
title: Weapon Aim and Fire Control
status: draft
version: 26.602.1554
tags: [ weapon, aiming, fire-control, ads, hip-fire, networking ]
---

# Weapon Aim and Fire Control

## Purpose

This document defines the interaction-facing aim and fire readiness model for weapons held in hands.

It covers only the parts of aiming/firing that are needed for weapon interaction:

```text
aim source as an external input
aim target as an external input
muzzle/sight alignment as a weapon pose constraint
HipFire / NonADSFire as a weapon/hand/contact pose
point aim as a weapon/hand/contact pose
ADS as a weapon/hand/contact pose
interaction fire readiness from interaction state
local visual fire feedback requests
remote fire visualization requests
interaction with reload/manipulation state
```

This document does not own projectile simulation, hit validation, damage, ammo economy, inventory, camera implementation, fire-rate balance, or the full fire backend.

Pose states are defined in [Weapon Pose State Model](./weapon-pose-state-model.md). Scope boundaries are defined in [Weapon Interaction Boundaries](./weapon-interaction-boundaries.md).

---

## Core Principle

```text
Aiming is a weapon pose constraint.
Interaction fire readiness is an interaction-state output.
The actual fire backend decides whether a shot is executed and what it does.
```

Animation visualizes aim/fire interaction state. It does not decide projectile, damage, hit, ammo, or backend fire validity.

---

## External Aim Inputs

Weapon interaction may receive aim intent from external systems.

Common external aim sources:

```text
CameraCenter
EyeLine
WeaponSight
MuzzleForward
ControllerRotation
AIIntentDirection
ScriptedTarget
```

Weapon interaction consumes the aim intent and attempts to maintain a plausible weapon/hand/contact pose for that intent.

It does not own camera behavior or AI decision making.

---

## Aim Target For Interaction

For interaction, aim target can be represented as:

```text
world point
world direction
actor/component target reference
```

Recommended interaction-facing runtime state:

```text
AimOrigin
AimDirection
AimTargetPoint optional
AimSource
AimMode
AimAlpha
```

Do not store full projectile prediction or damage result here.

---

## Aim Modes

```text
None
Relaxed
LowReady
HipFire / NonADSFire
PointAim
AimDownSights
CoverAim as external cover-facing pose input if cover system exists
```

Aim mode usually corresponds to weapon pose state, but may be represented separately for blending.

---

## HipFire / NonADSFire

HipFire is a game-facing label for a non-sight-aligned fire-ready weapon pose.

It does not necessarily mean the weapon is literally held at the hip.

Interaction rules:

```text
weapon muzzle follows external aim direction approximately
sight alignment is not strict
main hand remains on grip
support grip remains active for long guns unless released by another interaction
hand-to-hand support may be active for two-handed handguns
shoulder contact may be partial or absent depending on weapon
weapon remains visually plausible relative to external aim direction
```

Weapon interaction may output NonADSFire readiness. The external fire backend decides final shot execution.

---

## Aim Down Sights

ADS is a stricter weapon/hand/contact pose.

Interaction rules:

```text
weapon sight aligns toward external aim intent
muzzle direction follows aim direction more tightly than NonADSFire
support grip contact is preferred or required for long guns
hand-to-hand support contact is preferred for two-handed handguns
shoulder contact is preferred or required for stocked weapons
cheek contact may be preferred for stocked ADS presentation
sight-eye alignment quality must pass ADS-ready threshold
ADS entry/exit is a transition, not an instant snap
```

ADS does not own camera zoom, FOV, body yaw, turn-in-place, or locomotion speed.

---

## Point Aim

Point aim is between NonADSFire and ADS as a weapon interaction pose.

Interaction rules:

```text
weapon is raised deliberately
sight alignment is not strict
muzzle follows aim direction more closely than NonADSFire
contacts are more constrained than relaxed/low-ready poses
support/hand-to-hand contact quality is higher than NonADSFire
```

---

## Interaction Fire Readiness

Weapon interaction may produce an interaction-facing fire readiness result.

It should consider only interaction-owned or interaction-readable state:

```text
weapon pose state
aim mode
mechanical interaction state exposed to interaction
reload/manipulation phase
hand occupation
hold stability
contact quality thresholds
hand-to-hand support quality for two-handed handguns
shoulder/cheek/sight quality for stocked ADS
external fire backend policy result if available
```

Examples:

```text
ADS + stable long-gun hold + sight-eye alignment + external fire backend allows fire → interaction-ready
Two-handed handgun + hand-to-hand support stable + external fire backend allows fire → interaction-ready
NonADSFire + acceptable stability + external fire backend allows fire → interaction-ready
Reloading before required interaction commit → interaction-not-ready
Reloading after required commit + policy allows → interaction-ready or recovery-needed
```

Weapon interaction must not implement ammo economy, damage, projectile, or full fire-rate logic.

---

## Fire Readiness Request

Conceptual interaction-facing request to an external fire backend:

```text
FireReadinessRequest:
  WeaponReference
  PoseState
  AimMode
  AimOrigin
  AimDirection
  InteractionPhase
  HandsOccupied
  HoldStability
  ContactQualitySummary
  InteractionCommitState
```

External backend may answer:

```text
CanFire
BlockedReason
FirePolicyResult
```

Weapon interaction can use that answer to choose a visual response, recovery, or blocking pose.

---

## Fire Visual Event

When an external fire backend confirms or predicts a fire event, weapon interaction may visualize it.

Interaction-facing event:

```text
FireVisualEvent:
  FireSequenceId
  FireVisualTime
  AimMode
  PoseState
  MuzzleTransform
  RecoilVisualSeed optional
```

Weapon interaction may use this for:

```text
weapon pose impulse
hand recoil offset
muzzle flash attachment point request
remote fire pose reconstruction
```

Actual muzzle flash spawning, sound, projectile, hit, and damage may belong to other systems.

---

## Recoil and Sway As Interaction Disturbance

This document treats recoil/sway only as pose disturbance inputs/outputs.

```text
Recoil disturbance:
  temporary offset applied to weapon/hands/pose after a fire visual event.

Sway disturbance:
  continuous low-frequency pose variation coming from external stance/movement/breathing systems.
```

Weapon interaction may expose or consume:

```text
WeaponPoseDisturbance
HandTargetDisturbance
AimAlignmentDisturbance
```

It must not define final weapon damage, fire balance, or shooter spread formula.

---

## Muzzle vs External Aim Direction

Weapon interaction can measure visual alignment between muzzle/sight and external aim direction.

It may output:

```text
SightAlignmentErrorDegrees
MuzzleAlignmentErrorDegrees
SightEyeAlignmentQuality optional
bAimPoseVisuallyAligned
```

The fire backend decides how much this matters for shot execution.

---

## Fire Networking For Interaction Visualization

Replicate or receive fire events/state needed for visual reconstruction, not per-frame IK.

Interaction-facing fields may include:

```text
bIsFiringVisual
FireSequenceId
LastFireVisualServerTime
AimMode
PoseState
RecoilVisualSeed optional
```

Remote clients reconstruct:

```text
weapon fire pose impulse
hand recoil offset
muzzle attachment point timing
brief grip/contact disturbance
```

Projectile, damage, hit, ammo, and authoritative backend fire state remain external.

---

## Client Prediction Boundary

Owning client may predict interaction visuals:

```text
weapon pose impulse
hand recoil offset
brief muzzle/weapon visual response request
camera recoil request to external camera system
```

Owning client must not authoritatively decide:

```text
ammo consumption
hit confirmation
damage
projectile authority
fire backend acceptance
```

---

## Interaction With Reload

Reload can affect interaction fire readiness.

Interaction policies:

```text
CannotRequestFireDuringReloadInteraction
CanRequestFireAfterInteractionCommit
CanRequestFireWithRecoveryAfterCommit
CanRequestFireOnlyWhenStableHoldRestored
```

If a fire request happens during reload:

```text
weapon interaction reports current phase, hand occupation, contact quality, and commit state
external fire backend decides whether a fire event is allowed
weapon interaction performs visual recovery or fire-pose transition if needed
```

---

## Debug Requirements

Debug should show interaction-facing data:

```text
pose state
aim mode
aim origin
aim direction
muzzle direction
sight alignment error
sight-eye alignment quality
contact quality summary
interaction fire readiness
blocked reason from interaction or external backend
mechanical interaction state
reload/manipulation phase
fire visual sequence id
prediction id
```

---

## Final Formula

```text
Interaction-facing aim/fire =
  external aim intent
  + weapon pose state
  + weapon/muzzle/sight alignment
  + hand/contact stability
  + archetype-specific support contacts
  + reload/manipulation state
  + interaction fire readiness output
  + local/remote visual fire response.

It is not the full weapon fire backend.
```
