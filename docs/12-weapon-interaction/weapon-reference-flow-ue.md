---
id: weapon-reference-implementation-flow-for-unreal-engine
title: Weapon Reference Implementation Flow for Unreal Engine
status: draft
version: 26.602.1419
tags: [ weapon, unreal-engine, ue5.7, reference-flow, implementation ]
---

# Weapon Reference Implementation Flow for Unreal Engine

## Purpose

This document gives one complete end-to-end implementation flow for a detachable magazine reload in Unreal Engine 5.7.

It connects the documents in this section into one concrete path:

```text
profile data
→ reload sequence profile
→ planner output
→ runtime replicated state
→ object lifecycle
→ animation state
→ Control Rig visualization
→ acceptance tests
```

Use this document as the first reference scenario when implementing the system.

---

## Reference Scenario

Weapon:

```text
stocked rifle
bottom detachable magazine
right-shoulder and left-shoulder supported
optional bolt/charging handle after magazine lock
```

Initial right-shoulder state:

```text
RightHand = MainGrip
LeftHand = SupportGrip
RightShoulderContact = active
MagazineInserted = true
MagazineLocked = true
RoundChambered may be true or false
```

---

## Step 1: Author Weapon Profile

Create [Weapon Interaction Profile for Unreal Engine](./weapon-interaction-profile-ue.md).

Required sockets:

```text
MainGripSocket
SupportGripSocket
StockShoulderSocket
MagazineWellSocket
MuzzleSocket
```

Optional mechanism socket/bone:

```text
BoltBone or ChargingHandleBone
BoltGrabSocket or ChargingHandleSocket
```

Magazine well interaction point:

```text
Name = MagazineWell
SocketName = MagazineWellSocket
Type = MagazineWell
AccessRegion = Bottom
HandPolicy = OppositeShoulderSide
RequiredStability = OneHandPlusShoulder
LocalInsertAxis = +X
LocalExtractAxis = -X unless authored otherwise
InsertDistance = authored magazine travel distance
```

Validation must pass before runtime use.

---

## Step 2: Author Reload Object Profile

Create magazine visual/gameplay object data.

Required object points:

```text
HandGripSocket
InsertTipSocket
LockSocket optional
```

Alignment rule:

```text
DesiredMagazineWorldTransform =
  MagazineWellWorldTransform
  * Inverse(MagazineInsertTipLocalTransform)
```

The magazine actor root does not need to be at the insert tip.

---

## Step 3: Author Reload Sequence Profile

Create [Weapon Reload Sequence Profile for Unreal Engine](./weapon-reload-sequence-profile-ue.md).

Reference sequence:

```text
Step 1: PrepareWeaponPose
Step 2: Reach old magazine
Step 3: Grip old magazine
Step 4: Extract magazine
  CommitPoint = MagazineDetached
Step 5: Drop or stow old magazine
Step 6: Fetch new magazine
Step 7: Align magazine
Step 8: Insert magazine
Step 9: Lock magazine
  CommitPoint = MagazineLocked
Step 10: Return hand to support grip
Step 11: Restore weapon pose
```

For a weapon that needs cycling, planner may append:

```text
OperateChargingHandle or OperateBolt
CommitPoint = BoltClosed / RoundChambered depending on weapon policy
```

---

## Step 4: Runtime Reload Request

Owning client calls:

```cpp
WeaponReloadComponent->TryStartReload(EReloadIntent::FullReload);
```

Owning client may immediately start predicted visual reload.

Server receives:

```cpp
ServerStartReload(EReloadIntent::FullReload, ClientPredictionId);
```

Server authority starts here.

---

## Step 5: Build Planner Context

`UWeaponReloadComponent` builds `FReloadPlanBuildContext`:

```text
Intent
Character
WeaponActor
InteractionComponent
InventoryComponent
CharacterState
HoldState
MechanicalState
ClientPredictionId
```

Then calls:

```cpp
FReloadPlanBuildResult Result = ReloadPlanner->BuildReloadPlan(Context);
```

If result fails, server rejects and owning client recovers predicted visuals.

---

## Step 6: Hand Assignment

For right shoulder bottom magazine:

```text
PreferredHand = LeftHand
Stabilization = RightHand + RightShoulder
```

For left shoulder bottom magazine:

```text
PreferredHand = RightHand
Stabilization = LeftHand + LeftShoulder
```

The planner records resolved hand in runtime steps. It must not hard-code left hand as reload hand.

---

## Step 7: Runtime Plan Output

Server stores `FReloadActionPlan` with runtime-resolved steps.

Example simplified output:

```text
0 PrepareWeaponPose      ResolvedHand=None
1 ReleaseGrip            ResolvedHand=Left
2 ReachSocket            ResolvedHand=Left Target=MagazineWell
3 GripObject             ResolvedHand=Left
4 MoveAlongAxis          ResolvedHand=Left Axis=ExtractAxis Commit=MagazineDetached
5 DropObject             ResolvedHand=Left
6 FetchObject            ResolvedHand=Left Source=SelectedBodySlot
7 AlignObject            ResolvedHand=Left Target=MagazineWell
8 MoveAlongAxis          ResolvedHand=Left Axis=InsertAxis
9 LockObject             ResolvedHand=Left Commit=MagazineLocked
10 ReturnHand            ResolvedHand=Left Target=SupportGrip
11 RestorePose           ResolvedHand=None
```

---

## Step 8: Start Authoritative Reload State

`UWeaponReloadComponent` updates replicated state:

```text
ReloadInstance.bIsReloading = true
ReloadInstance.ReloadSequenceId = selected sequence id
ReloadInstance.StepIndex = 0
ReloadInstance.CurrentStepId = first step id
ReloadInstance.StepStartServerTime = server time
ReloadInstance.StepDuration = step duration
ReloadInstance.Phase = step phase
ReloadInstance.ResolvedHand = resolved hand
ReloadInstance.ReloadRevision++
```

Clients receive `OnRep_ReloadInstance` and reconstruct visuals.

---

## Step 9: Object Lifecycle

Object lifecycle follows [Weapon Reload Object Lifecycle for Unreal Engine](./weapon-object-lifecycle-ue.md).

Old magazine:

```text
InWeapon → InLeftHand → DroppedWorld or InBodySlot
```

New magazine:

```text
InBodySlot or Hidden → InLeftHand → InWeapon
```

Gameplay inventory changes only at commit points.

---

## Step 10: Animation State

Manipulation component builds `FWeaponInteractionAnimState`:

```text
CurrentStepId
CurrentPhase
StepAlpha
ManipulationHand
LeftHand target
RightHand target
WeaponPoseOffset
MovingPart states optional
```

Then pushes to `UWeaponAnimInstance`.

Control Rig receives the state and solves:

```text
stabilizing hand first
manipulation hand second
wrist/finger pose after IK
```

---

## Step 11: Commit Points

When extraction commit occurs:

```text
MagazineDetached
```

Server updates:

```text
MechanicalState.bMagazineInserted = false
MechanicalState.bMagazineLocked = false
MechanicalState.StateRevision++
```

When lock commit occurs:

```text
MagazineLocked
```

Server updates:

```text
MechanicalState.bMagazineInserted = true
MechanicalState.bMagazineLocked = true
MechanicalState.AmmoInMagazine = selected magazine ammo count
MechanicalState.StateRevision++
```

If weapon requires cycling:

```text
MechanicalState.bNeedsCycle = true until mechanism commit completes
```

---

## Step 12: Remote Client Reconstruction

Remote client uses:

```text
ReloadInstance.StepIndex
ReloadInstance.StepStartServerTime
ReloadInstance.StepDuration
ReloadInstance.ObjectVisualState
MechanicalState.StateRevision
```

It computes:

```text
StepAlpha = (ServerTimeNow - StepStartServerTime) / StepDuration
```

It does not replay from the beginning.

---

## Step 13: Completion

At final step:

```text
hand returned to support grip
weapon pose restored or updated
stability valid
ReloadInstance.bIsReloading = false
ReloadRevision increments
```

Fire permission is evaluated from:

```text
mechanical state
reload state
weapon policy
hold stability
```

---

## Acceptance Tests

This reference flow must pass tests from [Weapon Interaction Tests and Acceptance Criteria](./weapon-interaction-tests.md):

```text
Bottom Magazine Right Shoulder
Bottom Magazine Left Shoulder
Remote Client Joins Mid Reload
Owner Prediction Rejected
Magazine Locked But Chamber Empty
Visual Object Duplicate Prevention
Invalid Socket Axis Validation
LOD Does Not Affect Gameplay
```

---

## Final Formula

```text
Reference reload flow =
  authored profile
  + authored sequence
  + planner context
  + resolved action plan
  + authoritative replicated state
  + object lifecycle
  + animation state bridge
  + Control Rig visualization
  + acceptance tests.
```
