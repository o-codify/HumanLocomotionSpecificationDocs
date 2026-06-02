---
id: weapon-interaction-phase-taxonomy
title: Weapon Interaction Phase Taxonomy
status: draft
version: 26.602.2053
tags: [ weapon, interaction, phases, sequencing, animation ]
---

# Weapon Interaction Phase Taxonomy

## Purpose

This document defines a shared vocabulary for weapon interaction phases.

Different actions should not invent unrelated names for the same procedural meaning. Reloads, draw/holster, object insertion, moving part operation, and support hand recovery should use a common phase vocabulary where possible.

---

## Scope Boundary

This document owns phase names for weapon interaction only:

```text
hand approach
pre-grip
grip
release
object movement
alignment
insert/extract
operate
commit
settle
recover
```

It does not define inventory states, damage states, projectile states, camera states, or locomotion states.

---

## Core Rule

```text
Phase names describe visible/procedural interaction progress.
Commit points describe semantic state changes.
```

Do not confuse phase with commit.

Example:

```text
Insert phase may be visually in progress.
MagazineLocked commit happens once at the authored point inside or after Insert.
```

---

## Common Phases

Recommended phase vocabulary:

```text
Idle
Approach
PreGrip
Grip
Detach
Move
Align
Insert
Extract
Operate
Hold
Commit
Settle
Recover
Release
Clear
Failed
```

---

## Phase Meanings

### Idle

No active interaction step.

### Approach

Hand/object moves toward a target.

### PreGrip

Hand prepares the shape before stable contact.

### Grip

Hand establishes contact with weapon/object/moving part.

### Detach

Object or weapon visual detaches from current parent.

Examples:

```text
magazine detaches from weapon
weapon detaches from body socket during draw
object detaches from body slot visual
```

### Move

Object/hand/weapon moves through a path that is not yet final alignment.

### Align

Object/hand/moving part aligns to a target socket, axis, or contact.

### Insert

Object moves into the target interaction point.

### Extract

Object moves out of the target interaction point.

### Operate

A moving part or control is actively manipulated.

Examples:

```text
pump cycle
bolt pull
slide rack
lever rotate
button press
latch pull
```

### Hold

The interaction intentionally pauses or maintains contact.

### Commit

The interaction semantic state changes.

Examples:

```text
MagazineDetached
MagazineLocked
ShellInserted
BoltOpen
BoltClosed
WeaponDetachedFromBody
WeaponAttachedToBody
```

### Settle

Visual contacts and pose stabilize after commit/movement.

### Recover

The system returns hand/contact/pose to a stable state.

### Release

Hand/object contact releases by plan.

### Clear

Interaction state clears stale targets/objects.

### Failed

The interaction cannot continue or recover.

---

## Phase vs Step

A step is an authored unit of action.

A phase is the current procedural meaning inside the step.

Example:

```text
Step: InsertMagazine
Phases:
  Align
  Insert
  Commit
  Settle
```

---

## Phase vs Contact State

Contact state describes the quality of a contact.

Phase describes action progress.

Example:

```text
Phase = Grip
ContactState = PreGrip → Active
```

---

## Phase vs Lifecycle

Hold pose lifecycle describes the pose-level state.

Phase describes the active interaction action.

Example:

```text
PoseLifecycle = TemporarilyDisrupted
InteractionPhase = Extract
```

---

## Recommended Step Structure

Many interaction steps can use:

```text
Approach → PreGrip → Grip → Move/Align → Operate/Insert/Extract → Commit → Settle → Release/Recover
```

Not every step needs every phase.

---

## Debug Requirements

Debug should show:

```text
CurrentStepId
CurrentPhase
PreviousPhase
PhaseStartTime
PhaseDuration
PhaseAlpha
CommitPointInsidePhase
PhaseFailureReason
```

---

## Final Formula

```text
Interaction phase taxonomy =
  shared procedural phase names
  + clear distinction from commit points
  + reusable sequencing vocabulary
  + less ambiguity across reload/draw/holster/moving-part actions.
```
