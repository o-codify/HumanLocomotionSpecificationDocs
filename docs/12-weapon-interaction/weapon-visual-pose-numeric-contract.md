---
id: weapon-visual-pose-numeric-contract
title: Weapon Visual Pose Numeric Contract
status: draft
version: 26.603.1349
tags: [ weapon, interaction, visual-pose, ik, constraints, acceptance ]
---

# Weapon Visual Pose Numeric Contract

## Purpose

This document defines numeric visual acceptance rules for weapon holding and weapon manipulation poses.

Weapon interaction must not be accepted from vague descriptions such as:

```text
hand is on weapon
weapon is held correctly
magazine goes into weapon
```

Those are not implementation criteria. A game implementation needs measurable constraints for contact error, joint limits, reach, elbow pole stability, object alignment, frame-to-frame stability, and visual failure states.

This document is the HLS-level numeric contract that game-specific specs may reference.

---

## Scope Boundary

This document defines visual-pose correctness for weapon interaction:

```text
hand contact frames
weapon grip frames
wrist/elbow/shoulder constraints
elbow pole stability
arm reach/stretch limits
rifle/long-gun hold metrics
handgun hold metrics
magazine/object insertion metrics
object ownership visual correctness
floating/penetration thresholds
frame-to-frame stability
numeric debug output
```

It does not define:

```text
full biomechanical simulation
real firearm handling training
projectile simulation
damage
ammo economy
camera/FOV
locomotion speed
body yaw
turn-in-place
inventory
```

---

## Core Rule

```text
A weapon interaction pose is valid only if its numeric pose metrics are valid.
State correctness is not enough.
```

A pose may have the right state machine phase and still be invalid if:

```text
the wrist is twisted beyond limits
the elbow flips across frames
the hand floats away from the grip
the magazine enters the well at the wrong angle
the support hand is treated like a rifle foregrip on a handgun
the arm is fully stretched and marked stable
```

---

## Coordinate Frames

Every authored contact must define explicit frames.

Required frames:

```text
WeaponContactFrame_World
HandContactFrame_World
ExpectedHandContactFrame_World
```

For a grip contact:

```text
ExpectedHandContactFrame_World =
  WeaponSocketWorld * Inverse(HandGripOffsetLocal)
```

The solver must not compare only world positions. It must compare both position and orientation.

---

## Contact Error Metrics

For each contact:

```text
PositionErrorCm =
  Distance(HandContactFrame.Location, ExpectedHandContactFrame.Location)

RotationErrorDeg =
  AngleDelta(HandContactFrame.Rotation, ExpectedHandContactFrame.Rotation)

PalmNormalErrorDeg =
  AngleBetween(HandPalmNormal_World, ExpectedPalmNormal_World)

GripAxisErrorDeg =
  AngleBetween(HandGripAxis_World, ExpectedGripAxis_World)
```

Default thresholds:

| Metric | Stable hold | Reload/manipulation | Hard fail |
| --- | ---: | ---: | ---: |
| `PositionErrorCm` | `<= 3 cm` | `<= 5 cm` | `> 10 cm` |
| `RotationErrorDeg` | `<= 12°` | `<= 20°` | `> 35°` |
| `PalmNormalErrorDeg` | `<= 20°` | `<= 30°` | `> 45°` |
| `GripAxisErrorDeg` | `<= 15°` | `<= 25°` | `> 40°` |

A contact with errors above stable thresholds must not be marked `ActiveStable`. It must be `Partial`, `Recovering`, or `Failed` depending on severity.

---

## Arm Joint Metrics

Each solved arm must expose approximate joint metrics.

Required metrics:

```text
ShoulderPitchDeg
ShoulderYawDeg
ShoulderRollDeg
ElbowFlexDeg
ForearmTwistDeg
WristPitchDeg
WristYawDeg
WristRollDeg
```

Default limits:

| Joint metric | Preferred range | Soft limit | Hard fail |
| --- | ---: | ---: | ---: |
| `ElbowFlexDeg` | `25°..145°` | `10°..160°` | `< 5° or > 170°` |
| `WristPitchDeg` | `-35°..45°` | `-55°..65°` | `< -70° or > 80°` |
| `WristYawDeg` | `-25°..25°` | `-45°..45°` | `< -60° or > 60°` |
| `WristRollDeg` | `-45°..45°` | `-70°..70°` | `< -90° or > 90°` |
| `ForearmTwistDeg` | `-60°..60°` | `-90°..90°` | `< -120° or > 120°` |
| `ShoulderRollDeg` | `-70°..90°` | `-100°..120°` | outside soft by `> 25°` |

If a hand target requires hard-fail values, the solver must not accept the contact. It must request one of:

```text
weapon pose offset
clavicle/shoulder assist
different hand path
reload pose adjustment
fallback pose
```

---

## Elbow Pole Stability

For each arm:

```text
ElbowPoleVector = Normalize(ElbowPoleWorld - ShoulderWorld)
UpperArmVector = Normalize(ElbowWorld - ShoulderWorld)
ForearmVector = Normalize(HandWorld - ElbowWorld)
ElbowBendPlaneNormal = Normalize(Cross(UpperArmVector, ForearmVector))
PoleErrorDeg = AngleBetween(ElbowBendPlaneNormal, ExpectedBendPlaneNormal)
```

Acceptance:

```text
PoleErrorDeg <= 35° for stable hold
PoleErrorDeg <= 50° for transient reload/manipulation motion
FrameToFramePoleDeltaDeg <= 25° unless phase changed intentionally
```

Hard fail:

```text
elbow flips side across frames without phase transition
elbow pole causes the arm to bend backward through the torso
forearm crosses weapon in a way not authored by the profile
```

---

## Arm Reach And Stretch Limits

Measure reach from shoulder to hand target:

```text
ArmReachRatio = Distance(ShoulderWorld, HandTargetWorld) / MaxArmLength
```

Default limits:

```text
Preferred: 0.35..0.92
Soft:      0.25..0.98
Hard fail: > 1.03 or < 0.18
```

If:

```text
ArmReachRatio > 0.98
```

then the solver must request one of:

```text
weapon pose offset
clavicle/shoulder assist
different hand path
reload pose adjustment
fallback pose
```

It must not silently accept a fully stretched arm as stable.

---

## Rifle / Stocked Long Gun Metrics

For stocked rifles and long guns, validate:

```text
MainGripContact
SupportGripContact
StockShoulderContact optional/required by pose
SightEyeAlignment optional/required by ADS pose
```

Main hand requirements:

```text
MainGripPositionErrorCm <= 3
MainGripRotationErrorDeg <= 12
MainWristPitchDeg within preferred or soft limits
MainWristYawDeg within preferred or soft limits
MainWristRollDeg within preferred or soft limits
```

Support hand requirements:

```text
SupportGripPositionErrorCm <= 4
SupportGripRotationErrorDeg <= 18
SupportElbowFlexDeg between 25° and 150° preferred
SupportWristRollDeg within -60°..60° soft
```

Stock contact requirements:

```text
StockToShoulderDistanceCm <= 4 stable
StockToShoulderDistanceCm <= 7 partial
StockToShoulderDistanceCm > 10 fail for ADS if shoulder contact is required
StockPenetrationDepthCm <= 3
```

Sight/ADS presentation requirements:

```text
SightForwardErrorDeg <= 8° stable ADS presentation
SightForwardErrorDeg <= 15° partial ADS presentation
EyeReliefDistanceCm inside authored [MinEyeReliefCm, MaxEyeReliefCm] if defined
```

These values describe interaction/presentation quality. They do not decide projectile accuracy or damage.

---

## Handgun Metrics

For one-handed pistol:

```text
MainGripPositionErrorCm <= 3
MainGripRotationErrorDeg <= 12
PistolGripAxisErrorDeg <= 15
MainWristPitchDeg within preferred or soft limits
MainWristYawDeg within preferred or soft limits
MainWristRollDeg within preferred or soft limits
ElbowFlexDeg between 20° and 155° soft
```

For two-handed handgun support, validate the support hand relative to the main hand, not as a rifle foregrip.

```text
SupportRelativeFrameExpected =
  MainHandFrame * SupportHandToMainHandOffset

SupportRelativePositionErrorCm =
  Distance(SupportHandFrame.Location, SupportRelativeFrameExpected.Location)

SupportRelativeRotationErrorDeg =
  AngleDelta(SupportHandFrame.Rotation, SupportRelativeFrameExpected.Rotation)
```

Acceptance:

```text
SupportRelativePositionErrorCm <= 4 stable
SupportRelativeRotationErrorDeg <= 18 stable
SupportRelativePositionErrorCm > 8 fail
SupportRelativeRotationErrorDeg > 35 fail
```

A two-handed handgun pose is invalid if the support hand is placed on an arbitrary rifle-style `SupportGripSocket`, unless the weapon profile explicitly defines `SupportToWeaponFrameContact`.

---

## Magazine Reload Metrics

During magazine reload, validate hand and object.

Definitions:

```text
InsertTipWorld = ReloadObject.InsertTipSocket.WorldTransform
MagazineWellWorld = Weapon.MagazineWellSocket.WorldTransform
InsertAxisWorld = WeaponTransform.TransformVectorNoScale(MagazineInsertAxisLocal)
ObjectInsertAxisWorld = ObjectTransform.TransformVectorNoScale(ObjectInsertAxisLocal)
```

Metrics:

```text
InsertTipPositionErrorCm =
  Distance(InsertTipWorld.Location, MagazineWellWorld.Location)

InsertAxisErrorDeg =
  AngleBetween(ObjectInsertAxisWorld, InsertAxisWorld)

MagazineRotationErrorDeg =
  AngleDelta(ObjectExpectedRotation, ObjectCurrentRotation)

InsertionDepthAlpha =
  Project(ObjectPosition - PreInsertPosition, InsertAxisWorld) / InsertDistance
```

Acceptance by phase:

| Phase | Position | Axis | Rotation | Depth |
| --- | ---: | ---: | ---: | ---: |
| `PreAlign` | `<= 25 cm` | `<= 60°` | `<= 75°` | not required |
| `AxisAlign` | `<= 12 cm` | `<= 20°` | `<= 35°` | not required |
| `InsertTravel` | `<= 6 cm` | `<= 12°` | `<= 20°` | `0..1` increasing |
| `LockCommit` | `<= 3 cm` | `<= 8°` | `<= 12°` | `>= 0.95` |

Commit is forbidden if the `LockCommit` thresholds are not met.

---

## Object Ownership And Duplication Metrics

At every frame, a visual reload object must have exactly one semantic owner:

```text
Hidden
InWeapon
InHand
MovingToWeapon
MovingFromWeapon
DroppedWorld optional
Cleared optional
```

Hard fail:

```text
same magazine visible in hand and weapon as two uncontrolled objects
object state says InWeapon but object actor is still attached to hand
object state says InHand but weapon also shows inserted magazine without explicit hidden/alternate mesh policy
```

---

## Floating And Penetration Thresholds

Visual collision does not require full physics, but obvious visual failures must be measured.

```text
HandToGripFloatingDistanceCm <= 3 stable
HandToGripFloatingDistanceCm > 6 partial/fail depending on pose
HandToGripFloatingDistanceCm > 10 hard fail
MajorHandWeaponPenetrationCm <= 3 tolerated
MajorHandWeaponPenetrationCm > 6 fail
MagazineWeaponPenetrationOutsideWellCm > 5 fail
```

If mesh penetration is expensive to measure, use authored proxy shapes:

```text
palm capsule/box
forearm capsule
magazine body box
weapon grip box
magazine well box
stock box
```

---

## Frame-To-Frame Stability

No contact target should snap without a phase transition or correction event.

```text
HandTargetDeltaCmPerFrame <= 12 cm for normal update
HandTargetRotationDeltaDegPerFrame <= 35° for normal update
ObjectDeltaCmPerFrame <= 15 cm outside snap/commit events
```

If a larger change is required, mark it explicitly:

```text
PhaseTransition
AuthorityCorrection
SnapRequired
```

---

## Visual Pose Score

Each pose should compute a numeric score.

```text
ContactScore =
  1 - Clamp(PositionErrorCm / FailPositionCm, 0, 1)

RotationScore =
  1 - Clamp(RotationErrorDeg / FailRotationDeg, 0, 1)

JointScore =
  1 - Clamp(JointLimitViolation / JointFailMargin, 0, 1)

ReachScore =
  1 - Clamp(Max(ArmReachRatio - 0.92, 0) / 0.11, 0, 1)

ObjectScore =
  object-specific score or 1 if no object

VisualPoseScore = WeightedAverage(
  ContactScore * 0.30,
  RotationScore * 0.20,
  JointScore * 0.20,
  ReachScore * 0.15,
  ObjectScore * 0.15
)
```

Acceptance:

```text
VisualPoseScore >= 0.85 stable
VisualPoseScore >= 0.70 usable/transient
VisualPoseScore < 0.70 recovery required
VisualPoseScore < 0.50 fail or fallback
```

---

## Debug Output Requirements

Implementation must expose these values in debug overlay or log:

```text
CurrentPoseState
InteractionPhase
VisualPoseScore
PositionErrorCm per contact
RotationErrorDeg per contact
PalmNormalErrorDeg per contact
GripAxisErrorDeg per contact
Joint angles per arm
ArmReachRatio per arm
PoleErrorDeg per arm
GripPoseId per hand
VisualReloadObjectState
InsertAxisErrorDeg
InsertionDepthAlpha
CommitAllowed true/false
FailureReason
```

---

## First Milestone Acceptance

### M416-Style Rifle Hold And Reload

Accepted only if:

```text
MainGripPositionErrorCm <= 3
SupportGripPositionErrorCm <= 4
ElbowFlexDeg for both arms inside 10°..160° soft range
WristPitch/Yaw/Roll not hard-failing
StockToShoulderDistanceCm <= 7 when ADS/shoulder pose requires it
VisualPoseScore >= 0.85 during stable hold
Magazine LockCommit thresholds pass before commit
no duplicate magazine object exists
support hand returns with VisualPoseScore >= 0.80 after reload
```

### M9-Style Handgun Hold And Reload

Accepted only if:

```text
MainGripPositionErrorCm <= 3
MainGripRotationErrorDeg <= 12
if two-handed, SupportRelativePositionErrorCm <= 4
if two-handed, SupportRelativeRotationErrorDeg <= 18
no rifle-style fake support grip is required
magazine LockCommit thresholds pass before commit
VisualPoseScore >= 0.85 during stable hold
VisualPoseScore >= 0.80 after reload recovery
```

A result that only “looks close” but fails these metrics is not accepted.

---

## Final Formula

```text
Visual pose correctness =
  contact frame error
  + joint limit validity
  + elbow pole stability
  + reach/stretch validity
  + object insertion metrics
  + ownership correctness
  + floating/penetration thresholds
  + frame-to-frame stability
  + numeric debug visibility.
```
