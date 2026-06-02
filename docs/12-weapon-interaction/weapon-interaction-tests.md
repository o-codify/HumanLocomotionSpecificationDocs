---
id: weapon-interaction-tests-and-acceptance-criteria
title: Weapon Interaction Tests and Acceptance Criteria
status: draft
version: 26.602.1414
tags: [ weapon, tests, validation, acceptance, multiplayer, unreal-engine ]
---

# Weapon Interaction Tests and Acceptance Criteria

## Purpose

This document defines test scenarios and acceptance criteria for the weapon interaction system.

The goal is to prevent the implementation from drifting away from the design described in the weapon interaction documents.

These tests cover data validation, hand assignment, reload planning, animation execution, object lifecycle, networking, and UE integration.

---

## Test Categories

```text
profile validation tests
solver/planner tests
reload scenario tests
animation execution tests
object lifecycle tests
networking tests
editor preview tests
regression tests
```

---

## General Acceptance Rules

Every weapon interaction test must verify:

```text
no unsupported floating weapon
resolved hand is valid for current shoulder side
active stabilization contacts remain valid
movement uses authored local axes
gameplay state changes only at commit points
clients reconstruct visual phase from replicated state
authoritative server state wins over prediction
invalid authored data fails validation
```

---

## Test 1: Bottom Magazine Right Shoulder

Setup:

```text
RightShoulder
RightHand = MainGrip
LeftHand = SupportGrip
MagazineWell.AccessRegion = Bottom
Weapon has stock
```

Expected:

```text
LeftHand changes magazine
RightHand + shoulder stabilize weapon
magazine extracts along authored extract axis
magazine inserts along authored insert axis
MagazineLocked commit updates mechanical state
LeftHand returns to SupportGrip
```

Fail if:

```text
right hand releases main grip without regrip/stability support
insert direction uses point-to-point vector instead of socket axis
weapon loses all valid contacts
```

---

## Test 2: Bottom Magazine Left Shoulder

Setup:

```text
LeftShoulder
LeftHand = MainGrip
RightHand = SupportGrip
MagazineWell.AccessRegion = Bottom
Weapon has stock
```

Expected:

```text
RightHand changes magazine
LeftHand + shoulder stabilize weapon
same weapon profile works without separate left-shoulder reload asset
```

Fail if:

```text
system hard-codes left hand as reload hand
mirroring flips authored socket axes incorrectly
```

---

## Test 3: Right-Side Magazine

Setup:

```text
MagazineWell.AccessRegion = Right
HandPolicy = SameSide
RightShoulder and LeftShoulder variants
```

Expected:

```text
RightHand is preferred
stability solver may require regrip if right hand is main grip
resolved plan records regrip or alternate stabilization if required
```

Fail if:

```text
right hand releases only stabilizing contact without regrip
solver silently picks left hand without rejected reason/fallback record
```

---

## Test 4: Pistol No-Stock Reload

Setup:

```text
Weapon has no stock
RightHand = MainGrip
LeftHand = TwoHandPistolSupport or free
MagazineWell.AccessRegion = Bottom
Slide interaction point exists
```

Expected:

```text
RightHand remains stabilizing hand unless regrip policy allows otherwise
LeftHand performs magazine or slide manipulation
no shoulder contact is required
stability requirement is satisfied by pistol-specific rule
```

Fail if:

```text
planner assumes shoulder contact exists
trigger/main hand releases without valid stabilization transfer
```

---

## Test 5: Pump Shotgun Cycle

Setup:

```text
Pump moving part exists
PumpGrip follows moving part
RightShoulder or LeftShoulder
```

Expected:

```text
pump hand remains constrained to PumpGrip
PumpBack commit fires after backward motion
PumpForward commit fires after forward motion
hand follows moving part, not the other way around
```

Fail if:

```text
pump hand detaches and reaches as if pump were static
pump forward happens before pump back
mechanical state commits out of order
```

---

## Test 6: Shotgun Shell Loop Interrupt

Setup:

```text
Tube-fed shotgun
ShellInsert.AccessRegion = Bottom
LoadOne loop
interrupt after one committed shell
```

Expected:

```text
committed shell remains loaded
uncommitted shell is stowed/dropped/recovered according to policy
hand returns to valid hold
server owns interruption state
```

Fail if:

```text
client keeps visual shell but server did not commit it
reload clears without recovery while hand is away from grip
```

---

## Test 7: Remote Client Joins Mid Reload

Setup:

```text
server reload in progress
remote client becomes relevant at step index > 0
```

Expected:

```text
remote client reads ReloadInstance
computes alpha from server time
sets correct object visual state
seeks visual step instead of replaying from beginning
```

Fail if:

```text
remote client starts reload from step 0
remote client shows wrong magazine attachment state
```

---

## Test 8: Owner Prediction Rejected

Setup:

```text
owning client starts predicted reload
server rejects due to no ammo or changed mechanical state
```

Expected:

```text
predicted visual reload is cancelled
hands recover to valid hold pose
no ammo or magazine state changes on client persist
rejection reason is available for debug/UI
```

Fail if:

```text
predicted magazine remains visible in hand
client ammo diverges from server
weapon remains in reload pose indefinitely
```

---

## Test 9: Magazine Locked But Chamber Empty

Setup:

```text
MagazineLocked = true
RoundChambered = false
NeedsCycle = true
```

Expected:

```text
planner adds mechanism operation if weapon policy requires cycling
server blocks firing until cycle commit if policy requires chambered round
```

Fail if:

```text
system treats magazine ammo count as chambered round
fire permission ignores mechanical state
```

---

## Test 10: Invalid Socket Axis Validation

Setup:

```text
MagazineWell exists
LocalInsertAxis = zero vector or socket +X points wrong way in preview
```

Expected:

```text
validation fails for zero vector
preview clearly draws wrong axis direction
profile cannot pass production validation until fixed
```

Fail if:

```text
runtime silently falls back to point-to-point direction
axis error appears only as runtime animation bug
```

---

## Test 11: Visual Object Duplicate Prevention

Setup:

```text
owning client predicts magazine in hand
server confirms authoritative object state
```

Expected:

```text
predicted visual is reused, rebound, or destroyed
only one visible magazine remains
visual state matches ReloadInstance
```

Fail if:

```text
two magazines appear in hand/weapon
predicted object survives after rejection
```

---

## Test 12: Tactical Reload Stow Failure

Setup:

```text
TacticalReload requested
old magazine exists
no reachable stow slot
```

Expected based on policy:

```text
reject tactical reload
or convert to emergency reload if allowed
or drop old magazine if policy allows
```

Fail if:

```text
old magazine disappears without commit/drop/stow policy
planner silently changes intent without recording fallback
```

---

## Test 13: Editor Preview Right/Left Shoulder

Setup:

```text
same weapon profile
preview right shoulder and left shoulder
```

Expected:

```text
hand assignment changes according to shoulder side and access region
socket axes remain authored and are not blindly mirrored
rejected reasons are visible
```

Fail if:

```text
left shoulder uses right shoulder hand assignment
socket local axes are negative-scaled incorrectly
```

---

## Test 14: Moving Part Reference Validation

Setup:

```text
Bolt/Pump/Slide interaction point references missing moving part
```

Expected:

```text
profile or sequence validation fails
error names missing moving part
```

Fail if:

```text
runtime reaches null moving part and silently skips mechanism motion
```

---

## Test 15: LOD Does Not Affect Gameplay

Setup:

```text
remote client at low animation LOD
reload completes on server
```

Expected:

```text
mechanical state and commit points still replicate
visual detail may be simplified
fire permission remains correct
```

Fail if:

```text
reload gameplay depends on Control Rig/IK being evaluated
```

---

## Automation Recommendations

Recommended automated tests:

```text
profile validation automation
sequence validation automation
planner unit tests with mocked profiles/states
network phase reconstruction tests
prediction rejection tests
object lifecycle duplicate tests
```

Recommended manual/editor tests:

```text
socket axis preview
left/right shoulder preview
reload scrub preview
moving part preview
body slot reach preview
```

---

## Definition of Done

A weapon interaction implementation is not production-ready until:

```text
all MVP tests pass
invalid profile data fails validation
remote clients can join mid reload
prediction rejection recovers cleanly
object visuals do not duplicate
server mechanical state is authoritative
animation LOD does not affect gameplay
```

---

## Final Formula

```text
Weapon interaction acceptance =
  validated data
  + deterministic planning
  + stable contacts
  + explicit commit points
  + correct object lifecycle
  + server-authoritative networking
  + recoverable prediction
  + editor-visible debug.
```
