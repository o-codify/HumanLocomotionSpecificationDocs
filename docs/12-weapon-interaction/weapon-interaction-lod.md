---
id: weapon-interaction-lod
title: Weapon Interaction LOD
status: draft
version: 26.602.1533
tags: [ weapon, interaction, lod, performance, animation ]
---

# Weapon Interaction LOD

## Purpose

This document defines level-of-detail rules for weapon interaction.

Interaction LOD decides which procedural details can be simplified while preserving the readable meaning of the interaction.

---

## Scope Boundary

This document covers LOD for weapon interaction presentation:

```text
hand IK precision
finger/grip detail
object visual path detail
moving part visual detail
pose state reconstruction
debug/preview simplification
remote-client simplification
```

It does not own global game performance budgeting, renderer LOD, projectile simulation, damage, inventory, camera, or locomotion LOD.

---

## Core Rule

```text
LOD may simplify visual detail.
LOD must not change authoritative interaction state or commit timing.
```

A magazine lock commit, object visual state, pose state, or interaction phase must remain semantically correct even if visual detail is simplified.

---

## Suggested LOD Levels

### LOD0: Full Interaction

Use for:

```text
owner
nearby important characters
cinematic/high-detail view
editor preview
```

Features:

```text
full hand IK
elbow pole solving
finger/grip alpha
precise object path
moving part following
pose disturbance
correction smoothing
canonical/mirrored debug targets
```

### LOD1: Reduced Detail

Use for:

```text
near remote characters
normal third-person view
```

Features:

```text
hand IK active
simplified finger detail
object path still visible
moving part following simplified
less frequent debug drawing
```

### LOD2: Broad Interaction

Use for:

```text
mid-distance characters
crowded scenes
```

Features:

```text
broad pose state only
hand targets simplified
object path may be linear or snapped at phase boundary
finger solving disabled
moving part visual optional
```

### LOD3: State-Only Interaction

Use for:

```text
far remote characters
very low significance actors
```

Features:

```text
pose state only
phase state only
no precise hand IK
no precise object path
object visible only in major states
no moving part detail
```

---

## What Must Not Be Removed

Do not remove semantic state:

```text
current pose state
current interaction phase
object visual state
commit status
mirroring state
authority correction state
```

Even at low LOD, the viewer should understand:

```text
weapon is held
weapon is being reloaded/manipulated
object is in hand/in weapon/hidden
pose is low ready/hip/ADS broadly
```

---

## What Can Be Simplified

Can simplify:

```text
finger pose
wrist micro-correction
elbow pole quality
curved object path
sub-step hand arcs
moving part exact alpha
minor recoil/pose disturbance
small correction smoothing details
```

---

## Owner vs Remote

Owner usually needs:

```text
LOD0 or high LOD interaction feedback
responsive hand/object visuals
prediction smoothing
fire visual pose response
```

Remote clients can use:

```text
LOD1/LOD2/LOD3 depending on distance/significance
phase reconstruction from replicated state
simplified object paths
```

---

## Object LOD

Object visuals may degrade:

```text
LOD0: full object path and attachment blend
LOD1: object path visible, simplified blend
LOD2: object appears at key states only
LOD3: object may be hidden except major state changes
```

Never show duplicate authoritative-looking objects.

---

## Moving Part LOD

Moving parts may degrade:

```text
LOD0: moving part follows authored travel
LOD1: moving part follows simplified alpha
LOD2: moving part snaps at key phase
LOD3: moving part hidden or ignored visually
```

Commit state remains authoritative regardless of visual LOD.

---

## Mirroring And LOD

Mirroring must still be applied consistently.

Rules:

```text
LOD cannot skip the mirror stage if the actor is presented mirrored
LOD cannot mirror some targets and not others in a way that breaks interaction readability
canonical state remains canonical at all LODs
```

---

## Network And LOD

LOD changes do not require different replicated gameplay state.

Clients choose presentation detail locally from the same replicated interaction state.

Replicate:

```text
state
phase
time
revision
object visual state
mirror/presentation state if needed
```

Do not replicate per-LOD IK detail.

---

## LOD Switching

When switching LOD:

```text
preserve current interaction phase
preserve object visual state
blend hand targets if becoming more detailed
snap or fade minor details if becoming less detailed
avoid restarting the interaction animation
```

---

## Debug Requirements

Debug should show:

```text
current interaction LOD
enabled interaction features
disabled interaction features
reason for LOD selection
object LOD mode
moving part LOD mode
hand IK mode
```

---

## Final Formula

```text
Weapon interaction LOD =
  same authoritative interaction state
  + lower visual detail when needed
  + preserved semantic readability
  + no per-frame IK replication.
```
