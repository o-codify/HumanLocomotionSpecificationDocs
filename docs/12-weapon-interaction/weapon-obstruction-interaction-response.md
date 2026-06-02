---
id: weapon-obstruction-interaction-response
title: Weapon Obstruction Interaction Response
status: draft
version: 26.602.2053
tags: [ weapon, interaction, obstruction, response, pose ]
---

# Weapon Obstruction Interaction Response

## Purpose

This document defines how weapon interaction responds to obstruction results.

This is not a collision detection system. Weapon interaction consumes obstruction results from external query/collision systems and chooses an interaction response.

---

## Scope Boundary

Weapon interaction owns:

```text
pose fallback response
ADS/NonADSFire blocking response
hand path adjustment response
object insertion delay/recovery
compact hold pose request
visual obstruction state
```

External systems own:

```text
collision traces
physics queries
wall detection
cover detection
body placement
camera collision
projectile obstruction
```

---

## Obstruction Inputs

Conceptual input:

```text
WeaponObstructionResult:
  bWeaponBodyBlocked
  bMuzzleBlocked
  bHandPathBlocked
  bObjectPathBlocked
  bInsertionPathBlocked
  bADSBlocked
  ObstructionNormal
  ObstructionPoint
  Severity
  SourceSystem
```

---

## Response Types

Recommended responses:

```text
None
DegradePose
BlockPoseEntry
ExitPose
CompactHold
DelayStep
ChooseAlternatePath
RecoverObject
FailInteraction
```

---

## Muzzle / Weapon Body Blocked

If external system reports muzzle or weapon body obstruction:

```text
ADS may be blocked or degraded
NonADSFire pose may lower/compact
weapon pose offset may pull weapon closer
fire visual readiness may report not-ready from interaction state
```

Weapon interaction does not decide projectile collision or damage.

---

## Hand Path Blocked

If a hand path is blocked:

```text
try alternate approach arc if authored
try compact pose adjustment
try delay phase
try recover to previous stable pose
fail if hard target cannot be reached
```

---

## Object Path / Insertion Path Blocked

If an object path is blocked:

```text
pause before commit
back out along extract axis if safe
return to PreAlign or Recover
request fallback pose
fail before commit if no path exists
```

Do not commit insertion if insertion readiness thresholds are not met.

---

## ADS Blocked

If external obstruction says ADS is blocked:

```text
block ADS entry
or degrade ADS to PointAim/NonADSFire/LowReady
or request compact hold pose
```

Weapon interaction does not solve body yaw, feet, peeking, or cover exposure.

---

## Obstruction Severity

Suggested severity:

```text
None
Minor
Moderate
Severe
Invalid
```

Example behavior:

```text
Minor: increase pose offset or reduce alignment quality
Moderate: degrade pose or delay step
Severe: block pose/step and recover
Invalid: fail interaction or clear unsafe targets
```

---

## Networking

Replicate obstruction response only if needed for remote reconstruction.

Usually remote clients can reconstruct visual response from replicated pose/phase state.

Potential replicated fields:

```text
ObstructionResponseState
PoseFallbackReason
ResponseStartServerTime
ResponseRevision
```

Do not replicate per-frame obstruction query results.

---

## Debug Requirements

Debug should show:

```text
obstruction input source
blocked type
severity
selected response
pose fallback
affected hand/object path
commit blocked or allowed
recovery target
```

---

## Final Formula

```text
Obstruction interaction response =
  consume external obstruction result
  + choose pose/path/object response
  + preserve commit safety
  + do not own collision detection.
```
