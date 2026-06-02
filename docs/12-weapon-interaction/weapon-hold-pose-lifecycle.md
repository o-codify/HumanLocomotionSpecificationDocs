---
id: weapon-hold-pose-lifecycle
title: Weapon Hold Pose Lifecycle
status: draft
version: 26.602.2053
tags: [ weapon, interaction, hold-pose, lifecycle, stability ]
---

# Weapon Hold Pose Lifecycle

## Purpose

This document defines how a weapon hold pose is entered, stabilized, maintained, degraded, recovered, and exited.

Pose states such as `LowReady`, `HipFire`, and `AimDownSights` are not just labels. Each pose has a lifecycle.

---

## Scope Boundary

This document describes the lifecycle of weapon/hand/contact poses only.

It does not own locomotion speed, body yaw, camera placement, inventory, damage, or projectile behavior.

---

## Lifecycle Phases

Recommended phases:

```text
Inactive
Entering
Stabilizing
Stable
Degrading
TemporarilyDisrupted
Recovering
Exiting
Failed
```

---

## Inactive

No weapon hold pose is active.

Examples:

```text
unarmed
weapon attached to body visual socket
weapon not yet in hands
```

---

## Entering

The character begins moving into a weapon hold pose.

Typical work:

```text
main hand reaches grip
weapon aligns with hand/contact target
support hand starts moving toward support grip
weapon pose offset blends in
IK targets become active
```

Output:

```text
DesiredPoseState
PoseStartTime
PoseBlendDuration
initial contact targets
```

---

## Stabilizing

The hold pose exists but is not fully stable yet.

Requirements:

```text
main grip quality rising
support contact approaching/pre-grip if required
shoulder contact settling if required
weapon pose offset settling
```

A pose may visually look correct but still be in `Stabilizing` phase.

---

## Stable

The hold pose satisfies its required contact quality thresholds.

Examples:

```text
LowReady stable
HipFire stable
ADS stable
```

Stable pose can be used as:

```text
entry pose for reload
entry pose for mechanism manipulation
return pose after interruption
baseline for aim/fire visual readiness
```

---

## Degrading

The pose is losing quality without an explicit action plan release.

Examples:

```text
support grip quality drops
shoulder contact slips
weapon pose offset grows too large
hand target becomes unreachable
mirrored presentation creates target mismatch
```

Degrading pose should request recovery before becoming invalid.

---

## Temporarily Disrupted

The pose is intentionally disrupted by an action plan.

Examples:

```text
support hand leaves support grip for reload
ADS alignment degrades during magazine insertion
shoulder contact becomes partial during mechanism manipulation
object path takes priority over perfect hold pose
```

Temporary disruption must remember:

```text
EntryPoseState
EntryContactSet
AllowedBrokenContacts
RecoveryTargetPose
MaxDisruptionDuration or step range
```

---

## Recovering

The system attempts to restore a valid hold pose.

Recovery may target:

```text
previous stable pose
configured exit pose
fallback pose
compact safe pose
```

Example fallback order:

```text
PreviousStablePose
HipFire
LowReady
RelaxedCarry
Failed
```

---

## Exiting

The hold pose is intentionally ending.

Examples:

```text
holster starts
weapon switches out
interaction clears
weapon becomes external visual object
```

Exit should clear or fade:

```text
hand IK targets
weapon pose offsets
object visual attachment if no longer relevant
fire visual pose impulse
contact quality state
```

---

## Failed

The pose cannot be entered, maintained, or recovered.

Failure should produce:

```text
FailureReason
BlockingConstraint
LastStablePose
RecoveryAttempted
DebugData
```

---

## Pose Lifecycle State

Suggested conceptual state:

```text
WeaponHoldPoseLifecycleState:
  PoseState
  LifecyclePhase
  EntryPoseState
  PreviousStablePoseState
  DesiredExitPoseState
  PoseStartTime
  PoseBlendDuration
  HoldStabilityScore
  ActiveContacts
  BrokenContacts
  RecoveryTarget
  FailureReason
```

---

## Reload Overlay Example

```text
ADS Stable
→ Reloading TemporarilyDisrupted
→ support hand released by plan
→ MagazineLocked commit
→ support hand recovering
→ ADS Stable if valid
→ HipFire or LowReady fallback if ADS invalid
```

---

## Draw Example

```text
Inactive
→ Entering LowReady
→ main hand reaches grip
→ weapon detaches from body visual socket
→ support hand joins
→ Stabilizing LowReady
→ Stable LowReady
```

---

## Lifecycle And Networking

Replicate lifecycle state only when needed for reconstruction.

Useful replicated concepts:

```text
PoseState
LifecyclePhase
PoseStartServerTime
PoseBlendDuration
PreviousStablePoseState
RecoveryTargetPose
Revision
```

Do not replicate per-frame IK.

---

## Lifecycle And Mirroring

Lifecycle state is canonical.

Presentation mirroring affects final transforms, not lifecycle semantics.

Correct:

```text
Canonical PoseState = ADS
LifecyclePhase = Recovering
PresentationMirror = true
```

Incorrect:

```text
MirroredADSRecovering as a separate gameplay lifecycle state
```

---

## Debug Requirements

Debug should show:

```text
pose state
lifecycle phase
previous stable pose
entry pose
desired exit pose
hold stability score
active contacts
broken contacts
recovery target
phase time
failure reason
```

---

## Final Formula

```text
Hold pose lifecycle =
  enter
  + stabilize
  + maintain
  + temporarily disrupt
  + recover
  + exit
  + fail cleanly when constraints cannot be restored.
```
