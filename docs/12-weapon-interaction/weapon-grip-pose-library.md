---
id: weapon-grip-pose-library
title: Weapon Grip Pose Library
status: draft
version: 26.602.2053
tags: [ weapon, interaction, grip, hands, animation ]
---

# Weapon Grip Pose Library

## Purpose

This document defines a shared library of grip pose ids for weapon interaction.

Grip poses are animation/control-rig hand shapes used by weapon interaction. They are not inventory items, gameplay weapon classes, or damage rules.

---

## Scope Boundary

This document owns:

```text
grip pose ids
hand shape intent
finger grip alpha meaning
contact pose expectations
object grip expectations
hand-to-hand support poses
```

It does not own:

```text
weapon damage
ammo economy
projectile behavior
camera behavior
locomotion speed
full physical grip force simulation
```

---

## Core Rule

```text
GripPoseId selects the hand shape.
ContactQuality says how well the shape/contact is currently satisfied.
FingerGripAlpha blends the pose in/out.
```

Do not use one generic fist pose for every weapon interaction.

---

## Common Grip Pose IDs

Recommended ids:

```text
Grip.Pistol.Main
Grip.Handgun.SupportWrap
Grip.Rifle.Main
Grip.Rifle.Support
Grip.Foregrip.Vertical
Grip.Foregrip.Angled
Grip.Pump
Grip.Magazine.Body
Grip.Magazine.Baseplate
Grip.Shell.Pinch
Grip.Round.Pinch
Grip.ChargingHandle.Pinch
Grip.BoltKnob
Grip.Lever
Grip.Safety.Toggle
Grip.Button.Press
Grip.Latch.Pull
Grip.HeavyWeapon.Support
Grip.Object.Core
Grip.Object.Battery
```

---

## Main Weapon Grips

### Pistol Main Grip

```text
Grip.Pistol.Main
```

Used for:

```text
one-handed pistol
two-handed handgun main hand
```

Expected contact:

```text
palm wraps around pistol grip
trigger finger may be authored separately
wrist orientation should avoid extreme roll
```

### Rifle Main Grip

```text
Grip.Rifle.Main
```

Used for:

```text
rifle pistol grip
stocked long gun main grip
bullpup main grip
```

---

## Support Grips

### Rifle Support Grip

```text
Grip.Rifle.Support
```

Used for:

```text
standard fore-end support
magwell support if authored
long gun support hand
```

### Vertical Foregrip

```text
Grip.Foregrip.Vertical
```

### Angled Foregrip

```text
Grip.Foregrip.Angled
```

### Pump Grip

```text
Grip.Pump
```

Used when a support contact is also a moving part contact.

---

## Handgun Support Grip

```text
Grip.Handgun.SupportWrap
```

Used for:

```text
two-handed handgun support pose
support hand wraps/braces main hand
support hand may partially contact weapon frame
```

This grip is usually solved as hand-to-hand support, not as a separate weapon socket grip.

---

## Object Grips

### Magazine Grip

```text
Grip.Magazine.Body
Grip.Magazine.Baseplate
```

Used for:

```text
magazine extraction
magazine insertion
magazine visual handling
```

### Shell / Round Pinch

```text
Grip.Shell.Pinch
Grip.Round.Pinch
```

Used for:

```text
shotgun shell loading
single round insertion
clip/round manipulation
```

### Core / Battery Grip

```text
Grip.Object.Core
Grip.Object.Battery
```

Used for:

```text
sci-fi cell/core insertion
battery replacement
```

---

## Moving Part Grips

```text
Grip.ChargingHandle.Pinch
Grip.BoltKnob
Grip.Lever
Grip.Latch.Pull
```

Rules:

```text
moving part grip must match authored moving part operation profile
hand target follows moving part follow socket when active
finger/grip pose may release before moving part returns if authored
```

---

## Controls

```text
Grip.Safety.Toggle
Grip.Button.Press
```

These may use finger-specific poses if the rig supports them.

MVP may approximate with hand/finger alpha.

---

## Grip Pose Data

Suggested conceptual data:

```text
GripPose:
  GripPoseId
  HandShapeAsset optional
  FingerCurlMask optional
  WristOffset
  PalmContactReference
  PreferredContactType
  MinContactQuality
  MirroringPolicy
```

---

## Mirroring

Grip pose ids remain canonical.

Presentation mirroring mirrors the final hand pose/target when the global presentation system mirrors the character.

Do not create separate grip ids such as:

```text
Grip.Pistol.Main.LeftMirrored
```

unless a specific rig/asset pipeline requires a separate presentation-only asset name.

---

## Debug Requirements

Debug should show:

```text
GripPoseId
FingerGripAlpha
ContactQuality
ContactState
PalmContactReference
MirroringPolicy
MissingGripAssetWarning
```

---

## Final Formula

```text
Grip pose library =
  named hand shapes
  + contact expectations
  + object/moving-part grip variants
  + mirror-safe ids
  + animation/control-rig consumption.
```
