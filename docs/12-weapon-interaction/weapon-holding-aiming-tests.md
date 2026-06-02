---
id: weapon-holding-aiming-and-fire-tests
title: Weapon Holding Aiming and Fire Tests
status: draft
version: 26.602.1431
tags: [ weapon, tests, holding, aiming, fire, ads, hip-fire ]
---

# Weapon Holding Aiming and Fire Tests

## Purpose

This document defines tests and acceptance criteria for normal weapon-in-hands behavior:

```text
holding
low ready
hip fire
point aim
aim down sights
pose transitions
fire validation
reload overlay
remote fire reconstruction
```

Reload-specific tests are defined in [Weapon Interaction Tests and Acceptance Criteria](./weapon-interaction-tests.md).

---

## General Rules

All tests must verify:

```text
pose state is explicit
fire permission uses pose + mechanical + reload state
client visuals do not authoritatively change gameplay
remote clients reconstruct state from replicated pose/fire data
reload overlays current pose and exits to a valid pose
```

---

## Test 1: Low Ready To Hip Fire

Setup:

```text
weapon equipped
pose = LowReady
player presses fire or raise-to-fire input
mechanical state allows fire
```

Expected:

```text
pose transitions to HipFire or fires after allowed raise transition
weapon remains stabilized
muzzle moves toward aim direction
server validates fire permission
```

Fail if:

```text
shot fires while pose policy blocks fire
weapon fires without mechanical/chamber validation
hand IK snaps instantly to hip fire pose
```

---

## Test 2: Hip Fire

Setup:

```text
pose = HipFire
chambered round available
support grip active for rifle
```

Expected:

```text
fire allowed
muzzle direction plausibly follows camera/controller aim
spread/recoil policy is hip-fire policy
server owns ammo consumption and hit/projectile authority
```

Fail if:

```text
client consumes ammo authoritatively
muzzle points far away from gameplay fire direction
support hand drifts away without action plan
```

---

## Test 3: Hip Fire To ADS

Setup:

```text
pose = HipFire
player holds ADS input
movement state allows ADS
```

Expected:

```text
DesiredPose = AimDownSights
PoseAlpha blends over configured duration
sight alignment error decreases
movement freedom reduces
fire policy changes to ADS policy when transition threshold allows
```

Fail if:

```text
ADS snaps instantly without transition
sight alignment does not converge
server and client disagree on pose state
```

---

## Test 4: ADS Fire

Setup:

```text
pose = AimDownSights
sight alignment within tolerance
mechanical state allows fire
```

Expected:

```text
fire allowed
ADS spread/recoil policy used
local muzzle flash/recoil predicted
server validates and replicates fire state
remote clients reconstruct fire visual
```

Fail if:

```text
ADS fire uses hip-fire policy accidentally
remote clients need per-frame IK to see fire event
server allows fire while weapon is mechanically unable
```

---

## Test 5: Sprint Blocks ADS and Fire

Setup:

```text
pose = AimDownSights or HipFire
character starts sprinting
weapon policy does not allow sprint fire
```

Expected:

```text
pose transitions to SprintingWithWeapon or LowReady
ADS exits
fire blocked
reload interrupted or blocked according to policy
```

Fail if:

```text
character remains in ADS while sprinting without policy support
server accepts fire during blocked sprint state
```

---

## Test 6: Reload From ADS Returns To Valid Pose

Setup:

```text
pose = AimDownSights
reload starts
reload completes
movement state still allows ADS
```

Expected:

```text
PreviousPose = AimDownSights
pose becomes Reloading overlay
reload plan executes
pose returns to AimDownSights or configured exit pose
```

Fail if:

```text
reload permanently resets pose to arbitrary state
previous pose is lost
ADS hand/contact state is invalid after reload
```

---

## Test 7: Reload From ADS Falls Back When ADS Invalid

Setup:

```text
pose = AimDownSights
reload starts
character begins sprinting or enters state that blocks ADS
reload completes or interrupts
```

Expected:

```text
system does not return to invalid ADS
fallback order chooses HipFire, LowReady, or SprintingWithWeapon depending on context
pose transition is replicated
```

Fail if:

```text
system restores ADS even though movement state blocks ADS
client and server diverge on final pose
```

---

## Test 8: Server Rejects Fire During Reload Before Commit

Setup:

```text
reload active
MagazineLocked not committed
weapon policy = CannotFireDuringReload or blocked before commit
client presses fire
```

Expected:

```text
local fire visual may be suppressed or predicted then rejected
server rejects fire
no ammo consumed by client authority
reload continues or recovers according to policy
```

Fail if:

```text
client fires authoritatively
server mechanical state ignored
reload and fire states conflict permanently
```

---

## Test 9: Fire After Reload Commit Policy

Setup:

```text
reload active
MagazineLocked committed
weapon policy = CanFireAfterCommit
pose stability is valid
client presses fire
```

Expected:

```text
server allows fire if mechanical state and stability allow
reload either continues visually, cancels, or transitions according to policy
fire state replicates
```

Fail if:

```text
fire is always blocked despite policy
fire always allowed regardless of stability/mechanical state
```

---

## Test 10: Remote Fire Reconstruction

Setup:

```text
remote client receives replicated FireState
remote client did not see StartFire input
```

Expected:

```text
remote plays muzzle flash/sound/recoil animation from FireSequenceId and LastFireServerTime
remote does not need per-frame hand IK replication
remote uses replicated pose state for broad visual context
```

Fail if:

```text
remote fire visual depends on owning client's local-only variables
remote misses fire event because no per-frame IK was replicated
```

---

## Test 11: Pose Replication Late Relevancy

Setup:

```text
remote client becomes relevant while character is ADS or HipFire
```

Expected:

```text
remote receives PoseState
computes pose alpha from server time
shows correct broad pose
```

Fail if:

```text
remote defaults to LowReady until next input event
remote cannot reconstruct current pose without history
```

---

## Test 12: Shoulder Switch While Holding Weapon

Setup:

```text
pose = HipFire or ADS
player switches shoulder side
```

Expected:

```text
ShoulderSide replicates
hand/contact roles update through holding solver
weapon pose transitions without blind negative-scale mirroring
aim remains plausible
```

Fail if:

```text
socket axes are mirrored incorrectly
main/support hand assumptions hard-code one side
remote clients show wrong shoulder side
```

---

## Test 13: Weapon Switch Clears Pose/Aim/Interaction State

Setup:

```text
weapon active
pose = ADS or Reloading
player switches weapon
```

Expected:

```text
old weapon pose/aim/reload visual state clears
new weapon initializes valid pose
stale IK targets are disabled
server owns final equipped weapon and pose state
```

Fail if:

```text
old weapon hand targets remain active
old reload visual persists
client keeps firing old weapon locally
```

---

## MVP Acceptance Set

MVP weapon-in-hands acceptance requires:

```text
LowReady to HipFire
HipFire fire
HipFire to ADS
ADS fire
Sprint blocks ADS/fire
Reload from ADS returns or falls back correctly
Remote fire reconstruction
Pose replication late relevancy
Weapon switch clears state
```

---

## Final Formula

```text
Weapon-in-hands acceptance =
  explicit pose state
  + valid aim/fire permission
  + server authority
  + reload overlay integration
  + remote visual reconstruction
  + clean state transitions.
```
