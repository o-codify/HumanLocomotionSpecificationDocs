---
id: weapon-contact-quality
title: Weapon Contact Quality
status: draft
version: 26.602.1529
tags: [ weapon, interaction, contacts, stability, ik ]
---

# Weapon Contact Quality

## Purpose

This document defines how weapon interaction represents the quality of hand, shoulder, support, and object contacts.

A contact should not be only `active` or `inactive`. Procedural weapon interaction needs to know whether a contact is strong, partial, slipping, recovering, blocked, or lost.

---

## Scope Boundary

This document only describes contact quality for weapon interaction:

```text
hand to grip
hand to support grip
shoulder to stock
hand to magazine/object
hand to moving part
object to weapon socket
```

It does not own physical damage, locomotion balance, camera shake, or inventory state.

---

## Contact States

Recommended contact states:

```text
None
Approaching
PreGrip
Active
Partial
Slipping
Recovering
Blocked
Lost
ReleasedByPlan
```

### None

No contact exists.

### Approaching

The hand/object is moving toward the contact point.

### PreGrip

The hand is close enough to begin shaping fingers or aligning wrist, but the contact is not yet stable.

### Active

The contact is valid and stable enough for its role.

### Partial

The contact exists but is not fully stable.

Examples:

```text
support hand touches weapon but does not provide full support
shoulder contact is close but not seated
magazine is aligned but not locked
```

### Slipping

The contact is degrading over time.

### Recovering

The system is moving the hand/object back into a stable contact.

### Blocked

The target contact cannot be reached or maintained.

### Lost

The contact was expected but is no longer valid.

### ReleasedByPlan

The contact is intentionally released by the current action plan.

---

## Quality Values

Recommended continuous values:

```text
GripQuality: 0..1
SupportContactQuality: 0..1
ShoulderContactQuality: 0..1
ObjectAlignmentQuality: 0..1
MovingPartContactQuality: 0..1
ContactConfidence: 0..1
```

`ContactConfidence` represents how trustworthy the contact is for planning.

---

## Hold Stability Score

The hold system may compute:

```text
HoldStabilityScore: 0..1
```

Example inputs:

```text
main grip quality
support grip quality
shoulder contact quality
weapon archetype requirement
current pose state
temporary plan releases
object/manipulation load
```

Example interpretation:

```text
0.00 - 0.25: unsupported / invalid
0.25 - 0.50: weak hold
0.50 - 0.75: usable hold
0.75 - 1.00: stable hold
```

---

## Contact Requirements By Pose

### LowReady

```text
MainGrip: required
SupportGrip: optional/preferred for long guns
ShoulderContact: optional
```

### HipFire

```text
MainGrip: required
SupportGrip: preferred or required for long guns
ShoulderContact: optional/partial
```

### AimDownSights

```text
MainGrip: required
SupportGrip: preferred or required for long guns
ShoulderContact: preferred for stocked weapons
SightAlignment: high quality required for ADS-ready state
```

### Reloading

```text
Remaining contacts must keep weapon from becoming unsupported.
Released contacts must be marked ReleasedByPlan.
```

---

## Contact Transitions

Common transitions:

```text
None → Approaching → PreGrip → Active
Active → Partial → Slipping → Lost
Active → ReleasedByPlan → Recovering → Active
Blocked → Recovering → Active
Lost → Recovering → Active
```

Transitions should be time-smoothed to avoid snapping.

---

## Contact Quality And Planning

Planning should use contact quality to decide:

```text
can this hand be released?
can ADS be entered?
can reload start?
can fire visual readiness be reported?
can this moving part be operated?
should support hand recover first?
should action fall back to safer pose?
```

---

## Contact Quality And Animation

Animation may use contact quality for:

```text
IK alpha
finger grip alpha
wrist correction alpha
shoulder stock settle
weapon pose offset strength
object attachment blend
recovery blend speed
```

Example:

```text
SupportContactQuality = 0.3
→ support hand IK remains active but weak
→ weapon pose offset increases
→ solver may request support recovery
```

---

## Contact Quality And Mirroring

Contact quality is canonical state.

Mirrored presentation transforms contact targets but should not create duplicate contact states.

Correct:

```text
CanonicalSupportGripQuality = 0.8
PresentedTarget = Mirror(CanonicalTarget)
```

Incorrect:

```text
MirroredSupportGripQuality as separate gameplay state
```

---

## Failure Conditions

A contact may fail when:

```text
authored socket is missing
hand cannot reach target
object alignment is invalid
axis is wrong
mirror stage breaks target
network correction invalidates predicted target
required contact quality stays below threshold too long
```

---

## Debug Requirements

Debug should show:

```text
contact state
quality value
confidence value
required threshold
current pose requirement
released by plan or accidental loss
time below threshold
recovery target
```

---

## Final Formula

```text
Contact quality =
  discrete contact state
  + continuous quality score
  + confidence
  + pose-specific thresholds
  + recovery behavior.
```
