---
id: weapon-interaction-interruptions
title: Weapon Interaction Interruptions
status: draft
version: 26.602.2053
tags: [ weapon, interaction, interruptions, recovery, state ]
---

# Weapon Interaction Interruptions

## Purpose

This document defines how weapon interactions are interrupted and recovered.

Interruptions are not reload-specific. Any interaction can be interrupted:

```text
hold pose entry
reload
mechanism manipulation
draw
holster
object insertion
object extraction
support hand recovery
```

---

## Scope Boundary

This document owns interruption behavior for weapon interaction state only:

```text
hand targets
contacts
object visual state
moving part visual state
pose lifecycle
interaction phases
recovery targets
```

It does not own locomotion interruptions, damage reactions, inventory cancellation rules, camera changes, or gameplay backend authority.

---

## Interruption Types

Recommended types:

```text
SoftInterrupt
HardInterrupt
AuthorityCorrection
PredictionRejected
PoseInvalidated
ObjectInvalidated
WeaponUnavailable
MirrorStateChanged
```

### SoftInterrupt

The interaction can blend out safely.

Examples:

```text
player stops ADS
reload canceled before object detached
support hand recovery canceled into LowReady
```

### HardInterrupt

The interaction must stop immediately and recover.

Examples:

```text
weapon removed
required socket missing at runtime
action state becomes impossible
```

### AuthorityCorrection

Server corrected the interaction phase or state.

### PredictionRejected

Predicted owner interaction was rejected by authority.

### PoseInvalidated

Current pose can no longer be restored.

### ObjectInvalidated

Visual object state no longer matches authoritative interaction state.

### WeaponUnavailable

Weapon actor/mesh/component is no longer valid.

### MirrorStateChanged

Global presentation mirror changed mid-interaction.

---

## Commit Boundary

Every interrupt must consider commit status.

```text
BeforeCommit:
  safe to cancel and restore previous state.

AtCommit:
  commit must be applied exactly once or rejected before application.

AfterCommit:
  recovery must preserve committed interaction state.
```

Example:

```text
MagazineDetached commit already happened.
Interrupt cannot pretend magazine is still in weapon unless authority says so.
```

---

## Recovery Targets

Recovery should choose one target:

```text
PreviousStableHoldPose
ConfiguredExitPose
FallbackPose
CurrentAuthorityPose
ClearInteractionState
FailedState
```

Recommended fallback order:

```text
1. authority-corrected pose/state
2. previous stable hold pose
3. configured exit pose
4. LowReady or RelaxedCarry
5. clear interaction state
6. fail visibly and report debug reason
```

---

## Object Recovery

Object visual state must recover with the interaction state.

Examples:

```text
object was only predicted → destroy or hide predicted visual
object was detached by authoritative commit → keep detached visual state
object was inserted by authoritative commit → attach to weapon visual socket
object is in hand during interruption → either keep in hand, hide, or snap/blend to authority state
```

Do not create duplicate object visuals during recovery.

---

## Hand Target Recovery

When interrupted:

```text
freeze current target briefly if needed
choose recovery target
blend hand target toward recovery target
clear manipulation hand role after safe phase
restore support contact if required
clear object grip target when object state no longer needs it
```

Avoid instant snapping unless the current state is invalid or hidden by transition.

---

## Moving Part Recovery

Moving parts such as bolt, slide, pump, lever, or charging handle must recover according to committed state.

```text
if commit not reached:
  return moving part to pre-step position if authored

if commit reached:
  settle moving part to committed visual position

if authority corrected:
  blend to authority moving part state
```

---

## Prediction Rejection

Owner prediction rejection should:

```text
stop predicted step
clear predicted-only visual objects
restore authoritative object visual state
blend hands to authoritative targets
restore authoritative pose lifecycle
increment or consume correction revision
```

Do not leave stale IK targets or duplicate objects.

---

## Mirror Change During Interaction

If global presentation mirror changes during an interaction:

```text
canonical interaction state remains unchanged
presentation targets are recomputed with the new mirror state
in-flight visual paths blend to new presentation output
object visual paths update from canonical state
```

Do not rebuild a separate mirrored interaction plan.

---

## Interruption Result

Suggested conceptual result:

```text
FWeaponInteractionInterruptResult:
  bHandled
  InterruptType
  CommitStatus
  RecoveryTargetPose
  ObjectRecoveryAction
  HandRecoveryAction
  MovingPartRecoveryAction
  bRequiresAuthorityCorrection
  DebugReason
```

---

## Debug Requirements

Debug should show:

```text
interrupt type
interrupt source
commit status
current step
current phase
previous stable pose
recovery target
object recovery action
hand recovery action
moving part recovery action
prediction id
authority revision
```

---

## Final Formula

```text
Interaction interruption =
  identify interrupt type
  + respect commit boundary
  + recover hands/object/moving parts
  + preserve authority state
  + clear stale visual targets
  + blend whenever possible.
```
