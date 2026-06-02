---
id: weapon-interaction-archetype-test-matrix
title: Weapon Interaction Archetype Test Matrix
status: draft
version: 26.602.1614
tags: [ weapon, interaction, tests, archetypes, validation ]
---

# Weapon Interaction Archetype Test Matrix

## Purpose

This document defines a test matrix for weapon interaction archetypes.

The goal is to verify that each archetype works across hold poses, contacts, object manipulation, moving parts, mirroring, LOD, networking, and correction smoothing.

---

## Scope Boundary

These tests validate weapon interaction only:

```text
contacts
pose state
hand/object targets
moving parts
object visual state
mirroring
LOD
network reconstruction
prediction correction
```

They do not validate damage, ballistics, ammo economy, inventory UI, camera, locomotion speed, AI behavior, or full cover system behavior.

---

## Archetypes To Test

```text
OneHandedPistol
TwoHandedHandgun
StockedRifle
BullpupRifle
PumpShotgun
BoltActionRifle
BreakActionWeapon
HeavyWeapon
LauncherOrSciFiHeavy
```

---

## Test Categories

Each archetype should be tested against:

```text
HoldPoseEntry
ContactQuality
NonADSFire
ADS or AimPose if supported
ReloadOrObjectInsertion
MovingPartOperation if supported
InterruptionRecovery
Mirroring
LOD
NetworkReconstruction
CorrectionSmoothing
AuthoringValidation
```

---

## Matrix

| Archetype | Hold | Contact | NonADS | ADS | Object Insert | Moving Part | Interrupt | Mirror | LOD | Network |
|---|---|---|---|---|---|---|---|---|---|---|
| OneHandedPistol | yes | yes | yes | optional | magazine | slide | yes | yes | yes | yes |
| TwoHandedHandgun | yes | hand-to-hand | yes | optional | magazine | slide | yes | yes | yes | yes |
| StockedRifle | yes | shoulder/cheek | yes | yes | magazine | charging/selector | yes | yes | yes | yes |
| BullpupRifle | yes | rear access | yes | yes | rear magazine | charging | yes | yes | yes | yes |
| PumpShotgun | yes | pump/support | yes | optional | shells | pump | yes | yes | yes | yes |
| BoltActionRifle | yes | shoulder/support | yes | yes | magazine/clip optional | bolt cycle | yes | yes | yes | yes |
| BreakActionWeapon | yes | hinge/open | yes | optional | shells | hinge/latch | yes | yes | yes | yes |
| HeavyWeapon | yes | external support | optional | optional | belt/box/cell optional | charging/latch | yes | yes | yes | yes |
| Launcher/SciFiHeavy | yes | support/core | optional | optional | core/cell | latch/cover | yes | yes | yes | yes |

---

## Required Tests Per Archetype

### Hold Pose Entry

Verify:

```text
valid main contact
required support contact can be established
hold stability score reaches threshold
pose lifecycle reaches Stable
```

### Contact Quality

Verify:

```text
required contacts report procedural quality
partial/lost/recovering states work
contact confidence affects planning
```

### NonADSFire

Verify:

```text
weapon reaches NonADSFire/HipFire pose
muzzle follows external aim intent plausibly
interaction fire readiness is produced only from interaction state + external backend answer
```

### ADS / Aim Pose

Verify when supported:

```text
ADS pose reaches required contact thresholds
stocked ADS checks shoulder/cheek/sight quality if required
handgun ADS uses hand-to-hand support if authored
ADS does not own camera or body orientation
```

### Reload / Object Insertion

Verify:

```text
object visual state transitions correctly
insert tip aligns to target socket
insert axis/path is semantic
commit fires once
post-lock settle happens
```

### Moving Part Operation

Verify when supported:

```text
hand grips moving part
moving part follows authored path
commit happens at authored point
hand follow target clears or recovers
moving part settles to authority state
```

### Interruption / Recovery

Verify:

```text
before-commit recovery
at-commit handling
post-commit recovery
object visual state stays consistent
hands recover to valid pose
```

### Mirroring

Verify:

```text
canonical state remains unchanged
presentation mirrors once
hand targets, object paths, axes, and elbow poles mirror consistently
no separate left-handed gameplay plan is created
```

### LOD

Verify:

```text
LOD reduces visual detail only
semantic state remains correct
object/moving part state remains readable
no per-frame IK replication is needed
```

### Network Reconstruction

Verify:

```text
remote reconstructs from phase/time/state
late relevancy seeks to correct phase
owner prediction reconciles with authority
correction smoothing avoids unsafe snap when possible
```

### Authoring Validation

Verify:

```text
required sockets exist
required axes are valid
archetype-required contacts exist
mirror preview passes
missing optional data warns only when required by archetype/pose
```

---

## Acceptance Rule

An archetype is accepted when:

```text
all required contacts validate
all required object/moving part operations validate
mirroring is safe
LOD preserves semantic state
network reconstruction works without per-frame IK
failure/recovery paths are visible in debug
```

---

## Final Formula

```text
Archetype test matrix =
  each interaction archetype
  × hold/contact/aim/reload/moving-part/mirror/LOD/network tests
  + authoring validation
  + failure/recovery checks.
```
