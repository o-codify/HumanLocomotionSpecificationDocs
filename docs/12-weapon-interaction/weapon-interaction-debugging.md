---
id: weapon-interaction-debugging
title: Weapon Interaction Debugging
status: draft
version: 26.602.1531
tags: [ weapon, interaction, debugging, visualization, tools ]
---

# Weapon Interaction Debugging

## Purpose

This document defines a unified debug vocabulary and debug view set for weapon interaction.

Many documents define local debug requirements. This document brings them together into one practical debugging model.

---

## Scope Boundary

Debugging here covers weapon interaction state:

```text
pose state
contacts
constraints
hand targets
object visual state
reload/manipulation phases
mirroring
network reconstruction
prediction correction
```

It does not debug full inventory, damage, projectile simulation, camera system, body orientation solver, or locomotion speed.

---

## Debug Categories

Recommended categories:

```text
Pose
Contacts
Constraints
Hands
Objects
MovingParts
Mirroring
Networking
Prediction
Authoring
LOD
```

---

## Pose Debug

Show:

```text
CurrentPoseState
DesiredPoseState
PreviousStablePoseState
PoseLifecyclePhase
PoseAlpha
PoseStartServerTime
PoseBlendDuration
HoldStabilityScore
```

Useful colors/labels:

```text
Stable
Stabilizing
TemporarilyDisrupted
Recovering
Failed
```

---

## Contact Debug

Show each contact:

```text
ContactId
ContactType
ContactState
Quality
Confidence
RequiredThreshold
ReleasedByPlan
TimeBelowThreshold
RecoveryTarget
```

Important contacts:

```text
MainGrip
SupportGrip
ShoulderContact
ObjectGrip
ObjectInsertTip
MovingPartGrip
```

---

## Constraint Debug

Show:

```text
ConstraintName
ConstraintClass
Priority
bSatisfied
bTemporarilyBroken
BlockingReason
WinningConstraint
LosingConstraint
RecoveryTarget
```

Constraint class values:

```text
NeverBreak
Hard
TemporaryBreak
Soft
```

---

## Hand Target Debug

Show canonical and presented targets separately:

```text
CanonicalLeftHandTarget
CanonicalRightHandTarget
PresentedLeftHandTarget
PresentedRightHandTarget
LeftElbowPole
RightElbowPole
PositionAlpha
RotationAlpha
FingerGripAlpha
ManipulationHandRole
StabilizingHandRole
```

Draw lines:

```text
current hand → target
current elbow → pole
hand target path
recovery target path
```

---

## Object Debug

Show:

```text
ObjectVisualState
ObjectType
ObjectActor
GameplayObjectId optional
SourceBodySlotId optional
TargetInteractionPoint
CanonicalObjectTransform
PresentedObjectTransform
AttachmentParent
PredictedOnly
AuthorityConfirmed
DuplicateVisualDetected
```

---

## Moving Part Debug

Show:

```text
MovingPartName
BoneName
FollowSocketName
LocalTravelAxis
CurrentAlpha
CurrentDistance
CommittedState
HandFollowingPart
```

Draw:

```text
travel axis
rest position
committed position
current part position
hand follow target
```

---

## Mirroring Debug

Show:

```text
bMirroredPresentation
PresentationSide
MirrorPlane
MirrorStage
CanonicalPoseState
PresentedPoseTransform
CanonicalAxisWorld
PresentedAxisWorld
DoubleMirrorWarning
```

Common mirror warnings:

```text
Control Rig received mirrored target but mirror flag still enabled
object path not mirrored with hand path
axis semantic changed after mirror
remote client mirrored already-presented state
```

---

## Networking Debug

Show:

```text
ReloadRevision
PoseRevision
MechanicalInteractionRevision
StepId
StepIndex
StepStartServerTime
StepDuration
ComputedStepAlpha
AuthorityState
PredictedState
CorrectionPending
```

Remote reconstruction should show:

```text
server time
local estimated server time
phase alpha
replicated pose state
replicated object visual state
```

---

## Prediction Debug

Show:

```text
PredictionId
PredictedAction
PredictedStartTime
AcceptedByAuthority
RejectedByAuthority
CorrectionBlendTime
PredictedObjectVisuals
RemovedPredictedVisuals
```

---

## Authoring Debug

Show validation failures:

```text
missing socket
invalid local axis
zero distance
unreachable hand target
object insert tip mismatch
moving part travel invalid
mirrored preview invalid
sequence step references missing interaction point
```

---

## LOD Debug

Show:

```text
InteractionLOD
EnabledSolvers
DisabledSolvers
FingerSolvingEnabled
ObjectPathDetail
MovingPartDetail
RemoteSimplificationLevel
```

---

## Console Commands

Recommended command names:

```text
wpn.DebugInteraction 0|1
wpn.DebugContacts 0|1
wpn.DebugConstraints 0|1
wpn.DebugObjects 0|1
wpn.DebugMirroring 0|1
wpn.DebugNetwork 0|1
wpn.DebugPrediction 0|1
wpn.DebugLOD 0|1
```

---

## Debug Overlay Rule

The overlay should always distinguish:

```text
canonical state
presented/mirrored state
predicted state
authoritative state
```

Most weapon interaction bugs come from confusing these states.

---

## Final Formula

```text
Weapon interaction debugging =
  visible pose lifecycle
  + contact quality
  + constraint priority
  + canonical/presented targets
  + object/moving part state
  + prediction/authority comparison
  + authoring validation errors.
```
