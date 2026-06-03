---
id: weapon-procedural-animation-ownership-policy
title: Weapon Procedural Animation Ownership Policy
status: draft
version: 26.603.1613
tags: [ weapon, interaction, animation, procedural, control-rig ]
---

# Weapon Procedural Animation Ownership Policy

## Purpose

This document defines the animation ownership policy for weapon interaction.

Weapon interaction in this section is **procedural-first**.

Authored animation clips, montages, or pose assets are not the primary source of weapon interaction motion. They may be used only as optional references, style layers, hand-shape assets, or fallback presentation, while procedural targets, constraints, phases, contacts, and numeric pose validity remain the source of truth.

---

## Scope Boundary

This document owns procedural animation policy for weapon interaction:

```text
procedural hand targets
procedural weapon pose offsets
procedural object paths
procedural moving part following
Control Rig / IK solving
contact quality enforcement
numeric visual pose acceptance
correction smoothing
LOD simplification
optional authored reference usage
```

It does not own:

```text
locomotion animation as a whole
camera animation
inventory UI animation
damage reaction animation
projectile visuals
full cinematic animation direction
```

---

## Core Rule

```text
Procedural interaction is the source of truth.
Authored animation is optional presentation support.
```

The implementation must not require a full authored reload animation to make weapon interaction work.

The required state is generated from:

```text
weapon interaction profile
weapon/object sockets
semantic local axes
interaction phase
step alpha
contact constraints
joint/reach constraints
numeric visual pose contract
network authority state
```

---

## Procedural System Owns

Procedural weapon interaction owns:

```text
hand target generation
elbow pole generation
wrist/contact target orientation
weapon pose offset request
object insert/extract path
moving part follow path
contact quality state
GripPoseId selection
interaction phase and step alpha
commit-safe visual state
mirror-safe presentation target generation
network correction target rebuild
numeric visual pose validity
```

These are not authored montage responsibilities.

---

## Authored Assets May Provide

Authored assets may provide optional support:

```text
reference pose for broad style
pose asset for grip/finger shape
optional additive flavor
optional cinematic-only presentation
optional fallback when procedural detail is disabled by LOD
```

Authored assets must not be required for correctness of:

```text
hand contact placement
magazine insertion alignment
moving part operation path
reload commit point
object ownership state
network reconstruction
```

---

## Not Allowed

The following implementation model is not allowed for HLS weapon interaction:

```text
play a full reload montage
then use IK only to patch hands roughly onto the weapon
```

Also not allowed:

```text
authored montage determines magazine commit timing without interaction state
authored hand motion owns object position during insertion
hand placement accepted because montage looks close
Control Rig used only as final cosmetic correction while authored clip drives the real action
```

---

## Procedural Layer Order

Recommended order:

```text
1. base locomotion pose from external locomotion system
2. weapon interaction pose state selects procedural target set
3. procedural weapon pose offset is computed
4. procedural hand targets and elbow poles are generated
5. procedural object/moving-part targets are generated
6. optional authored reference/additive style is applied only where allowed
7. Control Rig / IK solves contacts against procedural targets
8. grip/finger pose assets apply from GripPoseId if available
9. numeric visual pose contract validates result
10. correction smoothing applies when authority or prediction changes state
```

Authored reference layers may be disabled and the interaction must still remain functionally valid.

---

## IK Alpha Policy

IK/contact solving is not a cosmetic afterthought. It is part of the procedural interaction.

Suggested alpha behavior:

```text
Approach: procedural path alpha active, contact alpha increasing
PreGrip: contact orientation alpha increasing
Grip: contact alpha high
Align/Insert/Operate: contact/object/moving part alpha high
Commit: procedural alignment alpha high
Settle: maintain required contact, smooth non-critical offsets
Recover: blend to recovery target under constraint priority
Release: reduce contact alpha only when ReleasedByPlan
```

Do not set IK alpha to zero during required contacts.

Do not make authored montage alpha the authority for interaction correctness.

---

## Object Path Policy

Default object path policy is procedural.

Object movement should be generated from:

```text
object local sockets
weapon target sockets
semantic insert/extract axes
interaction phase
step alpha
object insertion quality thresholds
```

Allowed optional authored input:

```text
style curve
non-critical arc bias
timing curve that does not override commit validation
```

Not allowed:

```text
magazine position comes only from montage hand animation
insert commit happens because montage reached frame N without socket/axis validation
object teleports to weapon because authored animation ended
```

---

## Moving Part Policy

Moving part visuals follow procedural interaction state.

Source of truth:

```text
MovingPartOperationProfile
MovingPartAlpha
CommitState
AuthorityCorrection
```

Authored animation may add style, but it must not own the final mechanical interaction state.

---

## Grip And Finger Poses

Grip and finger assets may be authored.

But the selection and blending are procedural:

```text
GripPoseId selected by interaction profile/state
FingerGripAlpha driven by contact phase/quality
hand contact frame validated by numeric pose contract
```

A good finger pose cannot compensate for an invalid wrist, elbow, or contact frame.

---

## Procedural Validation

Every generated pose must be validated against:

```text
Weapon Visual Pose Numeric Contract
ContactQuality thresholds
ConstraintPriority rules
ObjectInsertionQuality thresholds
MovingPartOperationProfile commit rules
```

If validation fails, the system must choose:

```text
recover
adjust pose
request support assist
fallback pose
fail interaction
```

It must not continue only because an authored animation is playing.

---

## Avoiding Fighting

Common fighting cases:

```text
authored clip moves hand while procedural target moves it elsewhere
object follows a montage while interaction state also sets object transform
Control Rig mirrors a target already mirrored by presentation layer
recoil additive moves weapon but required contacts are not re-solved
correction smoothing blends stale targets while procedural planner writes new targets
```

Fix by defining:

```text
single procedural source of truth per phase
one mirror stage
one object attachment authority
one final contact solve stage
authored layers restricted to non-authoritative style/reference
```

---

## Low LOD Policy

At lower LOD, procedural detail may be simplified.

Allowed:

```text
reduce finger detail
reduce hand path curvature
use broad procedural pose instead of exact target
hide minor moving part details
```

Not allowed:

```text
replace semantic reload/object state with unrelated montage-only playback
change commit timing
drop object ownership state
replicate per-frame authored pose instead of semantic interaction state
```

---

## Debug Requirements

Debug should show:

```text
ProceduralSourceOfTruth: true
CurrentInteractionPhase
StepAlpha
HandTargetSource
ObjectPathSource
MovingPartSource
GripPoseId
ContactQuality
VisualPoseScore
AuthoredReferenceAlpha if used
AuthoredReferenceIsAuthority: false
CurrentMirrorStage
FightingWarning
```

---

## Final Formula

```text
Procedural weapon animation policy =
  procedural targets are authority
  + authored assets are optional support
  + contact/object/moving-part correctness is numeric
  + commit timing comes from interaction state
  + no montage-only reload correctness.
```
