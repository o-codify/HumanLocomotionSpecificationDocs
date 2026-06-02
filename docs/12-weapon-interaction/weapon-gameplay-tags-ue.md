---
id: weapon-gameplay-tags-for-unreal-engine
title: Weapon Gameplay Tags for Unreal Engine
status: draft
version: 26.602.2053
tags: [ weapon, unreal-engine, ue5.7, gameplay-tags, implementation ]
---

# Weapon Gameplay Tags for Unreal Engine

## Purpose

This document defines Gameplay Tag naming conventions for the weapon interaction system in Unreal Engine.

Gameplay Tags are used for reload sequence ids, step ids, commit points, rejection reasons, recovery policies, grip poses, contact states, weapon policies, and debug classification.

The runtime usage is defined in [Weapon Runtime Implementation for Unreal Engine](./weapon-runtime-implementation-ue.md). Planner rejection tags are used by [Weapon Reload Planner for Unreal Engine](./weapon-reload-planner-ue.md).

---

## Naming Principles

Tags should be:

```text
hierarchical
stable
readable
specific enough for debugging
not tied to one weapon asset unless intentionally asset-specific
```

Avoid using plain strings for gameplay-critical state.

---

## Root Namespace

Use:

```text
Weapon
```

Primary branches:

```text
Weapon.Reload
Weapon.Interaction
Weapon.Mechanical
Weapon.Policy
Weapon.Reject
Weapon.Recovery
Weapon.GripPose
Weapon.Contact
Weapon.Debug
```

---

## Reload Sequence Tags

Sequence ids:

```text
Weapon.Reload.Sequence.Full
Weapon.Reload.Sequence.Tactical
Weapon.Reload.Sequence.Emergency
Weapon.Reload.Sequence.LoadOne
Weapon.Reload.Sequence.CycleOnly
Weapon.Reload.Sequence.Unload
```

Weapon-specific sequences may extend:

```text
Weapon.Reload.Sequence.Rifle.BottomMagazine.Full
Weapon.Reload.Sequence.Shotgun.Tube.LoadOne
Weapon.Reload.Sequence.Pistol.SlideLock.Full
```

Use generic sequence tags where possible. Use weapon-specific tags only when the sequence structure is genuinely different.

---

## Reload Step Tags

Step ids:

```text
Weapon.Reload.Step.PreparePose
Weapon.Reload.Step.Regrip
Weapon.Reload.Step.ReachMagazine
Weapon.Reload.Step.GripMagazine
Weapon.Reload.Step.ExtractMagazine
Weapon.Reload.Step.DropMagazine
Weapon.Reload.Step.StowMagazine
Weapon.Reload.Step.FetchMagazine
Weapon.Reload.Step.AlignMagazine
Weapon.Reload.Step.InsertMagazine
Weapon.Reload.Step.LockMagazine
Weapon.Reload.Step.OperateBolt
Weapon.Reload.Step.OperateSlide
Weapon.Reload.Step.OperatePumpBack
Weapon.Reload.Step.OperatePumpForward
Weapon.Reload.Step.ReturnHand
Weapon.Reload.Step.RestorePose
Weapon.Reload.Step.Recovery
```

---

## Commit Point Tags

Commit points are server-authoritative gameplay transitions.

```text
Weapon.Reload.Commit.MagazineDetached
Weapon.Reload.Commit.MagazineLocked
Weapon.Reload.Commit.RoundInserted
Weapon.Reload.Commit.RoundChambered
Weapon.Reload.Commit.ShellInserted
Weapon.Reload.Commit.BoltOpened
Weapon.Reload.Commit.BoltClosed
Weapon.Reload.Commit.SlideRacked
Weapon.Reload.Commit.PumpBack
Weapon.Reload.Commit.PumpForward
Weapon.Reload.Commit.ReloadCompleted
```

Native tag constants should use clear names:

```cpp
TAG_Weapon_Reload_Commit_MagazineDetached
TAG_Weapon_Reload_Commit_MagazineLocked
TAG_Weapon_Reload_Commit_RoundChambered
TAG_Weapon_Reload_Commit_BoltOpened
TAG_Weapon_Reload_Commit_BoltClosed
```

---

## Rejection Tags

Planner and server rejection reasons:

```text
Weapon.Reload.Reject.NoWeapon
Weapon.Reload.Reject.NoProfile
Weapon.Reload.Reject.InvalidProfile
Weapon.Reload.Reject.Busy
Weapon.Reload.Reject.NoAmmo
Weapon.Reload.Reject.InvalidMechanicalState
Weapon.Reload.Reject.CharacterStateBlocked
Weapon.Reload.Reject.NoReachableBodySlot
Weapon.Reload.Reject.NoStableHandAssignment
Weapon.Reload.Reject.NoReachableInteractionPoint
Weapon.Reload.Reject.SequenceInvalid
Weapon.Reload.Reject.InventoryMismatch
Weapon.Reload.Reject.ServerStateChanged
```

Rejection tags must be stored in planner results for debug and optional UI.

---

## Recovery Policy Tags

```text
Weapon.Recovery.ReturnMainHandFirst
Weapon.Recovery.ReturnSupportHandFirst
Weapon.Recovery.DropHeldObject
Weapon.Recovery.StowHeldObject
Weapon.Recovery.CompleteCriticalStep
Weapon.Recovery.ResetMechanism
Weapon.Recovery.RestorePoseOnly
Weapon.Recovery.CancelPredictedVisual
```

Recovery policy is chosen by planner/server and visualized by animation execution.

---

## Mechanical State Tags

Mechanical state is primarily stored as replicated fields, not only tags.

Tags may be used for events/debug/policies:

```text
Weapon.Mechanical.MagazineInserted
Weapon.Mechanical.MagazineLocked
Weapon.Mechanical.ChamberLoaded
Weapon.Mechanical.ChamberEmpty
Weapon.Mechanical.BoltOpen
Weapon.Mechanical.BoltClosed
Weapon.Mechanical.NeedsCycle
Weapon.Mechanical.PumpBack
Weapon.Mechanical.PumpForward
```

Do not replace replicated mechanical state fields with only tags unless the system is intentionally tag-driven.

---

## Weapon Policy Tags

```text
Weapon.Policy.CannotFireDuringReload
Weapon.Policy.CanFireAfterCommit
Weapon.Policy.CanFireWithPenaltyAfterCommit
Weapon.Policy.CanFireOnlyWhenStableHoldRestored
Weapon.Policy.RequiresCycleAfterMagazineLock
Weapon.Policy.DropOldMagazineOnEmergencyReload
Weapon.Policy.StowOldMagazineOnTacticalReload
Weapon.Policy.AllowTacticalToEmergencyFallback
```

Policies should be stored in weapon reload policy data and checked by planner/server.

---

## Contact State Tags

```text
Weapon.Contact.None
Weapon.Contact.Approaching
Weapon.Contact.PreGrip
Weapon.Contact.Contact
Weapon.Contact.VisualAttached
Weapon.Contact.GameplayAttached
Weapon.Contact.Manipulating
Weapon.Contact.Released
Weapon.Contact.Stabilizing
```

These may be used by `FWeaponHandIKTarget.ContactState`.

---

## Grip Pose Tags

```text
Weapon.GripPose.None
Weapon.GripPose.Rifle.MainGrip
Weapon.GripPose.Rifle.SupportGrip
Weapon.GripPose.Pistol.MainGrip
Weapon.GripPose.Pistol.TwoHandSupport
Weapon.GripPose.Magazine.Standard
Weapon.GripPose.Round.Pinched
Weapon.GripPose.Shell.Pinched
Weapon.GripPose.Bolt.Pinch
Weapon.GripPose.Pump.ForeEnd
```

Grip pose tags select hand/finger pose presets or Control Rig finger profiles.

---

## Interaction Point Tags Optional

Interaction points are usually referenced by `FName` because they map to sockets/profile entries.

Tags may be used for generic categories:

```text
Weapon.Interaction.Point.MagazineWell
Weapon.Interaction.Point.Bolt
Weapon.Interaction.Point.Slide
Weapon.Interaction.Point.Pump
Weapon.Interaction.Point.ShellInsert
Weapon.Interaction.Point.Chamber
```

---

## Native Tag Declaration Pattern

Recommended C++ pattern:

```cpp
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Weapon_Reload_Commit_MagazineLocked);
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Weapon_Reload_Reject_NoAmmo);
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Weapon_Recovery_ReturnMainHandFirst);
```

Implementation file:

```cpp
UE_DEFINE_GAMEPLAY_TAG(TAG_Weapon_Reload_Commit_MagazineLocked, "Weapon.Reload.Commit.MagazineLocked");
UE_DEFINE_GAMEPLAY_TAG(TAG_Weapon_Reload_Reject_NoAmmo, "Weapon.Reload.Reject.NoAmmo");
UE_DEFINE_GAMEPLAY_TAG(TAG_Weapon_Recovery_ReturnMainHandFirst, "Weapon.Recovery.ReturnMainHandFirst");
```

Keep tag names centralized. Do not scatter literal tag strings through planner/runtime code.

---

## Validation Rules

Validation should check:

```text
reload sequence id tag is valid
step id tags are valid
commit tags are valid for commit policies
recovery tags are valid for interruptible steps
policy tags are known by weapon reload policy
rejection tags are from approved namespace
```

---

## Debug Requirements

Debug output should print tags for:

```text
selected sequence
current step
current commit point
rejection reasons
recovery policy
weapon policy
contact state
grip pose
```

Tags are easier to inspect than raw enum/integer state in multiplayer logs.

---

## Final Formula

```text
Weapon Gameplay Tags =
  stable names for reload sequence
  + step identity
  + commit points
  + rejection reasons
  + recovery policy
  + weapon policy
  + contact/grip visualization.
```
