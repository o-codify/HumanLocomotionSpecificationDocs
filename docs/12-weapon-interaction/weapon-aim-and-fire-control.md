---
id: weapon-aim-and-fire-control
title: Weapon Aim and Fire Control
status: draft
version: 26.602.1430
tags: [ weapon, aiming, fire-control, ads, hip-fire, networking ]
---

# Weapon Aim and Fire Control

## Purpose

This document defines the engine-agnostic aim and fire control model for weapons held in hands.

It covers:

```text
aim source
aim target
muzzle direction
sight alignment
hip fire
point aim
aim down sights
fire permission
recoil/sway/spread concepts
server-authoritative firing
client prediction
remote fire visualization
```

Pose states are defined in [Weapon Pose State Model](./weapon-pose-state-model.md). Networking principles are defined in [Weapon Interaction Networking](./weapon-networking.md).

---

## Core Principle

```text
Aiming is a stateful weapon pose constraint.
Firing is a server-authoritative gameplay action.
Animation visualizes aim and fire but does not decide whether a shot is valid.
```

---

## Aim Sources

Common aim sources:

```text
CameraCenter
EyeLine
WeaponSight
MuzzleForward
ControllerRotation
AIThreatTarget
ScriptedTarget
```

Player-controlled weapons usually use camera/controller intent as the desired aim direction, then the weapon pose system attempts to align the weapon appropriately.

---

## Aim Target

Aim target can be represented as:

```text
world point
world direction
actor/component target
predicted projectile path target
```

Recommended runtime state:

```text
AimOrigin
AimDirection
AimTargetPoint optional
AimSource
AimMode
AimAlpha
```

---

## Aim Modes

```text
None
Relaxed
LowReady
HipFire
PointAim
AimDownSights
CoverAim
```

Aim mode should usually correspond to weapon pose state, but may be represented separately for blending.

---

## Hip Fire

Hip fire allows firing without strict sight alignment.

Rules:

```text
weapon muzzle follows aim direction approximately
camera does not need to align with weapon sight
spread/recoil/sway usually worse than ADS
movement freedom is higher
shoulder contact may be partial or absent depending on weapon
```

Hip fire should not require the weapon to visually point at a completely different direction from the shot. Visual muzzle direction should remain plausibly close to gameplay fire direction.

---

## Aim Down Sights

ADS requires stricter alignment.

Rules:

```text
weapon sight aligns to camera/eye line
muzzle direction tightly follows aim direction
shoulder contact preferred for long guns
support hand contact preferred
movement freedom reduced
spread/sway/recoil policy improves or changes
```

ADS entry should be a transition, not an instant snap.

---

## Point Aim

Point aim is between hip fire and ADS.

Rules:

```text
weapon is raised and aimed deliberately
sight alignment is not strict
muzzle follows aim direction more closely than hip fire
movement freedom is higher than ADS
precision is better than hip fire but worse than ADS
```

---

## Fire Permission

Fire permission must consider:

```text
weapon pose state
aim mode
mechanical state
reload state
stability state
movement state
weapon policy
server cooldown / fire rate
ammo/chamber state
```

Examples:

```text
ADS + chambered round + stable hold → fire allowed
HipFire + chambered round + acceptable stability → fire allowed
Reloading before MagazineLocked → fire blocked
Reloading after commit with CanFireAfterCommit policy → fire allowed if stable enough
SprintingWithWeapon → fire blocked unless weapon policy allows
```

---

## Fire Request

Conceptual fire request:

```text
FireRequest:
  WeaponId
  FireMode
  AimOrigin
  AimDirection
  ClientFireTime
  ClientPredictionId
  PoseState
  AimMode
```

Server validates:

```text
weapon equipped
fire cooldown
ammo/chamber state
mechanical state
reload/fire policy
pose state allows fire
aim direction within allowed tolerance
```

---

## Recoil, Sway, and Spread

This document defines concepts, not final weapon balance values.

```text
Recoil:
  short impulse caused by firing.

Sway:
  continuous low-frequency aim movement from breathing/movement/stance.

Spread:
  gameplay dispersion applied to shot direction.
```

Pose state affects them:

```text
HipFire → higher spread / less stable visual aim
PointAim → medium spread / medium stability
ADS → lower spread / stricter sight alignment
Sprinting → fire blocked or very high penalty
```

Recoil and sway should be visualized locally, but server must own gameplay shot validation and authoritative hit/projectile state.

---

## Muzzle vs Camera Direction

Gameplay must define which direction is authoritative for shot validation.

Common policies:

```text
CameraAuthoritativeWithMuzzleValidation
MuzzleAuthoritative
HybridCameraAimMuzzleSpawn
AIWeaponMuzzleAuthoritative
```

Recommended player policy:

```text
Camera gives desired aim direction.
Muzzle/sight must be within allowed angular tolerance.
Projectile or trace starts according to weapon policy.
Server validates pose and mechanical state.
```

This avoids impossible shots while still keeping responsive player aiming.

---

## Fire Networking

Replicate fire events/state, not per-frame weapon IK.

Conceptual replicated fields:

```text
bIsFiring
FireSequenceId
LastFireServerTime
FireMode
AimMode
CompressedAimDirection optional
RecoilSeed optional
```

Remote clients reconstruct:

```text
muzzle flash
sound
recoil animation
weapon fire pose
projectile/tracer visuals
```

Gameplay damage/projectiles remain server-authoritative.

---

## Client Prediction

Owning client may predict:

```text
muzzle flash
fire sound
recoil animation
camera recoil
local projectile/tracer cosmetic
```

Owning client must not authoritatively decide:

```text
ammo consumption
hit confirmation
damage
projectile authority
mechanical state
```

---

## Interaction With Reload

Reload can affect fire permission.

Policies:

```text
CannotFireDuringReload
CanFireAfterCommit
CanFireWithPenaltyAfterCommit
CanFireOnlyWhenStableHoldRestored
```

If fire interrupts reload:

```text
server validates policy
server commits or cancels reload according to recovery policy
animation executes recovery or fire transition
```

---

## Debug Requirements

Debug should show:

```text
pose state
aim mode
aim origin
aim direction
muzzle direction
sight alignment error
fire permission result
blocked reason
mechanical state
reload state
fire sequence id
server fire time
prediction id
```

---

## Final Formula

```text
Aim/fire control =
  pose state
  + aim source/target
  + weapon/muzzle alignment
  + mechanical state
  + fire permission policy
  + server-authoritative shot validation
  + local visual prediction.
```
