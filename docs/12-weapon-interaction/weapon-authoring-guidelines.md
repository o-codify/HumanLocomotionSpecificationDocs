---
id: weapon-interaction-authoring-guidelines
title: Weapon Interaction Authoring Guidelines
status: draft
version: 26.602.1555
tags: [ weapon, authoring, sockets, axes, editor, validation ]
---

# Weapon Interaction Authoring Guidelines

## Purpose

This document gives practical authoring rules for weapon interaction assets.

It is written for technical animators, gameplay animators, weapon artists, and engineers who create weapon profiles, sockets, axes, reload object profiles, contact references, and preview data.

---

## Scope Boundary

This document describes authoring of weapon interaction data:

```text
sockets
interaction points
local axes
hand grip points
support grip points
hand-to-hand support references
stock/shoulder contact
cheek/sight alignment references for stocked ADS presentation
magazine/object alignment
moving part paths
mirror-safe authored data
editor preview checks
```

It does not define inventory, ammo economy, damage, projectile behavior, camera behavior, body orientation, or locomotion speed.

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
MuzzleSocket
WeaponRootSocket or RootBone reference
```

Depending on archetype, also define:

```text
SupportGripSocket optional or required for long guns
HandToHandSupportReference optional/required for two-handed handgun poses
StockShoulderSocket for stocked weapons
CheekReferenceSocket optional/preferred for stocked ADS presentation
SightReferenceSocket optional/preferred for ADS presentation
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

`SupportGripSocket` should represent a weapon-side support hand contact.

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

Do not force pistols into a fake support grip socket when their support is actually hand-to-hand.

---

## Hand-To-Hand Support Authoring For Handguns

Two-handed handgun poses often require support hand relation to the main hand instead of a separate weapon socket.

Author:

```text
HandToHandSupportReference
SupportHandRelativeToMainHandOffset
SupportHandGripPoseId
OptionalSupportToWeaponFrameReference
```

Rules:

```text
main hand remains the primary weapon contact
support hand may wrap around or brace the main hand
support hand may optionally align partly to weapon frame
combined two-hand grip pose should be previewed as a unit
```

Validation:

```text
support hand can reach relative pose
support hand does not intersect main hand excessively
finger pose supports combined grip
mirrored presentation preserves relative hand-to-hand relationship
```

---

## Stock Shoulder Socket

`StockShoulderSocket` should represent the intended shoulder contact point.

Rules:

```text
place at the point that rests against the shoulder
orient forward axis consistently with weapon forward
validate ADS and NonADSFire shoulder contact preview
```

Stock contact should be authored as a contact point, not as a body orientation rule.

---

## Cheek And Sight References For Stocked ADS

Stocked ADS presentation may need references for cheek/sight relation.

Optional authored references:

```text
CheekReferenceSocket
SightReferenceSocket
RearSightSocket optional
FrontSightSocket optional
EyeReliefReference optional
```

Rules:

```text
CheekReferenceSocket describes where the face/cheek relation should settle visually
SightReferenceSocket describes the sight alignment reference in weapon local space
EyeReliefReference describes preferred eye distance if the project models it visually
```

These references are for weapon interaction presentation. They do not own camera FOV, camera placement, body yaw, or turn-in-place.

Validation:

```text
ADS preview can settle shoulder contact
ADS preview can reach acceptable sight-eye alignment quality
mirrored presentation preserves sight/cheek relation
missing cheek/sight references warn only if the archetype/pose requires them
```

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
hand-to-hand support relation
elbow poles
magazine path
object insert axis
moving part path
stock contact
cheek/sight relation
muzzle/sight visual alignment
```

Do not author fake mirrored controls unless the weapon profile explicitly represents a mirrored mesh/control layout.

---

## Naming Rules

Recommended naming style:

```text
Socket_MainGrip
Socket_SupportGrip_Default
Socket_HandToHandSupportRef
Socket_StockShoulder
Socket_CheekReference
Socket_SightReference
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
HandToHandSupport
StockShoulder
CheekReference
SightReference
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
HipFire / NonADSFire
PointAim
ADS interaction pose
hand-to-hand handgun support if archetype uses it
shoulder/cheek/sight settling for stocked ADS if archetype uses it
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
pistol forced to use fake support socket instead of hand-to-hand support
stock socket too high/low for shoulder contact
cheek/sight reference missing for stocked ADS profile that requires it
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
  + archetype-specific contacts
  + semantic local axes
  + object alignment sockets
  + moving part travel data
  + hand-to-hand support data when needed
  + cheek/sight references when needed
  + mirror-safe preview
  + validation before runtime.
```
