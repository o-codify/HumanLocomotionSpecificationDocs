---
id: weapon-contact-quality
title: Weapon Contact Quality
status: draft
version: 26.602.2053
tags: [ weapon, interaction, contacts, stability, ik ]
---

# Weapon Contact Quality

## Purpose

This document defines how weapon interaction represents the quality of hand, shoulder, support, sight, cheek, object, and moving-part contacts.

A contact should not be only `active` or `inactive`. Procedural weapon interaction needs to know whether a contact is strong, partial, slipping, recovering, blocked, or lost.

Important boundary:

```text
Contact quality is procedural confidence for animation, IK, planning, and visual interaction stability.
It is not an exact physics simulation of grip force, friction, recoil control, or human biomechanics.
```

---

## Scope Boundary

This document only describes contact quality for weapon interaction:

```text
hand to weapon grip
hand to support grip
support hand to main hand for two-handed handgun support
shoulder to stock
cheek/face relation to stocked weapon for ADS presentation
sight/eye alignment quality for ADS presentation
hand to magazine/object
hand to moving part
object to weapon socket
```

It does not own physical damage, locomotion balance, camera shake, projectile behavior, real firearm ballistics, or inventory state.

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

The contact is valid and stable enough for its procedural role.

### Partial

The contact exists but is not fully stable.

Examples:

```text
support hand touches weapon but does not provide full support
support hand touches main hand for pistol support but is not fully seated
shoulder contact is close but not seated
cheek/sight relation is close but not ADS-ready
magazine is aligned but not locked
```

### Slipping

The contact is degrading over time.

### Recovering

The system is moving the hand/object/weapon back into a stable contact.

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
HandToHandSupportQuality: 0..1
ShoulderContactQuality: 0..1
CheekContactQuality: 0..1
SightEyeAlignmentQuality: 0..1
ObjectAlignmentQuality: 0..1
MovingPartContactQuality: 0..1
ContactConfidence: 0..1
```

`ContactConfidence` represents how trustworthy the contact is for procedural planning and visual stability.

It should not be interpreted as exact physical grip strength.

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
hand-to-hand support quality for pistols
shoulder contact quality for stocked weapons
cheek/sight alignment quality for ADS presentation
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

## Contact Requirements By Weapon Archetype

### One-Handed Pistol

```text
MainGrip: required
SupportGrip: none or not applicable
HandToHandSupportContact: optional
ShoulderContact: none
CheekContact: none
SightEyeAlignment: optional for visual aim alignment only
```

### Two-Handed Handgun

```text
MainGrip: required
HandToHandSupportContact: preferred or required by pose
SupportHand may contact main hand and/or weapon frame
ShoulderContact: none
CheekContact: none
```

The support hand for a handgun is often not a simple support socket on the weapon. It may support the firing hand, wrap around the main hand, or create a combined two-hand grip.

### Stocked Rifle / Long Gun

```text
MainGrip: required
SupportGrip: preferred or required
ShoulderContact: preferred or required by pose
CheekContact: optional/preferred for ADS presentation
SightEyeAlignment: high quality required for ADS-ready presentation
```

### Pump / Foregrip Weapon

```text
MainGrip: required
SupportGrip or PumpGrip: required when operating pump/foregrip
MovingPartContactQuality: required during manipulation phase
ShoulderContact: weapon/pose dependent
```

---

## Contact Requirements By Pose

### LowReady

```text
MainGrip: required
SupportGrip: optional/preferred for long guns
HandToHandSupportContact: optional for handguns
ShoulderContact: optional
CheekContact: none
```

### HipFire / NonADSFire Pose

```text
MainGrip: required
SupportGrip: preferred or required for long guns
HandToHandSupportContact: preferred for two-handed handgun pose
ShoulderContact: optional/partial for stocked weapons
SightEyeAlignment: not strict
```

`HipFire` is a game-facing label for a non-sight-aligned fire-ready weapon pose. It does not necessarily mean the weapon is literally held at the hip.

### AimDownSights

```text
MainGrip: required
SupportGrip: preferred or required for long guns
HandToHandSupportContact: preferred for two-handed handgun ADS/presentation pose
ShoulderContact: preferred or required for stocked weapons
CheekContact: optional/preferred for stocked ADS presentation
SightEyeAlignment: high quality required for ADS-ready presentation
```

### Reloading

```text
Remaining contacts must keep weapon from becoming unsupported.
Released contacts must be marked ReleasedByPlan.
Required reload object contact must reach its authored quality threshold before commit.
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
should handgun support recover as hand-to-hand contact instead of weapon socket contact?
```

---

## Contact Quality And Animation

Animation may use contact quality for:

```text
IK alpha
finger grip alpha
wrist correction alpha
hand-to-hand support pose alpha
shoulder stock settle
cheek/sight presentation settle
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

Example for two-handed handgun:

```text
HandToHandSupportQuality = 0.4
→ support hand remains visually near main hand
→ two-hand pistol pose is not yet fully stable
→ fire visual readiness may stay degraded until support settles
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
hand-to-hand support target cannot be formed
cheek/sight alignment reference is missing for a stocked ADS pose that requires it
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
weapon archetype requirement
released by plan or accidental loss
time below threshold
recovery target
procedural confidence note when shown as non-physical value
```

---

## Final Formula

```text
Contact quality =
  procedural contact state
  + continuous confidence/quality score
  + weapon archetype requirements
  + pose-specific thresholds
  + recovery behavior.
```
