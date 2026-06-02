---
id: weapon-mvp-task-checklist-for-unreal-engine
title: Weapon MVP Task Checklist for Unreal Engine
status: draft
version: 26.602.1422
tags: [ weapon, unreal-engine, ue5.7, mvp, checklist, implementation ]
---

# Weapon MVP Task Checklist for Unreal Engine

## Purpose

This document is the compact implementation checklist for building the first working Unreal Engine MVP of the weapon interaction system.

The MVP target is one reliable multiplayer-safe detachable magazine reload for a stocked rifle with a bottom magazine well, supporting both right-shoulder and left-shoulder use.

Use this checklist with:

- [Weapon Reference Implementation Flow for Unreal Engine](./weapon-reference-flow-ue.md)
- [Weapon Runtime Implementation for Unreal Engine](./weapon-runtime-implementation-ue.md)
- [Weapon Interaction Tests and Acceptance Criteria](./weapon-interaction-tests.md)
- [Weapon Interaction Implementation Roadmap](./weapon-implementation-roadmap.md)

---

## MVP Scope

Implement only:

```text
stocked rifle
bottom detachable magazine
right shoulder
left shoulder
straight-axis magazine extract/insert
MagazineDetached commit
MagazineLocked commit
visual magazine actor
owner prediction
remote phase reconstruction
basic two-hand IK through Control Rig
editor socket/axis validation
```

Do not implement in MVP:

```text
shotgun shell loops
pump action
rock-in magazine
bullpup magazine
right-side magazine
full inventory UI
lootable dropped magazines unless required
advanced finger controls
advanced network prediction smoothing
```

---

## Source Files To Create

### Runtime Public Headers

```text
Source/Game/Weapons/Public/Interaction/WeaponInteractionTypes.h
Source/Game/Weapons/Public/Interaction/WeaponGameplayTags.h
Source/Game/Weapons/Public/Interaction/WeaponInteractionProfile.h
Source/Game/Weapons/Public/Interaction/WeaponReloadTypes.h
Source/Game/Weapons/Public/Interaction/WeaponInteractionComponent.h
Source/Game/Weapons/Public/Interaction/WeaponReloadComponent.h
Source/Game/Weapons/Public/Interaction/WeaponReloadPlanner.h
Source/Game/Weapons/Public/Interaction/ProceduralWeaponManipulationComponent.h
Source/Game/Weapons/Public/Interaction/WeaponAnimInstance.h
```

### Runtime Source Files

```text
Source/Game/Weapons/Private/Interaction/WeaponGameplayTags.cpp
Source/Game/Weapons/Private/Interaction/WeaponInteractionProfile.cpp
Source/Game/Weapons/Private/Interaction/WeaponInteractionComponent.cpp
Source/Game/Weapons/Private/Interaction/WeaponReloadComponent.cpp
Source/Game/Weapons/Private/Interaction/WeaponReloadPlanner.cpp
Source/Game/Weapons/Private/Interaction/ProceduralWeaponManipulationComponent.cpp
Source/Game/Weapons/Private/Interaction/WeaponAnimInstance.cpp
```

### Editor Files

```text
Source/GameEditor/Weapons/Private/WeaponInteractionValidation.cpp
Source/GameEditor/Weapons/Private/WeaponInteractionPreviewActor.cpp
```

---

## Step 1: Types and Tags

Create:

```text
EHand
EWeaponInteractionPhase
EReloadIntent
EReloadActionType
EReloadObjectVisualState
FReplicatedWeaponMechanicalState
FReplicatedReloadInstance
FReloadActionStep
FReloadActionPlan
FWeaponHandIKTarget
FWeaponPoseOffsetAnimState
FWeaponInteractionAnimState
```

Create native gameplay tags for MVP:

```text
Weapon.Reload.Sequence.Full
Weapon.Reload.Step.PreparePose
Weapon.Reload.Step.ReachMagazine
Weapon.Reload.Step.GripMagazine
Weapon.Reload.Step.ExtractMagazine
Weapon.Reload.Step.DropMagazine
Weapon.Reload.Step.FetchMagazine
Weapon.Reload.Step.AlignMagazine
Weapon.Reload.Step.InsertMagazine
Weapon.Reload.Step.LockMagazine
Weapon.Reload.Step.ReturnHand
Weapon.Reload.Step.RestorePose
Weapon.Reload.Commit.MagazineDetached
Weapon.Reload.Commit.MagazineLocked
Weapon.Reload.Reject.NoAmmo
Weapon.Reload.Reject.InvalidProfile
Weapon.Reload.Reject.NoStableHandAssignment
Weapon.Contact.PreGrip
Weapon.Contact.VisualAttached
Weapon.GripPose.Magazine.Standard
```

Acceptance:

```text
project compiles
tags resolve at startup
all structs are visible to Blueprint where needed
```

---

## Step 2: Weapon Interaction Profile

Implement:

```text
UWeaponInteractionProfile
FWeaponFeatureSet
FWeaponGripContactSet
FWeaponInteractionPoint
```

MVP required authored data:

```text
MainGripSocket
SupportGripSocket
StockShoulderSocket
MagazineWellSocket
MuzzleSocket
MagazineWell interaction point
LocalInsertAxis
LocalExtractAxis
InsertDistance
ApproachDistance
```

Acceptance:

```text
profile asset can be created
profile references weapon sockets
invalid zero axis fails validation
missing magazine well fails validation
missing stock socket fails validation for stocked rifle
```

---

## Step 3: Reload Sequence Profile

Implement:

```text
UReloadSequenceProfile
FReloadActionStepDefinition
FReloadPhaseTiming
FReloadStepVisualPolicy
FReloadStepCommitPolicy
```

Create MVP sequence:

```text
PreparePose
ReachMagazine
GripMagazine
ExtractMagazine with MagazineDetached commit
DropMagazine or HideOldMagazine
FetchMagazine
AlignMagazine
InsertMagazine
LockMagazine with MagazineLocked commit
ReturnHand
RestorePose
```

Acceptance:

```text
sequence validates
all step ids unique
durations > 0
commit tags valid
referenced interaction points exist
```

---

## Step 4: Weapon Actor Components

Add to weapon actor:

```text
USkeletalMeshComponent WeaponMesh
UWeaponInteractionComponent
UWeaponReloadComponent
```

`UWeaponInteractionComponent` must provide:

```text
GetInteractionPoint
GetInteractionPointTransform
GetAxisWorld
ValidateRuntimeProfile
```

Acceptance:

```text
component caches weapon mesh
can resolve MagazineWell transform
can transform LocalInsertAxis into world direction
runtime validation fails loudly on bad profile
```

---

## Step 5: Reload Planner

Implement:

```text
UWeaponReloadPlanner
FReloadPlanBuildContext
FReloadPlanBuildResult
BuildReloadPlan
BuildMagazineReloadPlan
ValidateCommonRequest
```

MVP hand assignment:

```text
RightShoulder bottom magazine → LeftHand reloads, RightHand stabilizes
LeftShoulder bottom magazine → RightHand reloads, LeftHand stabilizes
```

This MVP rule may be simple, but it must still go through a hand assignment function, not hard-coded directly inside step execution.

Acceptance:

```text
planner builds valid right-shoulder plan
planner builds valid left-shoulder plan
planner rejects no ammo
planner rejects invalid profile
planner records rejection tags
```

---

## Step 6: Reload Runtime Component

Implement:

```text
UWeaponReloadComponent
TryStartReload
ServerStartReload
ServerInterruptReload
OnRep_ReloadInstance
OnRep_MechanicalState
GetLifetimeReplicatedProps
StartAuthoritativePlan
AdvanceAuthoritativeReload
StartStep
FinishStep
ApplyCommitPoint
FinishReload
```

Acceptance:

```text
server starts authoritative reload
ReloadInstance replicates
MechanicalState replicates
DOREPLIFETIME includes ReloadInstance and MechanicalState
step timing advances on server
MagazineDetached and MagazineLocked mutate mechanical state only on server
```

---

## Step 7: Object Lifecycle MVP

Implement visual magazine handling:

```text
spawn or reuse visual magazine
attach magazine to weapon MagazineWellSocket
attach magazine to left/right hand socket
hide or cosmetically drop old magazine
attach new magazine to weapon on lock visual state
remove duplicate predicted visuals on server confirmation
```

Acceptance:

```text
only one old magazine visible
only one new magazine visible
prediction rejection removes predicted magazine
remote client sees correct object visual state mid reload
```

---

## Step 8: Manipulation Component

Implement:

```text
UProceduralWeaponManipulationComponent
ApplyReloadVisualState
BuildHandTargetForReloadStep
BuildPreGripTransform
SetHandTarget
SetWeaponPoseOffset
PushToAnimInstance
ClearInteractionState
```

Acceptance:

```text
builds coherent FWeaponInteractionAnimState
computes StepAlpha from replicated server time
left/right hand targets update per step
stabilizing hand remains active
anim state clears when reload finishes or weapon switches
```

---

## Step 9: AnimInstance and Control Rig

Implement:

```text
UWeaponAnimInstance
SetWeaponInteractionState
ClearWeaponInteractionState
AnimGraph Control Rig node
Control Rig variables
Two Bone IK or equivalent for both arms
elbow pole targets
basic grip alpha
```

Control Rig solve order:

```text
convert targets
spine/shoulder assist optional for MVP
solve stabilizing hand
solve manipulation hand
correct wrist
apply grip alpha
```

Acceptance:

```text
hands do not teleport
stabilizing hand does not drift
manipulation hand reaches magazine
left/right shoulder both work
low LOD does not affect gameplay state
```

---

## Step 10: Networking MVP

Implement:

```text
owner prediction visual start
server authoritative accept/reject
ReloadRevision
MechanicalState.StateRevision
remote StepAlpha reconstruction
prediction rejection recovery
```

Acceptance:

```text
owning client sees immediate visual reload
server rejection restores valid hold
remote client joining mid reload seeks to current step
mechanical state never changes from client prediction alone
```

---

## Step 11: Editor Validation MVP

Implement:

```text
Validate Profile button or data validation pass
Draw Interaction Axes button
Magazine alignment preview
right/left shoulder hand assignment preview
basic reload sequence validation
```

Acceptance:

```text
wrong insert axis is visible
missing socket reports exact socket name
right/left shoulder preview shows resolved reload hand
sequence validation catches missing MagazineWell reference
```

---

## Step 12: MVP Test Pass

Run these acceptance tests:

```text
Bottom Magazine Right Shoulder
Bottom Magazine Left Shoulder
Owner Prediction Rejected
Remote Client Joins Mid Reload
Magazine Locked But Chamber Empty
Invalid Socket Axis Validation
Visual Object Duplicate Prevention
LOD Does Not Affect Gameplay
```

MVP is not complete until all pass.

---

## MVP Done Definition

MVP is done when:

```text
profile and sequence assets validate
right/left shoulder reload works with same weapon profile
server owns reload and mechanical state
owner prediction recovers on rejection
remote clients reconstruct current phase
magazine visual lifecycle has no duplicates
AnimInstance receives coherent state struct
Control Rig solves both hands reliably
editor preview catches socket/axis mistakes
MVP acceptance tests pass
```

---

## Final Formula

```text
MVP implementation =
  types/tags
  + profile asset
  + sequence asset
  + planner
  + runtime replication
  + object visuals
  + manipulation component
  + AnimInstance / Control Rig
  + editor validation
  + acceptance tests.
```
