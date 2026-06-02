---
id: weapon-holding-and-stabilization
title: Weapon Holding and Stabilization
status: draft
version: 26.602.1227
tags: [ weapon, upper-body, ik, procedural-animation, unreal-engine ]
---

# Weapon Holding and Stabilization

## Purpose

This document defines how the Human Locomotion System represents a character holding a weapon before, during, and after weapon interaction tasks.

Weapon interaction is not allowed to start from an abstract empty pose. The character already holds the weapon. Reloading, cycling a bolt, inserting a shell, changing a magazine, or operating a mechanism is a temporary modification of the current weapon hold pose.

The core rule is:

```text
A weapon must always have a valid stabilization state.
```

In normal combat handling this means at least one hand, shoulder, sling, bipod, surface, or other valid support contact must stabilize the weapon while the other hand performs a task.

---

## Main Principle

The system must not hard-code behavior such as:

```text
right hand always holds
left hand always reloads
```

Instead, every frame and every action is resolved from:

```text
Current weapon pose
+ current shoulder side
+ weapon interaction geometry
+ active grip contacts
+ required stabilization
+ hand reachability
+ action access side
→ resolved hand assignment and stabilization plan
```

The same character may use the right hand as a trigger hand in one pose and as a manipulation hand in another pose.

---

## Weapon as Local Space

Every weapon has its own local coordinate space.

```text
WeaponRoot = local origin
```

All grip points, contact points, sockets, and manipulation points are defined relative to this root.

Examples:

```text
MainGripSocket
SupportGripSocket
StockShoulderSocket
MagazineWellSocket
BoltSocket
ChargingHandleSocket
ShellInsertSocket
EjectionPortSocket
MuzzleSocket
```

At runtime the system converts:

```text
Weapon local space → character/world space → hand IK targets
```

---

## Bones, Sockets, and Interaction Data

The weapon asset should separate spatial markers from semantic rules.

```text
Bones / sockets tell where something is.
Data tells what it means and how it can be used.
```

### Use bones for moving weapon parts

Bones are appropriate for parts that physically move relative to the weapon:

```text
bolt
charging handle
pistol slide
pump fore-end
lever
break-action hinge
folding stock
moving dust cover
```

Example:

```text
PumpBone
  └─ PumpGripSocket
```

The hand can follow a socket on a moving bone while the action drives the bone.

### Use sockets or markers for interaction points

Sockets or scene markers are enough for static points:

```text
main grip
support grip
stock shoulder contact
magazine well
magazine pre-insert point
shell insert point
bolt grab point
muzzle
```

A socket should store a full transform, not only a position.

---

## Weapon Contacts

Weapon holding is represented as a set of contacts.

Minimum contact types:

```text
RightHandContact
LeftHandContact
ShoulderContact
```

Extended contact types:

```text
CheekContact
SlingContact
BipodContact
SurfaceContact
BodyClampContact
```

A contact can be active, inactive, transitioning, or temporarily reserved for a manipulation action.

---

## Weapon Hold State

The weapon hold state stores the current relationship between hands, body, and weapon.

```text
WeaponHoldState
  WeaponPose
  ShoulderSide
  RightHandContact
  LeftHandContact
  ShoulderContact
  ActiveGripSockets
  WeaponStability
  PreviousWeaponPose
```

Common weapon poses:

```text
Aimed
LowReady
HipReady
ReloadPose
SprintLowered
ProneSupported
BipodSupported
```

The system should preserve the previous pose when entering an interaction:

```text
Aimed → ReloadPose → Aimed
LowReady → ReloadPose → LowReady
```

If gameplay input changes during the interaction, the return pose can be updated.

---

## Shoulder Side

Shoulder side is critical. It changes which hand is naturally free for top or bottom manipulation.

```text
RightShoulderStance
LeftShoulderStance
```

Example right-shoulder stance:

```text
RightHand may be on MainGrip
LeftHand may be on SupportGrip
RightShoulderContact active
```

Example left-shoulder stance:

```text
LeftHand may be on MainGrip
RightHand may be on SupportGrip
LeftShoulderContact active
```

The system must not assume that the right hand is always the main grip hand.

---

## Hand Roles Are Temporary

The system should avoid permanent labels such as:

```text
right hand = primary
left hand = support
```

Instead, roles are resolved per current pose and current action:

```text
TriggerHand
ForegripHand
ManipulationHand
StabilizingHand
```

These roles may change during a sequence.

For example:

```text
Before action:
  LeftHand = TriggerHand
  RightHand = ForegripHand
  ShoulderSide = Left

During bottom magazine reload:
  RightHand = ManipulationHand
  LeftHand + LeftShoulder = Stabilization
```

---

## Stability Rule

Before a hand releases its current grip, the system must check whether the weapon remains stable.

```text
CanReleaseHand(hand)?
```

The check evaluates:

```text
remaining hand contacts
shoulder contact
weapon weight and length
weapon type
stock presence
sling/bipod/surface support
current pose
required action stability
```

Examples:

```text
Rifle with stock:
  One hand + shoulder can temporarily stabilize the weapon.

Pistol without stock:
  The trigger hand usually cannot release the weapon unless the other hand first grabs the weapon body.

Heavy weapon:
  Releasing one hand may require bipod, sling, or surface support.
```

---

## Stock and Shoulder Contact

A stock changes the interaction model because it provides an additional stabilization contact.

With a stock:

```text
grip hand + shoulder contact
```

may be enough to free the other hand.

Without a stock:

```text
one hand alone may be the only valid support
```

This affects bolt operation, magazine changes, and shell insertion. The solver must not blindly assign the same-side hand if that hand is the only thing holding the weapon.

---

## Weapon Reload Pose

Reloading and other manipulation tasks often require moving the weapon from the current combat pose into a more accessible pose.

Examples:

```text
rifle lowers and rolls slightly outward
pistol lowers closer to the chest
shotgun rotates to expose bottom loading port
bullpup shifts to expose rear magazine well
```

This pose is not a separate animation that ignores the current state. It is a temporary weapon-hold configuration.

```text
CurrentWeaponPose
  ↓
ReloadPose
  ↓
Manipulation
  ↓
ReturnPose
```

Reload poses should support mirroring for left-shoulder and right-shoulder handling.

---

## Access Regions

Every interaction point on the weapon has an access region.

```text
Left
Right
Top
Bottom
Front
Rear
TopLeft
TopRight
BottomLeft
BottomRight
Custom
```

Side access normally prefers the same-side hand:

```text
Right access → prefer RightHand
Left access  → prefer LeftHand
```

Top or bottom access normally depends on shoulder side:

```text
Right shoulder → prefer LeftHand
Left shoulder  → prefer RightHand
```

This is only a preference. Final assignment must also pass the stability check.

---

## Hand Assignment Solver

The Hand Assignment Solver chooses the manipulation hand and stabilization contacts for an action.

Input:

```text
CurrentShoulderSide
CurrentGripState
WeaponHasStock
WeaponLength
WeaponWeight
InteractionPoint.AccessRegion
InteractionPoint.LocalTransform
InteractionPoint.RequiredDirection
ActionType
AvailableContacts
```

Output:

```text
ManipulationHand
StabilizationContacts
RequiredRegrip
RequiredWeaponPoseAdjustment
ReturnGrip
```

Resolution order:

```text
1. Resolve preferred hand from access region.
2. Check if the preferred hand can release its current grip.
3. Check if the weapon remains stabilized.
4. If valid, assign preferred hand.
5. If invalid, test alternate hand.
6. If alternate hand is awkward but possible, add weapon roll/tilt.
7. If neither hand can act directly, insert a regrip sequence.
8. If still invalid, block or request a supported pose.
```

---

## Preferred Hand vs Resolved Hand

The weapon interaction data may define a preferred hand policy, but runtime chooses the resolved hand.

```text
PreferredHand = what is ergonomically intended.
ResolvedHand  = what is currently possible and stable.
```

Example:

```text
Bolt access = Right
PreferredHand = RightHand

But:
  RightHand is the only hand holding a pistol.
  No stock exists.

Resolved behavior:
  keep RightHand on weapon
  use LeftHand if reachable
  or perform a regrip before RightHand operates the bolt
```

---

## Regrip

Regrip is a short transition that changes weapon contacts before an action.

Examples:

```text
LeftHand takes stronger support grip before RightHand operates right-side bolt.
RightHand grabs weapon body before LeftHand releases support.
Weapon is rolled inward before bottom magazine insertion.
```

Regrip is not optional for correctness. It prevents the weapon from visually floating or losing support.

---

## Return State

Every manipulation action must define where the hand returns.

```text
ReturnToMainGrip
ReturnToSupportGrip
StayOnPumpGrip
ReturnToTwoHandPistolSupport
ContinueToNextAction
```

At the end of the full sequence, the weapon must return to a valid hold state:

```text
both hands restored when required
shoulder contact restored when required
weapon pose restored or updated
weapon stability valid
```

---

## Unreal Engine Implementation Notes

Recommended asset structure:

```text
Weapon Skeletal Mesh
  bones for moving parts
  sockets for grip/contact/interaction points

WeaponInteractionProfile DataAsset
  semantic descriptions
  access regions
  hand policies
  required stability
  insert/extract/operate axes
```

The animation pipeline should be:

```text
Runtime solver computes targets
AnimInstance receives targets and state
Control Rig / IK Rig solves hands, arms, spine, shoulders
Weapon component applies procedural part movement
```

The AnimBP should not own the weapon interaction logic. It should apply the already resolved pose targets.

---

## Debug Requirements

Debug visualization should show:

```text
active hand contacts
active shoulder contact
current shoulder side
weapon stability score
preferred hand
resolved hand
access region
regrip requirement
return grip
```

This is required because most bugs in this system are not animation bugs. They are invalid contact-state bugs.

---

## Final Formula

```text
Weapon holding = contacts + pose + shoulder side + stability rules.
```

Weapon manipulation is valid only when it is planned on top of that holding state.

The weapon must never be treated as an unsupported prop while the hands perform procedural actions.
