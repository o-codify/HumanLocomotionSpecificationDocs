---
id: weapon-interaction-authoring-guidelines
title: Weapon Interaction Authoring Guidelines
status: draft
version: 26.602.1531
tags: [ weapon, authoring, sockets, axes, editor, validation ]
---

# Weapon Interaction Authoring Guidelines

## Purpose

This document gives practical authoring rules for weapon interaction assets.

It is written for technical animators, gameplay animators, weapon artists, and engineers who create weapon profiles, sockets, axes, reload object profiles, and preview data.

---

## Scope Boundary

This document describes authoring of weapon interaction data:

```text
sockets
interaction points
local axes
hand grip points
support grip points
stock/shoulder contact
magazine/object alignment
moving part paths
mirror-safe authored data
editor preview checks
```

It does not define inventory, ammo economy, damage, projectile behavior, camera behavior, or locomotion speed.

---

## General Rule

```text
Author the weapon once in canonical interaction space.
Validate canonical data.
Preview mirrored presentation separately.
```

Do not author duplicate left-side and right-side interaction plans unless a specific non-mirrored weapon mesh/control layout requires a separate physical interaction profile.

---

## Required Core Sockets

Most held weapons should define:

```text
MainGripSocket
SupportGripSocket optional or required by archetype
MuzzleSocket
WeaponRootSocket or RootBone reference
```

Stocked weapons should also define:

```text
StockShoulderSocket
```

Weapons with detachable magazines should define:

```text
MagazineWellSocket
MagazineLockSocket optional
```

Weapons with moving controls may define:

```text
BoltHandleSocket
ChargingHandleSocket
SlideSocket
PumpSocket
LeverSocket
SafetyControlSocket
```

---

## Main Grip Socket

`MainGripSocket` should represent the stable hand grip point.

Rules:

```text
place at the palm contact center
orient so hand can align without extreme wrist correction
keep local axes consistent across weapon family
avoid placing it at trigger finger tip only
```

Validation:

```text
socket exists
socket transform is finite
hand preview can reach it
wrist rotation is within allowed range
```

---

## Support Grip Socket

`SupportGripSocket` should represent the default support hand contact.

Rules:

```text
place where the support palm naturally stabilizes the weapon
avoid placing too close to main grip unless weapon is compact
ensure support hand can release and return cleanly
```

For weapons with multiple support points, author named interaction points:

```text
SupportGrip_Default
SupportGrip_Foregrip
SupportGrip_Magwell
SupportGrip_Pump
```

---

## Stock Shoulder Socket

`StockShoulderSocket` should represent the intended shoulder contact point.

Rules:

```text
place at the point that rests against the shoulder
orient forward axis consistently with weapon forward
validate ADS and hip-fire shoulder contact preview
```

Stock contact should be authored as a contact point, not as a body orientation rule.

---

## Magazine Well Socket And Axes

Magazine interaction requires semantic axes.

Required authored values:

```text
MagazineWellSocket
LocalInsertAxis
LocalExtractAxis
InsertDistance
ExtractDistance
PreInsertOffset
```

Rules:

```text
LocalInsertAxis points in the direction the magazine travels into the weapon
LocalExtractAxis points in the direction the magazine travels out of the weapon
axes are local to the weapon or interaction point
axes must not be inferred from arbitrary start/end positions
```

Validation:

```text
axis length is non-zero
insert/extract axes are expected opposite or authored as valid special case
preview object aligns with magazine well
mirrored preview transforms axes correctly
```

---

## Reload Object Sockets

Reload objects such as magazines, shells, rounds, batteries, or clips should define:

```text
HandGripSocket
InsertTipSocket
LockSocket optional
```

Rules:

```text
HandGripSocket is where the hand holds the object
InsertTipSocket is the part that aligns to the weapon interaction point
LockSocket is optional final seated alignment reference
```

Alignment formula:

```text
DesiredObjectWorldTransform =
  TargetWeaponPointWorldTransform * Inverse(ObjectInsertTipLocalTransform)
```

---

## Moving Part Authoring

Moving parts require named bone/socket and travel data.

Examples:

```text
Bolt
Slide
Pump
ChargingHandle
Lever
```

Author:

```text
MovingPartName
BoneName or ComponentName
FollowSocketName optional
LocalTravelAxis
TravelDistance
RestPosition
CommittedPosition optional
```

Rules:

```text
travel axis is semantic
hand target follows moving part socket when gripping the part
moving part visual state must match interaction commit state
```

---

## Mirror-Safe Authoring

Author canonical interaction data once.

Check mirrored presentation for:

```text
hand targets
elbow poles
magazine path
object insert axis
moving part path
stock contact
muzzle/sight visual alignment
```

Do not author fake mirrored controls unless the weapon profile explicitly represents a mirrored mesh/control layout.

---

## Naming Rules

Recommended naming style:

```text
Socket_MainGrip
Socket_SupportGrip_Default
Socket_StockShoulder
Socket_Muzzle
Socket_MagazineWell
Socket_BoltHandle
Socket_PumpGrip
Socket_Object_InsertTip
Socket_Object_HandGrip
```

Interaction point ids should be stable:

```text
MainGrip
SupportGrip
StockShoulder
MagazineWell
BoltHandle
PumpGrip
```

Do not rename ids casually after gameplay/animation data references them.

---

## Preview Checklist

Before accepting an authored weapon profile, preview:

```text
canonical hold pose
mirrored presentation hold pose
LowReady
HipFire
ADS interaction pose
reload path
object alignment
moving part operation
support hand release/recover
draw/holster if supported
network phase scrub if available
```

---

## Common Authoring Errors

```text
missing required socket
axis points opposite semantic direction
axis inferred from positions instead of authored local direction
magazine insert tip placed at wrong end
support grip too far for arm reach
stock socket too high/low for shoulder contact
moving part travel distance too short/long
mirrored preview not checked
object visual duplicates hidden by preview camera
socket renamed after sequence profile was authored
```

---

## Final Formula

```text
Good weapon interaction authoring =
  stable canonical sockets
  + semantic local axes
  + object alignment sockets
  + moving part travel data
  + mirror-safe preview
  + validation before runtime.
```
