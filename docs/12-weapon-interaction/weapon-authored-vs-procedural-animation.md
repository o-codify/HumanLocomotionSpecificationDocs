---
id: weapon-authored-vs-procedural-animation
title: Weapon Authored vs Procedural Animation
status: draft
version: 26.602.2053
tags: [ weapon, interaction, animation, procedural, control-rig ]
---

# Weapon Authored vs Procedural Animation

## Purpose

This document defines how authored animation and procedural weapon interaction work together.

Weapon interaction should not become a fight between authored animation, IK, Control Rig, correction smoothing, and replicated state.

---

## Scope Boundary

This document owns animation blending policy for weapon interaction:

```text
authored base pose
procedural hand targets
IK alpha
Control Rig correction
object path following
moving part following
network correction smoothing
LOD simplification
```

It does not own locomotion animation as a whole, camera animation, damage reaction animation, inventory UI animation, or projectile visuals.

---

## Core Rule

```text
Authored animation gives style and broad pose.
Procedural interaction preserves contacts, object alignment, and state correctness.
```

Neither should blindly overwrite the other.

---

## Authored Animation Owns

Authored animation should own:

```text
broad body style
weapon carry pose style
reload acting style
anticipation and follow-through
pose appeal
hand shape assets
finger pose assets
non-critical motion arcs
```

---

## Procedural System Owns

Procedural weapon interaction should own:

```text
final hand contact alignment
object insert/extract alignment
moving part follow target
contact quality recovery
network correction targets
mirror-safe target presentation
commit-safe visual state
```

---

## Blending Layers

Recommended order:

```text
1. base locomotion pose
2. authored weapon carry/aim pose
3. authored reload/manipulation montage or pose layer if used
4. procedural weapon pose offset
5. procedural hand/object target correction
6. Control Rig contact solve
7. finger/grip pose blend
8. final correction smoothing if needed
```

Project pipelines may differ, but the ownership order must be explicit.

---

## When Authored Animation Wins

Authored animation may win when:

```text
contact is not critical yet
hand is in Approach phase
motion arc is cosmetic
LOD disables fine IK
character is far away
editor preview is evaluating broad style
```

---

## When Procedural Wins

Procedural correction must win when:

```text
object must align to insert socket
moving part hand must follow authored travel
required grip contact must be stable
weapon would become unsupported
authority corrected state
commit point visual must match state
mirror stage requires target consistency
```

---

## IK Alpha Policy

Suggested alpha behavior:

```text
Approach: low to medium IK alpha
PreGrip: increasing IK alpha
Grip: high IK alpha
Align/Insert/Operate: high IK alpha
Commit: high IK alpha
Settle: decreasing or stable IK alpha
Recover: controlled blend to recovery target
Release: decreasing IK alpha
```

Do not set IK alpha to 1 for the entire montage if it destroys authored motion unnecessarily.

Do not set IK alpha to 0 during critical contacts.

---

## Avoiding Fighting

Common fighting cases:

```text
montage animates hand to one place while IK target pulls elsewhere
Control Rig mirrors a target already mirrored by AnimGraph
object path follows authored animation while runtime also sets object transform
recoil additive moves weapon but final hand contact does not correct it
correction smoothing blends old target while solver writes new target every frame
```

Fix by defining:

```text
single source of truth per phase
one mirror stage
one object attachment authority
one final hand correction stage
```

---

## Object Path Policy

Object path may be:

```text
authored animation driven
procedural spline/path driven
socket-axis driven
hybrid authored arc + procedural final alignment
```

Recommended default:

```text
authored/preferred arc for style
procedural final alignment for insertion/commit
```

---

## Moving Part Policy

Moving part visuals should follow runtime interaction state.

Authored animation may provide style, but the moving part final state must match:

```text
MovingPartAlpha
CommitState
AuthorityCorrection
```

---

## Network Correction Policy

When authority corrects interaction state:

```text
authority state wins immediately
visual correction smoothing blends when safe
procedural targets rebuild from authority state
authored montage section may be skipped/searched if needed
```

---

## Debug Requirements

Debug should show:

```text
authored pose alpha
procedural IK alpha
Control Rig correction alpha
object path source
authority correction active
mirror stage
current winning source
fighting warning
```

---

## Final Formula

```text
Authored vs procedural policy =
  authored style
  + procedural contact correctness
  + explicit per-phase ownership
  + no double mirroring
  + no competing object authorities.
```
