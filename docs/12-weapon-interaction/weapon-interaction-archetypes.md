---
id: weapon-interaction-archetypes
title: Weapon Interaction Archetypes
status: draft
version: 26.602.2053
tags: [ weapon, interaction, archetypes, contacts, animation ]
---

# Weapon Interaction Archetypes

## Purpose

This document defines common weapon interaction archetypes.

An archetype is not a gameplay weapon class, damage model, ammo system, or inventory category. It is an interaction preset describing contacts, manipulation patterns, moving parts, and authoring expectations.

---

## Scope Boundary

This document owns:

```text
required contacts
optional contacts
support model
reload/manipulation contact expectations
moving part expectations
ADS/contact expectations
what may be released
what must not be released
```

It does not own:

```text
damage
ballistics
ammo economy
weapon balance
inventory slots
camera behavior
locomotion speed
AI behavior
```

---

## Archetype Table

| Archetype | Required Contacts | Optional Contacts | Common Manipulation | Notes |
|---|---|---|---|---|
| One-Handed Pistol | MainGrip | HandToHandSupport | magazine insert, slide operation | support is optional and may be hand-to-hand |
| Two-Handed Handgun | MainGrip + HandToHandSupport | SupportToFrame | magazine insert, slide operation | support hand often braces main hand |
| Stocked Rifle | MainGrip + SupportGrip | Shoulder, Cheek/Sight | magazine insert, charging handle, selector | stock/shoulder improves stability |
| Bullpup Rifle | MainGrip + SupportGrip | Shoulder, Cheek/Sight | rear magazine, charging handle | rear access may affect reachability |
| Pump Shotgun | MainGrip + PumpGrip | Shoulder | pump cycle, shell loading | pump grip is both support and moving part |
| Bolt-Action Rifle | MainGrip + Support/Shoulder | Cheek/Sight | bolt operation, magazine/clip | bolt hand policy depends on support stability |
| Break-Action Weapon | MainGrip or Support + hinge support | Shoulder optional | break open, insert shells, close | hinge state is core moving part |
| Heavy Weapon | MainGrip + Support or external support | Bipod/Surface/Sling | charging, belt/box manipulation | release rules stricter |
| Launcher / Sci-Fi Heavy | MainGrip + Support | Shoulder/Surface | cell/core insertion, latch/cover | object insertion profile often custom |

---

## One-Handed Pistol

Required:

```text
MainGripContact
```

Optional:

```text
HandToHandSupportContact
```

Typical interactions:

```text
magazine extraction/insertion
slide pull/release
safety/selector press
```

Rules:

```text
do not require fake SupportGripSocket
main hand usually cannot release unless another authored support exists
ADS/NonADSFire may differ mostly by pose and aim alignment quality
```

---

## Two-Handed Handgun

Required:

```text
MainGripContact
HandToHandSupportContact for two-handed pose
```

Typical support model:

```text
support hand braces main hand
support hand may partially contact weapon frame
combined two-hand grip pose is authored as a relationship
```

Rules:

```text
support hand release degrades stability
support hand returns to hand-to-hand support, not necessarily a weapon socket
```

---

## Stocked Rifle / Long Gun

Required:

```text
MainGripContact
SupportGripContact for stable two-handed pose
```

Preferred:

```text
ShoulderContact
CheekContact for ADS presentation
SightEyeAlignmentQuality for ADS-ready presentation
```

Common interactions:

```text
magazine reload
charging handle
bolt release
selector manipulation
```

Rules:

```text
support hand may leave temporarily if main grip + shoulder contact keep weapon stable
ADS should settle shoulder/sight relation before reporting ADS-ready presentation
```

---

## Bullpup Rifle

Bullpup rifles have rearward magazine/action placement.

Interaction concerns:

```text
rear magazine well may be harder to reach
support hand may need larger pose offset
weapon may roll/tilt to expose magazine well
```

Rules:

```text
rear access should be authored explicitly
reachability must not assume rifle magazine is always forward of trigger area
```

---

## Pump Shotgun

Pump grip is both support contact and moving part contact.

Required:

```text
MainGripContact
PumpGripContact
```

Common interactions:

```text
pump back
pump forward
shell loading through port/tube
```

Rules:

```text
pump hand follows moving part during pump cycle
pump contact may be support contact when not cycling
pump axis must be authored semantically
```

---

## Bolt-Action Rifle

Common interactions:

```text
bolt lift
bolt pull
bolt push
bolt lock
stripper clip or magazine manipulation if supported
```

Rules:

```text
bolt operation may require main hand to leave grip if authored
weapon must remain stable through support/shoulder contacts
bolt handle path may combine rotation and translation
```

---

## Break-Action Weapon

Common interactions:

```text
open hinge
extract/eject shells
insert shells
close hinge
```

Rules:

```text
hinge angle is a moving part state
shell insertion is object insertion, not inventory logic
support contact must keep weapon controlled while open
```

---

## Heavy Weapon

Heavy weapons may require external support.

Contacts:

```text
MainGrip
SupportGrip
BipodContact optional/required
SurfaceContact optional/required
SlingContact optional
```

Rules:

```text
do not allow release of critical contact unless external support is active
manipulation may require lower/brace pose
LOD may simplify visuals but not support semantics
```

---

## Launcher / Sci-Fi Heavy Weapon

Common interactions:

```text
cell insertion
core replacement
latch open/close
side port manipulation
charging handle
```

Rules:

```text
use same interaction primitives: object, socket, axis, commit, contact quality
avoid inventing inventory mechanics inside interaction profile
custom object insertion paths should still expose semantic phases
```

---

## Debug Requirements

Debug should show:

```text
weapon archetype
required contacts
optional contacts
current contact satisfaction
archetype-specific support model
blocked release reason
missing authored data for archetype
```

---

## Final Formula

```text
Weapon archetype =
  contact requirements
  + support model
  + manipulation expectations
  + moving part expectations
  + authoring validation hints.
```
