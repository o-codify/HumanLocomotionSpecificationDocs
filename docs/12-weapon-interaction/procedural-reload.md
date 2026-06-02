---
id: procedural-weapon-reloading
title: Procedural Weapon Reloading
status: draft
version: 26.602.1237
tags: [ weapon, reload, upper-body, ik, procedural-animation, unreal-engine, multiplayer, replication ]
---

# Procedural Weapon Reloading

## Purpose

This document defines procedural weapon reloading as an upper-body manipulation system built on top of the weapon holding and stabilization model.

Reloading is not a hard-coded animation for a pistol, rifle, shotgun, or sniper rifle. It is a sequence of small manipulation actions that move hands, ammunition objects, magazines, and weapon mechanisms between valid contact states.

The core idea:

```text
Reloading = temporary disruption of a valid weapon hold pose
          + object transfer
          + directed insertion/extraction/operation
          + restoration of a valid weapon hold pose
```

---

## Scope

The system must support:

```text
magazine reloads
single-round insertion
shotgun shell loading
bolt cycling
slide racking
pump action
break-action open/close
bullpup magazine positions
side/top/bottom magazine wells
right-shoulder and left-shoulder stances
weapons with and without stock
multiplayer replication
owning-client prediction
remote-client visual reconstruction
```

The system must not require a separate hard-coded reload implementation per weapon type.

---

## Main Rule

Every reload action must answer these questions:

```text
Where is the right hand now?
Where is the left hand now?
Which contacts stabilize the weapon?
What object is the manipulation hand holding?
Where is that object attached?
Which direction is the action performed along?
Where does the hand return after the action?
What happens if the action is interrupted?
What gameplay state changes at this step?
What is replicated to other clients?
```

If a step cannot answer these questions, the step is incomplete.

---

## Runtime Input State

A reload plan is built from explicit runtime state.

```text
ReloadRequest
+ CharacterWeaponInteractionState
+ HandInteractionState
+ WeaponHoldState
+ WeaponMechanicalState
+ WeaponInteractionProfile
+ Inventory/BodySlots
→ Validated ReloadActionPlan
```

### Reload request

```cpp
struct FReloadRequest
{
    EReloadIntent Intent;        // FullReload, TacticalReload, EmergencyReload, LoadOne, CycleOnly
    TObjectPtr<AActor> Weapon;
    uint8 ClientPredictionId;
};
```

### Weapon feature set

```cpp
struct FWeaponFeatureSet
{
    bool bHasMagazine;
    bool bHasDetachableMagazine;
    bool bHasInternalMagazine;
    bool bHasSingleRoundChamber;
    bool bHasBolt;
    bool bHasChargingHandle;
    bool bHasSlide;
    bool bHasPump;
    bool bHasBreakAction;
    bool bHasStock;
    bool bHasShoulderContact;
};
```

### Weapon mechanical state

```cpp
struct FWeaponMechanicalState
{
    bool bMagazineInserted;
    bool bMagazineLocked;
    bool bRoundChambered;
    bool bBoltOpen;
    bool bNeedsCycle;
    int32 AmmoInMagazine;
    int32 AmmoInChamber;
    int32 InternalAmmoCount;
};
```

The mechanical state is required because reload is not binary. Examples:

```text
magazine inserted but not locked
magazine locked but chamber empty
bolt open
pump back but not forward
one shotgun shell inserted but tube is not full
```

---

## Reload Planner and Executor

The system must separate planning from visual execution.

```text
ReloadRequest
  ↓
ReloadPlanner
  ↓
ReloadActionPlan
  ↓
ReloadExecutor
  ↓
IK / weapon bones / object visuals / gameplay commit
```

The planner decides what must happen. The executor performs the steps over time.

The planner must answer:

```text
Does the weapon need reload?
Which reload intent is valid?
Does the weapon have a magazine, chamber, tube, bolt, pump, or slide?
Does the old magazine need to be removed?
Should the old magazine be stowed or dropped?
Where is the replacement object?
Which hand performs each action?
Which contacts stabilize the weapon?
Does the weapon need a reload pose adjustment?
Does the weapon need bolt/slide/pump operation after insertion?
Where are gameplay commit points?
What recovery is used if interrupted?
```

---

## Reload Action Step

Runtime reload steps should be explicit and small.

```cpp
struct FReloadActionStep
{
    EReloadActionType Type;

    EHand ResolvedHand;
    EHand OtherHand;

    FName TargetSocket;
    FName ReturnSocket;

    TObjectPtr<UObject> Object;

    FVector AxisWorld;
    float Distance;
    float Duration;

    EGameplayCommitPoint CommitPoint;

    bool bCanInterrupt;
    EInterruptRecoveryPolicy RecoveryPolicy;
};
```

Common step types:

```text
PrepareWeaponPose
ResolveHandAssignment
Regrip
ReleaseGrip
ReachSocket
GripObject
DetachObject
MoveAlongAxis
AttachObject
OperateWeaponPart
ReturnHand
RestorePose
```

---

## Reload as Action Sequence

A reload sequence is composed of reusable actions:

```text
PrepareWeaponPose
ResolveHandAssignment
ReleaseGrip
ReachPoint
GripObject
ExtractObject
MoveObjectToBodySlot
DropObject
FetchObjectFromBodySlot
AlignObject
InsertObject
LockObject
ReleaseObject
OperateMechanism
ReturnHandToGrip
RestoreWeaponPose
```

Weapon-specific behavior is expressed through data:

```text
interaction point transforms
access regions
insert/extract/operate axes
required stabilization
object sockets
sequence order
dependency rules
commit points
replication events
```

---

## Interaction Points

Every reload-relevant point must have a full semantic description.

Example fields:

```text
Name
SocketName
InteractionPointType
AccessRegion
PreferredHandPolicy
RequiredStability
LocalInsertAxis
LocalExtractAxis
LocalOperateAxis
ApproachDistance
InsertDistance
LockDistance
```

A socket transform alone is not enough for gameplay logic. It says where the point is, but not how it should be used.

---

## Weapon Profile Validation

A data-driven weapon system requires validation. Invalid assets should fail loudly in editor and debug builds.

Examples:

```text
If bHasDetachableMagazine:
  MagazineWellSocket must exist.
  InsertAxis must be valid.
  ExtractAxis must be valid.
  AccessRegion must be defined.
  Compatible MagazineObjectProfile must exist.

If bHasStock:
  StockShoulderSocket must exist.

If bHasPump:
  PumpBone must exist.
  PumpGripSocket must exist.
  PumpBackAxis and PumpForwardAxis must be valid.

If bHasBolt:
  BoltSocket must exist.
  OperateAxis must be valid.
```

Example validation error:

```text
WeaponProfile SMG_01 invalid:
- MagazineWellSocket missing
- Magazine InsertAxis is zero vector
- Bolt access region defined but BoltSocket missing
```

---

## Magazine Insertion Model

A magazine is normally a separate object, not a permanent weapon bone.

Magazine object sockets:

```text
MagazineRoot
HandGripSocket
InsertTipSocket
LockSocket optional
```

Weapon sockets:

```text
MagazineWellSocket
MagazinePreInsertSocket optional
```

The system aligns the magazine object so that the magazine insert tip matches the magazine well transform.

```text
DesiredMagazineWorldTransform =
  WeaponMagazineWellWorldTransform
  * Inverse(MagazineInsertTipLocalTransform)
```

This allows the magazine root to be anywhere and the magazine shape to be arbitrary.

---

## Direction Is Socket Axis, Not Point-To-Point Line

The insertion direction must not be derived only from the line between two positions.

Bad approach:

```text
direction = normalize(WeaponSocketPosition - MagazineTipPosition)
```

This fails for:

```text
angled magazines
curved magazines
side-mounted magazines
top-mounted magazines
bullpup magazines
custom sci-fi magazine wells
```

Correct approach:

```text
InsertDirection = WeaponMagazineWellSocket.TransformAxis(LocalInsertAxis)
```

Example convention:

```text
MagazineWellSocket +X = insertion direction
MagazineWellSocket +Z = magazine outward direction
MagazineWellSocket +Y = side direction
```

Then an MP5-style angled magazine, a bottom magazine, a top magazine, or a side magazine all use the same code.

---

## Pre-Insert and Final Insert

Insertion uses at least two target poses:

```text
PreInsertPose = MagazineWellTransform - InsertDirection * InsertDistance
FinalInsertPose = MagazineWellTransform
```

The magazine first aligns to the well, then travels along the insert axis.

```text
Approach → Align → Insert → Lock
```

The insert motion should not be a direct hand teleport to the final socket.

---

## Magazine Extraction

Extraction is the reverse of insertion, but it may use a different axis.

```text
ExtractDirection = MagazineWellSocket.TransformAxis(LocalExtractAxis)
```

For simple weapons:

```text
ExtractDirection = -InsertDirection
```

For some weapons:

```text
extract down/back
insert up/forward
rock-in magazine path
curved path
```

Therefore extract and insert axes should be separately configurable.

---

## Rock-In and Curved Magazine Paths

Some magazines do not move in a straight line. They rotate into place.

The system should support optional multi-stage insertion:

```text
Stage 1: hook front/rear point
Stage 2: rotate magazine around pivot
Stage 3: lock magazine
```

Data fields:

```text
bUseRockIn
RockPivotSocket
RockStartAngle
RockEndAngle
LockMotionDistance
```

This is still the same reload system. Only the insertion path changes.

---

## Access Region and Hand Selection

Hand selection is based on geometry and current stance.

Base preference:

```text
Right access → prefer RightHand
Left access  → prefer LeftHand
Top/Bottom access + RightShoulder → prefer LeftHand
Top/Bottom access + LeftShoulder  → prefer RightHand
```

But this preference is not final.

The system must check:

```text
Can the selected hand release its current grip?
Will the weapon remain stabilized?
Does the selected hand reach the point without impossible twisting?
Is a regrip required first?
Should the weapon roll/tilt into a reload pose?
```

Final result:

```text
PreferredHand → stability/reachability/cost check → ResolvedHand
```

---

## Body Slot Selection

Body slots are also interaction points.

Examples:

```text
ChestMagazinePouch
BeltMagazinePouch
LeftShellCarrier
RightShellCarrier
BackpackAmmoSlot
```

Body slot data should include:

```text
SocketName
AccessRegion
HandPolicy
ReloadObjectType
Priority
```

The planner should choose a compatible object from a reachable slot. This can influence hand choice.

```text
Chosen hand depends on weapon access + body slot access + stability.
```

---

## Weapons With Stock

A stocked weapon can often be stabilized by:

```text
grip hand + shoulder contact
```

This allows the other hand to leave the weapon.

Example right-side bolt on right shoulder:

```text
1. LeftHand strengthens support grip.
2. RightHand releases MainGrip.
3. RightHand operates BoltSocket.
4. RightHand returns to MainGrip.
5. Two-hand hold is restored.
```

Weapon is stabilized by:

```text
LeftHand + RightShoulder
```

---

## Weapons Without Stock

A weapon without stock has fewer stabilization contacts.

Example pistol:

```text
RightHand keeps MainGrip.
LeftHand manipulates magazine or slide.
```

If the same-side hand is the only valid stabilizer, the system must not release it without a regrip.

Possible solutions:

```text
use the opposite hand if reachable
rotate the weapon into a reload pose
first transfer stabilization to the other hand
block that reload variant
```

---

## Magazine Reload Example: Bottom Magazine, Right Shoulder

Initial state:

```text
RightHand = MainGrip
LeftHand = SupportGrip
RightShoulderContact = active
```

Sequence:

```text
1. PrepareWeaponPose: lower/roll weapon into reload pose.
2. RightHand + shoulder stabilize weapon.
3. LeftHand releases SupportGrip.
4. LeftHand reaches MagazineWellSocket.
5. LeftHand grips old magazine.
6. LeftHand extracts magazine along ExtractDirection.
7. Old magazine is dropped or stowed.
8. LeftHand fetches new magazine from body slot.
9. New magazine aligns InsertTipSocket to PreInsertPose.
10. LeftHand inserts magazine along InsertDirection.
11. Magazine locks to Weapon.MagazineWellSocket.
12. LeftHand releases magazine.
13. LeftHand returns to SupportGrip.
14. Restore previous weapon pose or updated pose.
```

---

## Magazine Reload Example: Bottom Magazine, Left Shoulder

Initial state:

```text
LeftHand = MainGrip
RightHand = SupportGrip
LeftShoulderContact = active
```

Sequence mirrors the right-shoulder version:

```text
1. LeftHand + left shoulder stabilize weapon.
2. RightHand releases SupportGrip.
3. RightHand changes the bottom magazine.
4. RightHand returns to SupportGrip.
```

The weapon data does not need a separate left-shoulder reload. The solver resolves it from shoulder side and access region.

---

## Magazine Reload Example: Right-Side Magazine

For a right-side magazine:

```text
AccessRegion = Right
PreferredHandPolicy = SameSide
PreferredHand = RightHand
```

If the weapon is on the left shoulder:

```text
LeftHand + LeftShoulder stabilize
RightHand changes magazine
```

If the weapon is on the right shoulder and RightHand is on MainGrip:

```text
1. LeftHand strengthens support contact.
2. RightHand releases MainGrip.
3. RightHand changes magazine.
4. RightHand returns to MainGrip.
```

Same-side access is preserved, but stabilization is explicitly planned.

---

## Bullpup / Rear Magazine Example

A bullpup weapon is not a special system case.

Data example:

```text
MagazineWellSocket = behind MainGripSocket
AccessRegion = BottomRear or Custom
InsertAxis = socket local +X
ExtractAxis = socket local -X or custom down/back
PreferredHandPolicy = OppositeShoulderSide or Custom
```

The solver uses the actual transform and access region. The sequence remains:

```text
extract old magazine
move/stow/drop old magazine
fetch new magazine
align insert tip
insert along axis
lock
return hand
```

---

## Single-Round Insertion

Single-round reloads use the same manipulation framework.

Objects:

```text
RoundObject
RoundHandGripSocket
RoundInsertTipSocket
```

Weapon points:

```text
ChamberSocket
ShellInsertSocket
EjectionPortSocket
```

Sequence:

```text
1. Stabilize weapon.
2. Manipulation hand fetches round from body slot.
3. Round insert tip aligns to chamber or shell insert socket.
4. Round moves along InsertDirection.
5. Round commits to weapon ammo state.
6. Hand releases or continues to next round.
7. Hand returns to grip when sequence ends.
```

---

## Shotgun Shell Loading

Tube-fed shotgun loading is a repeated single-round insertion sequence.

Example bottom loading port:

```text
ShellInsertSocket.AccessRegion = Bottom
PreferredHandPolicy = OppositeShoulderSide
```

Right shoulder:

```text
RightHand + shoulder stabilize
LeftHand fetches and inserts shells
```

Left shoulder:

```text
LeftHand + shoulder stabilize
RightHand fetches and inserts shells
```

Repeated loop:

```text
FetchShell → AlignShell → InsertShell → CommitShell → FetchNextShell
```

The loop can be interrupted after each shell.

---

## Bolt, Slide, and Charging Handle Operation

Mechanism operation is also a directed manipulation action.

Interaction point fields:

```text
OperateSocket
AccessRegion
OperateAxis
OperateDistance
ReturnAxis
ReturnDistance
RequiredStability
```

For moving parts, the action should drive the weapon bone and the hand should follow the socket on that bone.

```text
Action drives BoltBone / SlideBone / PumpBone
Hand IK follows socket on the moving bone
```

This is more stable than letting the hand pull the weapon part physically.

---

## Pump Action

Pump action uses a moving fore-end.

Weapon structure:

```text
PumpBone
  └─ PumpGripSocket
```

Sequence:

```text
1. Stabilization hand remains on MainGrip and shoulder contact.
2. Pump hand stays attached to PumpGripSocket.
3. PumpBone moves backward along PumpBackAxis.
4. PumpBone moves forward along PumpForwardAxis.
5. Pump hand remains or returns to support state.
```

Pump action is not a detached reach action. The hand is attached to a moving weapon part.

---

## Object Attachment States

Reload objects should have explicit states:

```text
AttachedToWeapon
AttachedToHand
AttachedToBodySlot
DroppedInWorld
Inserted
Consumed
```

Example magazine transition:

```text
Weapon.MagazineWell → Hand → BodySlot or World
BodySlot → Hand → Weapon.MagazineWell
```

Example round transition:

```text
BodySlot → Hand → Chamber/Tube → Consumed/Inserted
```

For multiplayer, distinguish gameplay objects from visual objects.

```text
Gameplay object = server-owned inventory/ammo state.
Visual object = locally animated representation.
```

If a dropped magazine can be picked up, the server spawns a replicated dropped magazine actor. If it is only cosmetic, clients can spawn local visuals.

---

## Gameplay Commit Points

Visual animation and gameplay state should be separated.

Examples:

```text
Magazine becomes active at MagazineLock.
Shell becomes loaded at ShellSeated.
Round becomes chambered at ChamberCommit.
Bolt cycle completes at BoltForwardComplete.
```

The hand may still be returning after gameplay commit.

Optional gameplay rules:

```text
CannotFireDuringReload
CanFireAfterCommit
CanFireWithPenaltyAfterCommit
CanFireOnlyWhenStableHoldRestored
```

The server must own these rules.

---

## Interruption and Recovery

Reload can be interrupted. The system must recover to a valid weapon hold state.

Possible partial states:

```text
old magazine removed
new magazine in hand
round in hand
right hand away from main grip
left hand away from support grip
bolt open
pump partially moved
```

Recovery rules:

```text
If weapon has no magazine and hand holds a magazine:
  insert quickly or drop/stow and return to grip.

If hand holds a round:
  stow/drop round and return to grip, or finish insertion if allowed.

If trigger/main hand is away from grip:
  return it first if weapon stability is poor.

If bolt or pump is partially operated:
  complete or reset mechanism before returning to ready state.
```

The recovery target is not necessarily fully reloaded. It is a valid hold pose.

In multiplayer, interruption is server-authoritative.

---

## Multiplayer Replication Model

The server owns gameplay state. Clients own visual reconstruction.

Server authority:

```text
reload start validation
reload intent
ammo count
magazine inventory state
magazine attachment state
round inserted state
mechanism state
commit points
interrupt/recovery
fire permission
```

Client visual reconstruction:

```text
hand IK targets
weapon pose interpolation
magazine movement
round movement
bolt/slide/pump visual motion
minor timing offsets
```

Do not replicate per-frame hand IK, elbow positions, finger poses, or weapon part transforms.

Replicate:

```text
reload sequence id
current step index
step start server time
step duration
resolved hand
object visual attachment state
mechanical state revision
commit events
interrupt/recovery state
```

---

## Replicated Reload Instance

```cpp
USTRUCT()
struct FReplicatedReloadInstance
{
    GENERATED_BODY()

    UPROPERTY()
    bool bIsReloading = false;

    UPROPERTY()
    FGameplayTag ReloadSequenceId;

    UPROPERTY()
    FGameplayTag CurrentStepId;

    UPROPERTY()
    uint8 StepIndex = 0;

    UPROPERTY()
    float StepStartServerTime = 0.f;

    UPROPERTY()
    float StepDuration = 0.f;

    UPROPERTY()
    EHand ResolvedHand = EHand::None;

    UPROPERTY()
    EReloadObjectType ObjectType;

    UPROPERTY()
    EReloadObjectVisualState ObjectVisualState;

    UPROPERTY()
    TObjectPtr<AActor> ReloadObjectActor;

    UPROPERTY()
    FGameplayTag CommitPoint;

    UPROPERTY()
    uint8 ReloadRevision = 0;
};
```

Clients compute step phase from server time:

```cpp
float Alpha = (ServerTimeNow - StepStartServerTime) / StepDuration;
```

This allows late relevancy and late packet arrival to seek directly to the correct visual phase.

---

## Replicated Weapon Mechanical State

```cpp
USTRUCT()
struct FReplicatedWeaponMechanicalState
{
    GENERATED_BODY()

    UPROPERTY()
    bool bMagazineInserted = false;

    UPROPERTY()
    bool bMagazineLocked = false;

    UPROPERTY()
    bool bRoundChambered = false;

    UPROPERTY()
    bool bBoltOpen = false;

    UPROPERTY()
    bool bNeedsCycle = false;

    UPROPERTY()
    int32 AmmoInMagazine = 0;

    UPROPERTY()
    int32 AmmoInChamber = 0;

    UPROPERTY()
    uint8 StateRevision = 0;
};
```

This is more useful than a single `AmmoCount` because it represents partial reload states.

---

## Owning Client Prediction

The owning client should start predicted visuals immediately after input.

```text
Input Reload
  ↓
Start predicted visual reload
  ↓
ServerStartReload(ReloadIntent, WeaponNetId, ClientPredictionId)
  ↓
Server confirms or rejects
```

If confirmed:

```text
client aligns predicted reload with replicated server time
```

If rejected:

```text
client cancels predicted visuals and recovers to valid weapon hold pose
```

Common rejection reasons:

```text
no ammo
weapon already busy
invalid mechanical state
character is sprinting/falling/climbing
weapon profile invalid
server state changed
```

---

## Remote Client Playback

Remote clients do not need exact hand trajectories. They need the correct phase and state.

On `OnRep_ReloadInstance`:

```text
resolve local reload profile
compute current alpha from server time
set visual object attachment state
play/seek procedural visual step
```

If a remote client becomes relevant mid-reload, it should not replay from the beginning. It should reconstruct from the replicated step and timestamp.

---

## UE Networking Pattern

Recommended RPC/state pattern:

```text
Owning client:
  ServerStartReload(ReloadIntent, WeaponNetId, ClientPredictionId)
  ServerInterruptReload(Reason)

Server replicated state:
  FReplicatedReloadInstance
  FReplicatedWeaponMechanicalState

Optional cosmetic events:
  ReloadStarted
  ReloadStepChanged
  MagazineDetached
  MagazineAttachedToHand
  MagazineDropped
  MagazineLocked
  RoundInserted
  BoltOpened
  BoltClosed
  PumpBack
  PumpForward
  ReloadInterrupted
  ReloadCompleted
```

Cosmetic multicast events may play sounds and effects, but gameplay results must come from replicated authoritative state.

---

## Unreal Engine Implementation Notes

Recommended runtime components:

```text
UProceduralWeaponManipulationComponent   // character-owned hand/contact/IK target state
UWeaponInteractionComponent              // weapon sockets/profile/moving parts
UWeaponReloadComponent                   // planner, executor, replication, commit points
```

Recommended data assets:

```text
UWeaponInteractionProfile
UReloadSequenceProfile
UAmmoObjectProfile
UBodySlotProfile
UWeaponReloadPolicy
```

Recommended animation flow:

```text
Reload planner resolves action plan
Reload component replicates state and timings
Runtime component updates hand/object/weapon targets
AnimInstance receives targets
Control Rig solves arms, shoulders, spine, hands
Weapon skeletal mesh applies part bone offsets
```

The AnimBP should not decide reload logic. It should only consume targets and states.

---

## Authoring Workflow

Recommended editor workflow:

```text
1. Add sockets/bones to weapon skeletal mesh.
2. Create WeaponInteractionProfile.
3. Assign MainGrip, SupportGrip, Stock, Muzzle.
4. Configure MagazineWell, Bolt, Slide, Pump, ShellInsert points as needed.
5. Configure access regions and local axes.
6. Assign compatible ammo/magazine object profiles.
7. Run Validate Profile.
8. Run Preview Reload.
9. Inspect debug axes, hand paths, pre-insert/final poses, and rejected hand reasons.
```

Preview is required because most data errors are transform/axis errors.

---

## Debug Requirements

Debug view should show:

```text
MagazineWellSocket transform
InsertAxis
ExtractAxis
PreInsertPose
FinalInsertPose
PreferredHand
ResolvedHand
Rejected hand reasons
Reachability cost
ActiveStabilizationContacts
Weapon pose offset
Object attachment state
Gameplay commit point
Reload step index and alpha
Replicated server time phase
Mechanical state revision
```

For angled magazines, debug must draw the socket axes. This is the easiest way to see whether the interaction point has been authored correctly.

---

## MVP Requirements

Minimum implementation:

```text
1. Weapon local-space sockets for grips, magazine well, bolt, muzzle.
2. Magazine object with hand grip and insert tip sockets.
3. Weapon profile validation.
4. Hand assignment solver using access region, shoulder side, stability, and reachability cost.
5. Stability check before releasing a hand.
6. Straight-axis magazine extract/insert using socket axis.
7. Body slot fetch for new magazine.
8. Magazine attach/detach states.
9. Optional bolt/slide operation.
10. Return hands to valid weapon hold pose.
11. Replicated ReloadInstance.
12. Replicated WeaponMechanicalState.
13. Owning client visual prediction.
14. Remote client phase reconstruction from server time.
15. Debug visualization for axes, contacts, step alpha, and replication phase.
```

---

## V2

Add:

```text
single-round insertion
shotgun shell loops
rock-in magazine insertion
left/right shoulder mirrored reload poses
pistol-specific no-stock constraints
interrupt recovery
network event refinement
Gameplay Tags for reload/mechanical states
editor preview tooling
```

---

## Final Formula

```text
Procedural reload =
  validated weapon profile
  + hold-state aware hand assignment
  + stabilized weapon contacts
  + object attachment transitions
  + socket-axis directed movement
  + mechanical state commit points
  + replicated reload phase
  + client-side visual reconstruction
  + return to valid weapon hold.
```

The reload system should work for conventional weapons and unusual weapons because the weapon describes geometry and interaction semantics, not because code knows a specific weapon class.
