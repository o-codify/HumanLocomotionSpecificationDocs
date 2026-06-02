---
id: weapon-interaction-boundaries
title: Weapon Interaction Boundaries
status: draft
version: 26.602.1448
tags: [ weapon, interaction, boundaries, ownership, scope ]
---

# Weapon Interaction Boundaries

## Purpose

This document defines the scope boundary of the weapon interaction section.

It exists to prevent the weapon interaction documentation from expanding into unrelated systems such as inventory, damage, camera, locomotion speed, body orientation, AI decision making, or full weapon backend logic.

Weapon interaction may influence those systems, and those systems may influence weapon interaction, but influence does not mean ownership.

---

## Core Rule

```text
Weapon interaction owns the character's physical/procedural interaction with the weapon.
It does not own every system affected by the weapon.
```

Do not decide scope by asking:

```text
What does weapon interaction affect?
```

Decide scope by asking:

```text
What is itself weapon interaction?
```

---

## Owned By Weapon Interaction

Weapon interaction owns:

```text
how hands hold the weapon
how hands release the weapon
how hands reach weapon interaction points
how hands grip magazines, shells, bolts, slides, pumps, levers, and similar manipulators
how weapon hold contacts remain stable
how support contact is preserved or temporarily broken
how a manipulation hand is selected
how reload/manipulation temporarily disrupts a hold pose
how the hold pose is restored after manipulation
how authored weapon sockets/axes/contact points are interpreted
how interaction phases are represented
how procedural hand/object/weapon targets are generated
how interaction state is exposed to animation
how interaction state is reconstructed remotely
```

---

## Owned Pose Concepts

Weapon interaction may own weapon-facing pose concepts:

```text
RelaxedCarry
LowReady
HighReady
HipFire
PointAim
AimDownSights
Reloading
ManipulatingMechanism
SprintingWithWeapon as a weapon hold pose only
```

These pose states describe the weapon/hand/contact relationship.

They do not own locomotion speed, body yaw, foot placement, camera behavior, or projectile behavior.

---

## Owned Animation Concepts

Weapon interaction owns animation-facing outputs:

```text
hand IK targets
elbow pole requests
finger grip alpha or grip pose id
weapon pose offset request
object visual attachment state
moving part follow target
interaction phase
step alpha
reload visual state
```

Weapon interaction does not own the entire animation stack. It provides weapon-specific interaction targets to the broader locomotion/animation system.

---

## Owned Networking Concepts

Weapon interaction owns network state for interaction reconstruction:

```text
current weapon pose state
current interaction phase
reload sequence id
reload step id
step start server time
step duration
resolved manipulation hand
object visual state
mechanical interaction state needed for visual/interaction correctness
revision counters for interaction correction
```

Weapon interaction does not own the full networking model of inventory, damage, projectile authority, AI, or camera.

---

## External Systems Weapon Interaction May Read

Weapon interaction may read state from external systems as inputs:

```text
locomotion state
body orientation state
equipped weapon reference
external inventory/ammo availability result
external fire permission result
camera/controller aim direction
environment obstruction query result
cover state
AI intent
```

Reading these inputs does not make those systems part of weapon interaction.

---

## External Systems Weapon Interaction May Request

Weapon interaction may output requests or constraints to external systems:

```text
request stable upper-body support
request a weapon pose state
request hand/arm animation targets
request muzzle/weapon clearance query
request fire permission evaluation
request inventory item reservation or confirmation
request body orientation support for ADS
request locomotion system to know weapon pose state
```

A request is not ownership.

Example:

```text
Weapon interaction may say: CurrentWeaponPoseState = AimDownSights.
The locomotion/body-orientation system decides whether to rotate pelvis, spine, feet, or start turn-in-place.
```

---

## Explicitly Not Owned

Weapon interaction does not own:

```text
locomotion speed calculation
acceleration/deceleration rules
turn-in-place implementation
body yaw / pelvis yaw ownership
foot placement
camera system
camera zoom/FOV
full first-person camera behavior
damage model
ballistics
hitscan/projectile simulation
ammo economy
inventory UI
loot system
weapon persistence/save-load
AI decision making
cover system as a whole
anti-cheat system as a whole
performance budgeting for the whole game
```

These systems may consume weapon interaction state, but they must be documented in their own sections.

---

## Boundary Examples

### Example 1: ADS affects speed

Correct boundary:

```text
Weapon interaction owns:
  ADS weapon/hand/contact pose state.

Locomotion owns:
  movement speed while ADS.
```

Weapon interaction may expose:

```text
CurrentWeaponPoseState = AimDownSights
RequiredStability = High
```

It must not define the final movement speed formula.

---

### Example 2: ADS affects body turn

Correct boundary:

```text
Weapon interaction owns:
  weapon wants aim-aligned hold pose.

Body orientation solver owns:
  pelvis rotation, spine twist, feet turn-in-place, body yaw limits.
```

Weapon interaction may expose:

```text
AimConstraintRequest
WeaponPoseState
ShoulderSide
```

It must not own turn-in-place.

---

### Example 3: Reload consumes ammo

Correct boundary:

```text
Weapon interaction owns:
  hand reaches magazine, extracts object, inserts object, commits MagazineLocked interaction point.

Inventory/ammo system owns:
  whether ammo exists, which magazine item is consumed, ammo count, stack rules.
```

Weapon interaction may ask:

```text
CanProvideReloadObject?
ReserveReloadObject?
ConfirmReloadObjectInserted?
```

It must not implement full inventory economy.

---

### Example 4: Fire while reload is active

Correct boundary:

```text
Weapon interaction owns:
  current reload phase, hand occupation, hold stability, whether the interaction pose can support fire.

Weapon/fire backend owns:
  shot validation, projectile/hitscan, damage, ammo consumption.
```

Weapon interaction may provide:

```text
InteractionFireReadiness
HandsOccupied
CurrentReloadCommitState
```

It must not define damage or projectile simulation.

---

### Example 5: Weapon hits a wall

Correct boundary:

```text
Environment/collision system owns:
  traces, collision detection, blocked volume queries.

Weapon interaction owns:
  response of the weapon hold/manipulation pose to an obstruction result.
```

Weapon interaction may consume:

```text
WeaponObstructionResult
```

and respond with:

```text
lower weapon
block ADS pose
choose compact hold pose
adjust hand path if possible
```

It must not own the whole collision system.

---

## Clean External Interfaces

Weapon interaction should communicate through small interfaces instead of importing whole systems.

### External Inventory Interface

```text
CanProvideReloadObject(request) -> yes/no + object descriptor
ReserveReloadObject(request) -> reservation id
ConfirmReloadObjectUsed(reservation id)
CancelReloadObjectReservation(reservation id)
```

The implementation of inventory remains external.

### External Fire Interface

```text
CanFireWithInteractionState(state) -> yes/no + reason
NotifyInteractionFireStarted(event)
NotifyInteractionFireEnded(event)
```

The implementation of firing, ammo, projectile, and damage remains external.

### External Obstruction Interface

```text
QueryWeaponObstruction(weapon pose/query shape) -> obstruction result
```

The implementation of traces and collision remains external.

### External Body/Locomotion Interface

```text
GetCurrentLocomotionState()
GetCurrentBodyOrientationState()
SubmitWeaponPoseConstraint(constraint)
```

The implementation of locomotion speed, pelvis rotation, feet, and turn-in-place remains external.

---

## What To Document Here

Document here:

```text
contacts
hand roles
weapon interaction points
weapon hold pose states
reload/manipulation phases
object visual attachment for interaction
animation targets
Control Rig inputs
interaction networking state
editor validation for weapon interaction data
```

Do not document here:

```text
full inventory design
full fire backend
full camera design
full locomotion solver
full body orientation solver
full AI combat behavior
full cover system
full damage system
```

---

## Final Formula

```text
Weapon interaction =
  physical/procedural relationship between character and weapon
  + hand/contact/manipulation states
  + weapon-facing pose states
  + animation targets
  + interaction reconstruction state.

Weapon interaction ≠ every system affected by holding a weapon.
```
