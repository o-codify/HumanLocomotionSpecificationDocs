---
id: weapon-holding-aiming-and-fire-tests
title: Weapon Holding Aiming and Fire Tests
status: draft
version: 26.602.1455
tags: [ weapon, tests, holding, aiming, fire, ads, hip-fire ]
---

# Weapon Holding Aiming and Fire Tests

## Purpose

This document defines tests and acceptance criteria for weapon-in-hands interaction behavior:

```text
holding
low ready
hip fire
point aim
aim down sights
pose transitions
interaction fire readiness
fire visual response
reload overlay
remote fire visual reconstruction
```

Reload-specific tests are defined in [Weapon Interaction Tests and Acceptance Criteria](./weapon-interaction-tests.md). Scope boundaries are defined in [Weapon Interaction Boundaries](./weapon-interaction-boundaries.md).

This document does not test projectile simulation, damage, ammo economy, camera implementation, locomotion speed, or the full external fire backend.

---

## General Rules

All tests must verify:

```text
pose state is explicit
fire readiness uses weapon interaction state
client visuals do not authoritatively change external gameplay state
remote clients reconstruct interaction visuals from replicated pose/fire-visual data
reload overlays current pose and exits to a valid weapon pose
```

---

## Test 1: Low Ready To Hip Fire

Setup:

```text
weapon equipped
pose = LowReady
external input requests raise-to-fire or hip-fire pose
external backend may request fire readiness
```

Expected:

```text
pose transitions to HipFire or starts allowed raise transition
weapon remains stabilized
muzzle moves toward external aim direction
interaction fire readiness reports whether pose is ready
```

Fail if:

```text
weapon visually fires while interaction pose policy blocks fire readiness
hand IK snaps instantly to hip fire pose
pose transition bypasses interaction state
```

---

## Test 2: Hip Fire Interaction Pose

Setup:

```text
pose = HipFire
support grip active for rifle
external aim intent is valid
```

Expected:

```text
interaction fire readiness can become ready
muzzle direction plausibly follows external aim intent
fire visual response can be played if external fire backend confirms/predicts fire
support hand remains stable unless released by another interaction
```

Fail if:

```text
weapon interaction consumes ammo or decides damage
muzzle points far away from external aim direction
support hand drifts away without action plan
```

---

## Test 3: Hip Fire To ADS

Setup:

```text
pose = HipFire
external input requests ADS pose
external locomotion/body systems do not block ADS pose request
```

Expected:

```text
DesiredPose = AimDownSights
PoseAlpha blends over configured duration
sight alignment error decreases
interaction pose becomes ADS-ready when alignment/contact rules pass
```

Fail if:

```text
ADS snaps instantly without transition
sight alignment does not converge
server and client disagree on weapon pose state
weapon interaction defines locomotion speed or body yaw directly
```

---

## Test 4: ADS Fire Visual Response

Setup:

```text
pose = AimDownSights
sight alignment within interaction tolerance
external fire backend confirms or predicts a fire visual event
```

Expected:

```text
interaction fire readiness is ready before fire visual event
ADS fire visual pose impulse is used
local fire visual response plays
remote clients reconstruct fire visual response
```

Fail if:

```text
ADS fire visual uses hip-fire interaction pose by mistake
remote clients need per-frame IK to see fire visual event
weapon interaction decides projectile/hit/damage outcome
```

---

## Test 5: Sprint Weapon Pose Blocks ADS Interaction Pose

Setup:

```text
pose = AimDownSights or HipFire
external locomotion state indicates sprinting
weapon interaction policy does not allow ADS while sprint weapon pose is active
```

Expected:

```text
weapon pose transitions to SprintingWithWeapon or another valid weapon pose
ADS interaction pose exits
interaction fire readiness reports blocked or not-ready
reload/manipulation may be interrupted or blocked according to interaction policy
```

Fail if:

```text
weapon remains in ADS interaction pose while external state blocks it
weapon interaction computes locomotion speed itself
```

---

## Test 6: Reload From ADS Returns To Valid Weapon Pose

Setup:

```text
pose = AimDownSights
reload starts
reload completes
ADS weapon pose is still allowed by external systems
```

Expected:

```text
PreviousPose = AimDownSights
pose becomes Reloading overlay
reload plan executes
pose returns to AimDownSights or configured exit weapon pose
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
external state later blocks ADS weapon pose
reload completes or interrupts
```

Expected:

```text
system does not return to invalid ADS weapon pose
fallback order chooses HipFire, LowReady, SprintingWithWeapon, or another allowed weapon pose
pose transition is replicated
```

Fail if:

```text
system restores ADS even though external systems block ADS
client and server diverge on final weapon pose
weapon interaction owns the external blocking system
```

---

## Test 8: Fire Readiness During Reload Before Commit

Setup:

```text
reload active
required interaction commit is not reached
external fire backend asks for interaction fire readiness
```

Expected:

```text
interaction fire readiness reports blocked/not-ready
local fire visual may be suppressed
external backend remains responsible for final fire acceptance
reload continues or recovers according to interaction policy
```

Fail if:

```text
weapon interaction authoritatively fires a shot
weapon interaction consumes ammo or applies damage
reload and fire visual states conflict permanently
```

---

## Test 9: Fire Readiness After Reload Commit Policy

Setup:

```text
reload active
required interaction commit reached
weapon interaction policy = CanRequestFireAfterInteractionCommit
pose stability is valid
external fire backend asks for interaction fire readiness
```

Expected:

```text
interaction fire readiness can report ready
reload either continues visually, cancels visually, or transitions according to interaction policy
fire visual state can replicate if external backend confirms fire event
```

Fail if:

```text
interaction readiness is always blocked despite policy
interaction readiness is always allowed regardless of stability/interaction state
```

---

## Test 10: Remote Fire Visual Reconstruction

Setup:

```text
remote client receives replicated FireVisualState
remote client did not see local fire input
```

Expected:

```text
remote plays interaction fire visual response from FireSequenceId and LastFireVisualServerTime
remote does not need per-frame hand IK replication
remote uses replicated pose state for broad visual context
```

Fail if:

```text
remote fire visual depends on owning client's local-only variables
remote misses fire visual event because no per-frame IK was replicated
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
shows correct broad weapon pose
```

Fail if:

```text
remote defaults to LowReady until next input event
remote cannot reconstruct current weapon pose without history
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
aim interaction remains plausible
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
external equipment system switches weapon
```

Expected:

```text
old weapon pose/aim/reload visual state clears
new weapon initializes valid interaction pose
stale IK targets are disabled
server owns final equipped weapon reference outside this interaction layer
```

Fail if:

```text
old weapon hand targets remain active
old reload visual persists
weapon interaction implements full equipment system here
```

---

## MVP Acceptance Set

MVP weapon-in-hands interaction acceptance requires:

```text
LowReady to HipFire
HipFire interaction fire readiness
HipFire to ADS
ADS fire visual response
Sprint weapon pose blocks ADS interaction pose
Reload from ADS returns or falls back correctly
Remote fire visual reconstruction
Pose replication late relevancy
Weapon switch clears interaction state
```

---

## Final Formula

```text
Weapon-in-hands interaction acceptance =
  explicit weapon pose state
  + valid interaction fire readiness
  + external backend boundaries
  + reload overlay integration
  + remote visual reconstruction
  + clean interaction state transitions.
```
