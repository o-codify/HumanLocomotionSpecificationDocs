---
id: weapon-first-and-third-person-interaction-presentation
title: Weapon First and Third Person Interaction Presentation
status: draft
version: 26.602.1614
tags: [ weapon, interaction, presentation, first-person, third-person, animation ]
---

# Weapon First and Third Person Interaction Presentation

## Purpose

This document defines how one canonical weapon interaction state can be presented through different first-person and third-person animation rigs.

This is presentation of interaction. It is not a camera system, FOV system, aiming system, or shooter viewmodel framework.

---

## Scope Boundary

Weapon interaction owns:

```text
shared canonical interaction state
owner high-fidelity interaction targets
third-person full-body interaction targets
remote simplified interaction reconstruction
object visual state consistency
moving part visual state consistency
```

External systems own:

```text
camera placement
FOV
camera collision
viewmodel rendering pipeline
visibility/culling policy
input mapping
```

---

## Core Rule

```text
One canonical interaction state may drive multiple presentation rigs.
```

Do not create separate gameplay interaction states for first person and third person.

---

## Presentation Modes

Recommended modes:

```text
OwnerFirstPersonHighFidelity
OwnerThirdPersonBody
RemoteThirdPerson
CinematicFullBody
LowLODRemote
```

---

## Shared Canonical State

Shared state includes:

```text
pose state
interaction phase
step alpha
contact quality
object visual state
moving part state
presentation mirror state
revision/correction state
```

Each presentation mode interprets this state at different fidelity.

---

## Owner First Person

Owner first-person presentation may use:

```text
higher hand/object fidelity
more precise fingers
more precise object insertion alignment
stronger correction smoothing
viewmodel-specific authored animation
```

But it must preserve:

```text
same phase
same commit timing
same object visual state
same canonical interaction meaning
```

---

## Owner Third Person Body

Owner third-person body presentation should remain coherent for shadows, mirrors, replay, or local full-body visibility.

Rules:

```text
use same canonical state
may simplify very small first-person details
must keep broad pose and object state correct
must not disagree with first-person commit/object state
```

---

## Remote Third Person

Remote clients use replicated state:

```text
pose state
phase/time
object visual state
moving part state
mirror state
revision
```

Remote clients should not need owner local-only IK variables.

---

## Rig Differences

Different rigs may have different target mappings:

```text
first-person arms rig
full-body third-person rig
cinematic rig
low-LOD rig
```

Each rig should implement a mapping from canonical interaction targets to rig-specific controls.

Do not duplicate interaction logic per rig.

---

## Object Visual Consistency

Object visual state must be consistent across presentation modes.

Example:

```text
if canonical state says magazine is InWeapon, first-person and third-person should not show it in hand
if owner predicted object in hand and server rejects, both presentations must correct
```

---

## Moving Part Consistency

Moving part state should remain semantically consistent:

```text
slide alpha
pump alpha
bolt state
hinge angle
latch state
```

Fidelity may differ, but state meaning should not.

---

## LOD And Presentation

First-person usually uses high fidelity.

Remote third-person may use lower interaction LOD:

```text
LOD0 owner/near cinematic
LOD1 near remote
LOD2 mid remote
LOD3 state-only remote
```

LOD changes presentation detail, not canonical state.

---

## Mirroring

Presentation mirroring must be applied consistently for each rig.

Rules:

```text
canonical interaction state is not duplicated
first-person and third-person must agree on mirror state if both are visible
mirror in one stage per rig
```

---

## Debug Requirements

Debug should show:

```text
canonical interaction state
active presentation mode
rig mapping profile
object visual state per presentation
moving part state per presentation
presentation LOD
mirror state
prediction/correction state
```

---

## Final Formula

```text
First/third-person interaction presentation =
  one canonical interaction state
  + multiple rig-specific presentation mappings
  + consistent object/moving-part state
  + no separate gameplay logic per view.
```
