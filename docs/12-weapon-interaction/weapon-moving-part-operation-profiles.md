---
id: weapon-moving-part-operation-profiles
title: Weapon Moving Part Operation Profiles
status: draft
version: 26.602.2053
tags: [ weapon, interaction, moving-parts, mechanisms, animation ]
---

# Weapon Moving Part Operation Profiles

## Purpose

This document defines common operation profiles for weapon moving parts.

Moving parts include:

```text
bolt
slide
pump
charging handle
lever
hinge
latch
button
selector/safety
cover/dust cover
```

This is visual/procedural mechanism interaction, not a full firearm simulation.

---

## Scope Boundary

This document owns:

```text
moving part grip phase
travel axis/path
hand follow rule
operation commit point
visual return behavior
failure/recovery
```

It does not own:

```text
ammo economy
ballistics
damage
fire backend
real firearm mechanical simulation
```

---

## Core Rule

```text
A moving part operation is an authored interaction profile: grip, move along semantic path, commit, settle or return.
```

The hand does not physically simulate the mechanism. Runtime drives the moving part state and the hand follows the authored target when appropriate.

---

## Common Operation Profiles

```text
LinearPull
LinearPush
PullAndRelease
ReciprocatingSlide
PumpCycle
RotatingLever
BoltCycle
BreakActionHinge
ToggleSwitch
PressButton
LatchOpenClose
CoverOpenClose
```

---

## Linear Pull / Push

Used for:

```text
charging handle
simple bolt handle pull
side handle
```

Phases:

```text
Approach → Grip → OperatePull/Push → Commit → Release/Settle
```

Authored data:

```text
FollowSocket
LocalTravelAxis
TravelDistance
RestPosition
PulledPosition
CommitAlpha
```

---

## Pull And Release

Used when the part returns automatically after release.

Examples:

```text
charging handle pulled and released
spring-return latch
```

Phases:

```text
Grip → Pull → Commit → Release → PartReturns → HandRecover
```

Rules:

```text
hand follows part during pull
hand may release before part returns
part return visual may be procedural or authored animation
```

---

## Reciprocating Slide

Used for:

```text
pistol slide
some firearm charging motions
```

Phases:

```text
Approach → GripSlide → PullBack → Commit → Release → SlideForward → Recover
```

Rules:

```text
support/manipulation hand grips slide using appropriate grip pose
slide travel axis is authored locally
main grip must remain stable
hand releases before slide returns unless profile says follow forward
```

---

## Pump Cycle

Used for pump-action weapons.

Phases:

```text
GripPump → PumpBack → PumpBackCommit → PumpForward → PumpForwardCommit → Settle
```

Rules:

```text
pump grip is both support contact and moving part contact
hand follows pump throughout the cycle
weapon stability may degrade if pump hand is also main support
shoulder/main grip contacts must keep weapon controlled
```

---

## Bolt Cycle

Bolt-action may combine rotation and translation.

Phases:

```text
GripBolt → RotateUnlock → PullBack → CommitOpen → PushForward → RotateLock → CommitClosed → Release
```

Authored data may include:

```text
BoltKnobSocket
UnlockRotationAxis
UnlockRotationAngle
PullAxis
PullDistance
LockRotationAxis
LockRotationAngle
```

Rules:

```text
do not model bolt action as simple linear pull if the authored weapon requires rotation
support/shoulder contact must keep weapon stable if main hand leaves grip
```

---

## Rotating Lever

Used for:

```text
lever action
selector lever
folding latch
```

Phases:

```text
Grip → RotateOpen → Commit → RotateClose/Settle
```

Authored data:

```text
PivotSocket
RotationAxis
RotationAngle
CommitAngle
```

---

## Break-Action Hinge

Used for break-action weapons.

Phases:

```text
Grip/Support → UnlockLatch → RotateOpen → CommitOpen → Insert/ExtractObjects → RotateClosed → CommitClosed → Settle
```

Rules:

```text
hinge angle is a moving part state
object insertion may happen while hinge is open
support contact must keep weapon controlled while open
```

---

## Button / Toggle / Safety

Used for small controls.

Phases:

```text
Approach → Press/Toggle → Commit → Release
```

Rules:

```text
may use finger-specific pose if available
MVP can approximate with hand/finger alpha
commit may happen at press threshold
```

---

## Operation Data

Suggested conceptual data:

```text
MovingPartOperationProfile:
  OperationId
  OperationType
  MovingPartName
  FollowSocket
  GripPoseId
  LocalAxis optional
  TravelDistance optional
  PivotAxis optional
  RotationAngle optional
  CommitAlpha
  ReturnPolicy
  RecoveryPolicy
```

---

## Return Policies

```text
StayAtCommittedPosition
ReturnAfterRelease
FollowHandBack
SnapWhenHidden
BlendToAuthorityState
```

---

## Failure And Recovery

Failures:

```text
missing moving part bone/socket
invalid travel axis
hand cannot reach follow socket
moving part state conflicts with authority
operation interrupted before commit
operation interrupted after commit
```

Recovery:

```text
return part to rest before commit
settle to committed state after commit
blend to authority moving part state
clear stale hand follow target
```

---

## Debug Requirements

Debug should show:

```text
operation id
operation type
moving part name
phase
alpha
local axis/world axis
rotation axis
travel distance
commit alpha
return policy
hand follow socket
failure reason
```

---

## Final Formula

```text
Moving part operation =
  authored grip
  + semantic travel/rotation path
  + commit point
  + hand follow rule
  + return/recovery policy.
```
