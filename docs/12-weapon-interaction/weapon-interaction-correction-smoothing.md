---
id: weapon-interaction-correction-smoothing
title: Weapon Interaction Correction Smoothing
status: draft
version: 26.602.1532
tags: [ weapon, interaction, networking, prediction, smoothing ]
---

# Weapon Interaction Correction Smoothing

## Purpose

This document defines how weapon interaction visually smooths corrections from authority, prediction rejection, late relevancy, and replicated state changes.

Correction smoothing is presentation-side interaction recovery. It must not change authoritative gameplay state.

---

## Scope Boundary

This document smooths weapon interaction visuals:

```text
hand targets
weapon pose offsets
object visual transforms
moving part visual transforms
pose lifecycle transitions
fire visual pose impulses
mirrored presentation targets
```

It does not smooth or own projectile state, damage, ammo, inventory, camera, locomotion speed, or body orientation authority.

---

## Correction Sources

Common correction sources:

```text
server step correction
prediction rejected
object visual state corrected
pose state corrected
mechanical interaction state corrected
mirror state changed
remote late relevancy
LOD changed
```

---

## Smoothing Rule

```text
Authoritative state changes immediately.
Visual interaction state blends toward the authoritative presentation when safe.
```

Do not delay gameplay/authoritative commit just to make visuals smooth.

---

## Correction Data

Suggested conceptual state:

```text
FWeaponInteractionCorrectionState:
  bCorrectionActive
  CorrectionReason
  FromCanonicalState
  ToCanonicalState
  FromPresentedTransform
  ToPresentedTransform
  StartTime
  Duration
  Alpha
  bSnapRequired
  DebugReason
```

---

## Snap vs Blend

### Snap Required

Snap when:

```text
a visual object is duplicated
weapon actor/component changed
required socket is invalid
current target is NaN/invalid
predicted object should not exist
state is hidden by transition/camera cut/external reset
```

### Blend Preferred

Blend when:

```text
hand target corrected slightly
step alpha changed
pose state corrected between compatible poses
object transform corrected within same visual state
moving part alpha corrected
remote joined mid-step
mirror state changed but canonical state is same
```

---

## Hand Target Correction

When hand targets are corrected:

```text
store current presented hand transform
compute target presented hand transform from authority state
blend position and rotation separately
preserve elbow pole continuity if possible
fade finger grip alpha to corrected value
```

Avoid blending through the weapon mesh if a recovery path exists.

---

## Object Visual Correction

When object visual state is corrected:

```text
if duplicate predicted object exists, remove/hide predicted duplicate
if object remains same state, blend transform
if object changes parent, use attach blend or snap depending on visibility
if object becomes hidden, fade or hide immediately depending on correction reason
```

Never keep two authoritative-looking copies of the same object.

---

## Moving Part Correction

When moving part state is corrected:

```text
blend moving part alpha to authority value
if commit boundary changed, settle to committed state
if moving part reference invalid, clear hand follow target and report failure
```

---

## Pose State Correction

Compatible pose correction can blend:

```text
HipFire ↔ ADS
LowReady ↔ HipFire
Reloading overlay ↔ previous stable pose
```

Incompatible correction may snap or use recovery:

```text
weapon unavailable
socket missing
interaction failed
object duplicate
```

---

## Mirroring Correction

If mirror/presentation side changes:

```text
canonical state remains unchanged
presented targets are recomputed
blend from old presented transforms to new presented transforms
clear any double-mirror stage warning
```

Do not rebuild mirrored gameplay state.

---

## Late Relevancy

When a remote client becomes relevant mid-interaction:

```text
receive replicated canonical state
compute current step alpha from server time
build presented target from current phase
start with either direct seek or short blend from current pose
```

If the character was not visible before, direct seek is acceptable.
If visible and state changed, short blend is preferred.

---

## Smoothing Durations

Suggested values:

```text
minor hand correction: 0.05 - 0.12 s
pose correction: 0.10 - 0.25 s
object transform correction: 0.05 - 0.15 s
moving part correction: 0.03 - 0.10 s
prediction rejection: 0.10 - 0.25 s if safe
```

Values are presentation tuning, not gameplay authority.

---

## Debug Requirements

Debug should show:

```text
correction active
correction reason
snap or blend
from state
to state
blend alpha
duration
predicted id
authority revision
object duplicate cleanup
```

---

## Final Formula

```text
Correction smoothing =
  authority state wins immediately
  + visual state blends when safe
  + snap when invalid/duplicated
  + no duplicate objects
  + no gameplay delay for visual smoothness.
```
