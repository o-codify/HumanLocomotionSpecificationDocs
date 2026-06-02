---
id: weapon-holding-and-stabilization
title: Weapon Holding and Stabilization
status: draft
version: 26.602.1236
tags: [ weapon, upper-body, ik, procedural-animation, unreal-engine, multiplayer ]
---

# Weapon Holding and Stabilization

## Purpose

This document defines how the Human Locomotion System represents a character holding a weapon before, during, and after weapon interaction tasks.

Weapon interaction is not allowed to start from an abstract empty pose. The character already holds the weapon. Reloading, cycling a bolt, inserting a shell, changing a magazine, or operating a mechanism is a temporary modification of the current weapon hold pose.

The core rule is:

```text
A weapon must always have a valid stabilization state.
```

In normal combat handling this means at least one hand, shoulder, sling, bipod, surface, or other valid support contact must stabilize the weapon while another hand performs a task.

---

## Main Principle

The system must not hard-code behavior such as:

```text
right hand always holds
left hand always reloads
```

Instead, every action is resolved from:

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

## Runtime Input State

Weapon holding must be solved from explicit runtime state, not from animation assumptions.

### Character state

```cpp
struct FCharacterWeaponInteractionState
{
    ECharacterStance Stance;          // Stand, Crouch, Prone
    EMovementState Movement;          // Idle, Walk, Run, Sprint, Falling
    EWeaponPose WeaponPose;           // Aimed, LowReady, HipReady, ReloadPose
    EShoulderSide ShoulderSide;       // Right, Left
    bool bIsAiming;
    bool bIsSprinting;
    bool bIsInCover;
};
```

### Hand state

```cpp
struct FHandInteractionState
{
    EHand Hand;
    EHandRole CurrentRole;            // MainGrip, SupportGrip, Free, Manipulating
    FName AttachedSocket;
    TObjectPtr<UObject> HeldObject;
    bool bCanRelease;
    bool bIsBusy;
};
```

### Weapon hold state

```cpp
struct FWeaponHoldState
{
    FName MainGripSocket;
    FName SupportGripSocket;
    FName ActiveShoulderSocket;

    EShoulderSide ShoulderSide;

    bool bRightHandContact;
    bool bLeftHandContact;
    bool bShoulderContact;
    bool bSlingContact;
    bool bBipodContact;
    bool bSurfaceContact;

    float StabilityScore;
    EWeaponPose PreviousWeaponPose;
};
```

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

## Muzzle Control During Manipulation

The system must define how much the weapon keeps its aim during manipulation.

Possible policies:

```text
KeepAimApproximate
LoweredSafe
FreeDuringManipulation
LockedToReloadPose
```

For normal gameplay, the recommended default is:

```text
KeepAimApproximate + reload pose offset
```

The muzzle stays generally aligned with the character/camera direction but is allowed to lower, roll, or offset enough to expose the interaction point.

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

This is only a preference. Final assignment must also pass stability and reachability checks.

---

## Cost-Based Hand Assignment Solver

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
Character stance and movement state
```

Output:

```text
ManipulationHand
StabilizationContacts
RequiredRegrip
RequiredWeaponPoseAdjustment
ReturnGrip
ReachabilityCost
RejectedHandReasons
```

The solver should evaluate both hands with a cost model:

```text
Cost =
  side mismatch penalty
+ reach distance penalty
+ weapon instability penalty
+ regrip penalty
+ torso twist penalty
+ current hand busy penalty
+ stance constraint penalty
```

The selected hand is the valid candidate with the lowest cost. If no candidate is valid, the system must insert a regrip, modify weapon pose, or block the action.

---

## Reachability Result

The system does not need full biomechanical simulation for MVP, but it needs a simple reachability test.

```cpp
struct FReachabilityResult
{
    bool bReachable;
    float Cost;
    bool bRequiresWeaponRoll;
    bool bRequiresTorsoTwist;
    bool bRequiresRegrip;
};
```

The reachability test should consider:

```text
hand-to-target distance
max arm extension
shoulder twist
current stance
weapon roll/tilt allowance
simple body/weapon collision avoidance
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

## Stance Constraints

The hold solver must consider stance-specific restrictions.

```text
Standing:
  Most reload poses are available.

Crouch:
  Weapon lowering and body slot access may be reduced.

Prone:
  Bottom magazine insertion may collide with ground.
  Large magazines may require weapon roll.
  Some reload variants may be blocked.

Sprint:
  Interaction is usually blocked or converted into a lowered reload pose.

Falling / climbing:
  Most reload tasks should be blocked or interrupted.
```

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

Recommended runtime components:

```text
UProceduralWeaponManipulationComponent   // on character
UWeaponInteractionComponent              // on weapon
UWeaponReloadComponent                   // on weapon or owning equipment component
```

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

## Multiplayer Rule

Weapon holding and contact state can influence gameplay, but the server must not evaluate full IK.

Server authority:

```text
weapon equipped state
active weapon pose category
whether weapon is in a valid gameplay hold state
reload/fire/block permissions
```

Client visual authority:

```text
hand IK targets
spine offsets
finger pose
minor contact blending
weapon pose interpolation
```

For networking, replicate state and time, not per-frame hand transforms.

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
reachability cost
rejected hand reasons
stance constraint result
```

This is required because most bugs in this system are not animation bugs. They are invalid contact-state bugs.

---

## Final Formula

```text
Weapon holding = contacts + pose + shoulder side + stability rules + reachability.
```

Weapon manipulation is valid only when it is planned on top of that holding state.

The weapon must never be treated as an unsupported prop while the hands perform procedural actions.
