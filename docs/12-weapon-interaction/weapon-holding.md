---
id: weapon-holding-and-stabilization
title: Weapon Holding and Stabilization
status: draft
version: 26.602.2053
tags: [ weapon, upper-body, ik, procedural-animation, unreal-engine, multiplayer ]
---

# Weapon Holding and Stabilization

## Purpose

This document defines how the Human Locomotion System represents a character holding a weapon before, during, and after weapon interaction tasks.

Weapon interaction is not allowed to start from an abstract empty pose. The character already holds the weapon. Reloading, cycling a bolt, inserting a shell, changing a magazine, drawing, holstering, or operating a mechanism is a temporary modification of the current weapon hold pose.

The core rule is:

```text
A weapon must always have a valid stabilization state.
```

In normal combat handling this means at least one valid hand, shoulder, sling, bipod, surface, hand-to-hand, or other support contact must stabilize the weapon while another hand performs a task.

---

## Main Principle

The system must not hard-code behavior such as:

```text
right hand always holds
left hand always reloads
```

Instead, every action is resolved from canonical authored interaction data:

```text
Current weapon pose
+ canonical hand roles
+ global presentation mirror state
+ weapon interaction geometry
+ active grip/contact quality
+ required stabilization
+ hand reachability
+ action access side
→ resolved canonical hand assignment and stabilization plan
→ optional mirrored presentation output
```

Important:

```text
Global left/right shoulder presentation mirroring is not a real gameplay swap of main/support hand roles.
```

The canonical authored interaction may use right-hand main grip and left-hand support grip, while the final character presentation is globally mirrored by the animation/presentation layer.

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
HandToHandSupportReference optional
StockShoulderSocket
CheekReferenceSocket optional
SightReferenceSocket optional
MagazineWellSocket
BoltSocket
ChargingHandleSocket
ShellInsertSocket
EjectionPortSocket
MuzzleSocket
```

At runtime the system converts:

```text
Weapon local space → canonical character/world space → optional mirrored presentation space → hand IK targets
```

---

## Weapon Hold Archetypes

Different weapons use different contact models.

### One-Handed Pistol

```text
MainHandGrip: required
SupportGripSocket: not required
HandToHandSupportContact: optional
ShoulderContact: none
CheekContact: none
```

The weapon is mainly stabilized by the main hand unless a two-handed pose is requested.

### Two-Handed Handgun

```text
MainHandGrip: required
HandToHandSupportContact: preferred or required by pose
SupportHand may support the main hand and/or weapon frame
ShoulderContact: none
CheekContact: none
```

For handgun handling, the support hand often does not simply attach to a separate weapon socket. It may wrap around or brace the main hand. The system should support:

```text
SupportHandToMainHandContact
SupportHandToWeaponFrameContact optional
CombinedTwoHandGripPose
```

### Stocked Rifle / Long Gun

```text
MainHandGrip: required
SupportGrip: preferred or required
ShoulderContact: preferred or required by pose
CheekContact: optional/preferred for ADS presentation
SightEyeAlignmentQuality: required for ADS-ready presentation
```

A stock changes the interaction model because it provides an additional support contact.

### Pump / Foregrip Weapon

```text
MainHandGrip: required
SupportGrip or PumpGrip: required when operating pump/foregrip
MovingPartContactQuality: required during manipulation phase
ShoulderContact: weapon/pose dependent
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
hand-to-hand support reference
stock shoulder contact
cheek/sight reference for ADS presentation
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
    EWeaponPose WeaponPose;           // LowReady, HipFire, ADS, Reloading, etc.
    EPresentationSide PresentationSide; // RightShoulderView, LeftShoulderView
    bool bIsAiming;
    bool bIsSprintingFromExternalLocomotion;
    bool bIsInCoverFromExternalCoverSystem;
};
```

`PresentationSide` may drive global mirroring. It is not a gameplay hand-role swap.

### Hand state

```cpp
struct FHandInteractionState
{
    EHand Hand;
    EHandRole CurrentRole;            // MainGrip, SupportGrip, HandToHandSupport, Free, Manipulating
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
    FName CheekReferenceSocket;
    FName SightReferenceSocket;

    EPresentationSide PresentationSide;

    bool bMainGripContact;
    bool bSupportGripContact;
    bool bHandToHandSupportContact;
    bool bShoulderContact;
    bool bCheekContact;
    bool bSlingContact;
    bool bBipodContact;
    bool bSurfaceContact;

    float StabilityScore;
    float SightEyeAlignmentQuality;
    EWeaponPose PreviousWeaponPose;
};
```

---

## Weapon Contacts

Weapon holding is represented as a set of contacts.

Minimum contact types:

```text
MainGripContact
SupportGripContact or HandToHandSupportContact depending on archetype
```

Extended contact types:

```text
ShoulderContact
CheekContact
SightEyeAlignment
SlingContact
BipodContact
SurfaceContact
BodyClampContact
MovingPartContact
ObjectContact
```

A contact can be active, inactive, transitioning, partial, slipping, recovering, blocked, lost, or temporarily reserved for a manipulation action.

---

## Presentation Side And Mirroring

Weapon interaction does not implement a true left-handed/right-handed gameplay model here.

The authored base interaction is canonical. A global presentation mirror may display the whole character/weapon stance on the opposite side.

Correct model:

```text
canonical interaction plan
+ PresentationSide / MirrorState
→ presented pose
```

Incorrect model:

```text
duplicate left-handed reload plan by default
runtime hand-role swap as a separate gameplay system
```

If a weapon mesh/control layout is not mirrored, weapon-side interaction points remain the authored truth. Do not invent mirrored bolt handles, ejection ports, or controls that do not exist.

---

## Hand Roles Are Interaction Roles

The system should avoid permanent labels such as:

```text
right hand = primary
left hand = support
```

Instead, roles are resolved in canonical interaction space:

```text
MainGripHand
SupportGripHand
HandToHandSupportHand
ManipulationHand
StabilizingHand
```

Presentation mirroring may visually move the whole pose to the other side without creating a different gameplay hand-role model.

---

## Stability Rule

Before a hand releases its current grip, the system must check whether the weapon remains stable.

```text
CanReleaseContact(contact)?
```

The check evaluates:

```text
remaining hand contacts
hand-to-hand support contact for handgun poses
shoulder contact
cheek/sight alignment if ADS presentation requires it
weapon size and length
weapon archetype
stock presence
sling/bipod/surface support
current pose
required action stability
contact quality values
```

Examples:

```text
Rifle with stock:
  Main grip + shoulder contact may temporarily stabilize the weapon.

Two-handed handgun:
  Support hand may stabilize by bracing the main hand, not by occupying a separate support socket.

Pistol without stock:
  The main hand usually cannot release the weapon unless another valid support or transfer state is explicitly authored.

Heavy weapon:
  Releasing one hand may require bipod, sling, or surface support.
```

---

## Stock, Shoulder, Cheek, and Sight Contacts

A stock changes the interaction model because it provides an additional stabilization contact.

With a stock:

```text
main grip + shoulder contact
```

may be enough to free the other hand.

For ADS presentation on stocked weapons, the system may also evaluate:

```text
CheekContactQuality
SightEyeAlignmentQuality
```

These are presentation/interaction qualities, not camera ownership.

They help decide:

```text
is ADS visually settled?
can ADS-ready interaction state be reported?
should weapon/hand targets continue settling?
```

Without a stock:

```text
one hand alone may be the only valid support
```

This affects bolt operation, magazine changes, shell insertion, and mechanism manipulation. The solver must not blindly release the only stabilizing contact.

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

Reload poses should support global presentation mirroring through the mirroring system, not through separate duplicated left/right gameplay plans by default.

---

## Muzzle Control During Manipulation

The system must define how much the weapon keeps its visual aim relationship during manipulation.

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

The muzzle stays generally aligned with external aim intent but is allowed to lower, roll, or offset enough to expose the interaction point.

---

## Access Regions

Every interaction point on the weapon has an access region.

```text
CanonicalLeft
CanonicalRight
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

Access regions are authored in canonical weapon interaction space. Presentation mirroring transforms final visuals; it should not automatically rewrite semantic access ids into a separate gameplay plan.

Final assignment must pass stability, contact quality, and reachability checks.

---

## Cost-Based Hand Assignment Solver

The Hand Assignment Solver chooses the manipulation hand and stabilization contacts for an action in canonical interaction space.

Input:

```text
Canonical hand roles
PresentationMirrorState
CurrentGripState
WeaponHasStock
WeaponArchetype
WeaponLength
WeaponWeightClass
InteractionPoint.AccessRegion
InteractionPoint.LocalTransform
InteractionPoint.RequiredDirection
ActionType
AvailableContacts
ContactQualityState
Character stance and external movement state
```

Output:

```text
ManipulationHand
StabilizationContacts
RequiredPoseAdjustment
ReturnContact
ReachabilityCost
RejectedHandReasons
```

The solver should evaluate valid canonical candidates with a cost model:

```text
Cost =
  access mismatch penalty
+ reach distance penalty
+ weapon instability penalty
+ contact break penalty
+ pose adjustment penalty
+ current hand busy penalty
+ stance constraint penalty
+ mirrored presentation risk penalty if applicable
```

The selected plan is the valid candidate with the lowest cost. If no candidate is valid, the system must modify weapon pose, recover contacts, or block the action.

---

## Reachability Result

The system does not need full biomechanical simulation for MVP, but it needs more than a simple distance check.

```cpp
struct FReachabilityResult
{
    bool bReachable;
    float Cost;
    bool bRequiresWeaponRoll;
    bool bRequiresShoulderOrClavicleAssist;
    bool bRequiresPoseAdjustment;
    bool bViolatesWristComfort;
    bool bViolatesElbowPolePreference;
};
```

The reachability test should consider:

```text
hand-to-target distance
max arm extension
elbow pole validity
wrist comfort
shoulder/clavicle assist request
current stance
weapon roll/tilt allowance
simple body/weapon collision avoidance
current contact quality
```

This is still procedural reachability, not full anatomical simulation.

---

## Stance Constraints

The hold solver may read external stance/movement state as input.

```text
Standing:
  Most interaction poses are available.

Crouch:
  Weapon lowering and body slot access may be reduced.

Prone:
  Bottom magazine insertion may collide with ground.
  Large magazines may require weapon roll.
  Some interaction variants may be blocked.

Sprint from external locomotion state:
  Weapon interaction may switch to a sprint-compatible hold pose or block fine manipulation.

Falling / climbing from external locomotion state:
  Most fine manipulation tasks should be blocked or interrupted.
```

Weapon interaction does not compute locomotion speed or own the movement mode.

---

## Return State

Every manipulation action must define where the hand/contact returns.

```text
ReturnToMainGrip
ReturnToSupportGrip
ReturnToHandToHandSupport
StayOnPumpGrip
ContinueToNextAction
```

At the end of the full sequence, the weapon must return to a valid hold state:

```text
required contacts restored
hand-to-hand handgun support restored when required
shoulder/cheek/sight contacts restored when required by pose
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
  hand/contact policies
  required stability
  contact quality thresholds
  insert/extract/operate axes
```

The animation pipeline should be:

```text
Runtime solver computes canonical targets
Presentation system applies optional global mirroring
AnimInstance receives targets and state
Control Rig / IK Rig solves hands, arms, shoulders/clavicle assist, and weapon contacts
Weapon component applies procedural part movement
```

The AnimBP should not own the weapon interaction logic. It should apply the already resolved pose targets.

---

## Multiplayer Rule

Weapon holding and contact state can influence interaction readiness, but the server must not evaluate full IK.

Server authority:

```text
weapon equipped/reference state from external system
active weapon pose category
whether weapon is in a valid interaction hold state
interaction readiness/block state
interaction phase/commit state
```

Client / animation responsibility:

```text
exact hand IK
finger pose
weapon pose offsets
shoulder/cheek visual settle
local smoothing
```

Replicate state/phase/time/revision, not every IK transform.

---

## Final Formula

```text
Weapon holding =
  canonical interaction roles
  + weapon-local contacts
  + contact quality
  + weapon archetype rules
  + stabilization requirements
  + procedural reachability
  + optional mirrored presentation
  + valid return pose.
```
