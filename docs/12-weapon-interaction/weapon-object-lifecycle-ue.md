---
id: weapon-reload-object-lifecycle-for-unreal-engine
title: Weapon Reload Object Lifecycle for Unreal Engine
status: draft
version: 26.602.1413
tags: [ weapon, unreal-engine, ue5.7, reload, objects, inventory, multiplayer ]
---

# Weapon Reload Object Lifecycle for Unreal Engine

## Purpose

This document defines how reload objects exist, move, attach, detach, replicate, and reconcile in an Unreal Engine multiplayer implementation.

It covers magazines, rounds, shells, batteries, energy cells, and other reload objects.

Conceptual attachment states are defined in [Weapon Interaction Data Model](./weapon-interaction-data-model.md). Multiplayer authority is defined in [Weapon Interaction Networking](./weapon-networking.md). Runtime execution is defined in [Weapon Runtime Implementation for Unreal Engine](./weapon-runtime-implementation-ue.md).

---

## Core Rule

Gameplay object state and visual object state are separate.

```text
Gameplay object:
  server-authoritative inventory/ammo/loot state.

Visual object:
  local representation used to show hands moving magazines, rounds, shells, or cells.
```

A visual object may be predicted. Gameplay object state must be committed by the server.

---

## Object Types

Supported reload object types:

```text
Magazine
Round
ShotgunShell
Battery
EnergyCell
Clip
SpeedLoader
Custom
```

UE representation:

```cpp
UENUM(BlueprintType)
enum class EReloadObjectType : uint8
{
    None,
    Magazine,
    Round,
    ShotgunShell,
    Battery,
    EnergyCell,
    Clip,
    SpeedLoader,
    Custom
};
```

---

## Object Runtime Roles

A reload object may have multiple runtime representations:

```text
Inventory item entry:
  authoritative gameplay inventory data.

Weapon-attached object actor:
  visible magazine/cell attached to weapon.

Hand-held visual actor:
  visible object attached to character hand during manipulation.

World dropped actor:
  replicated actor if object can be picked up.

Hidden visual-only object:
  cosmetic object used for local animation only.
```

Do not assume one magazine always equals one actor for its entire lifetime.

---

## Object Actor Interface

Recommended interface:

```cpp
UINTERFACE(BlueprintType)
class UReloadObjectVisualInterface : public UInterface
{
    GENERATED_BODY()
};

class IReloadObjectVisualInterface
{
    GENERATED_BODY()

public:
    virtual EReloadObjectType GetReloadObjectType() const = 0;
    virtual FGameplayTagContainer GetCompatibilityTags() const = 0;
    virtual FName GetHandGripSocketName() const = 0;
    virtual FName GetInsertTipSocketName() const = 0;
    virtual FName GetLockSocketName() const = 0;
};
```

This interface lets the manipulation component align objects without knowing the exact class.

---

## Visual Object State

Use the visual state enum from [Weapon Runtime Implementation for Unreal Engine](./weapon-runtime-implementation-ue.md):

```text
Hidden
InWeapon
InLeftHand
InRightHand
InBodySlot
DroppedWorld
```

This state is visual. It does not by itself change ammo or inventory.

---

## Lifecycle: Fetch From Body Slot

Typical visual flow:

```text
1. Planner selects body slot and object type.
2. Owning client may spawn predicted visual object.
3. Visual object attaches to hand at grip/contact phase.
4. Server confirms reload step through replicated reload state.
5. If rejected, predicted visual is destroyed or returned to body slot visual.
```

Gameplay inventory is not consumed until the server commit policy says so.

---

## Lifecycle: Insert Into Weapon

Typical visual flow:

```text
1. Object visual is held by hand.
2. InsertTipSocket aligns to weapon interaction point.
3. Object moves along insert path.
4. Object visual attaches to weapon at lock/contact phase.
5. Server applies MagazineLocked or equivalent commit point.
6. Mechanical state revision increments.
```

If the visual attachment happens before the server commit, the client must reconcile if the server rejects or corrects the state.

---

## Lifecycle: Remove From Weapon

Typical visual flow:

```text
1. Existing weapon-attached visual object is shown in weapon.
2. Hand reaches and grips object.
3. Object becomes visually attached to hand.
4. Object extracts along extract path.
5. Server applies MagazineDetached or equivalent commit point.
6. Object is stowed, dropped, or hidden depending on reload intent.
```

A tactical reload usually stows. An emergency reload usually drops.

---

## Lifecycle: Drop Object

Two drop modes exist:

```text
Cosmetic drop:
  local visual only, not lootable, may be destroyed after a delay.

Gameplay drop:
  server spawns a replicated world actor, can be picked up or interacted with.
```

Rules:

```text
Only server spawns authoritative lootable dropped objects.
Clients may spawn cosmetic-only drop visuals.
Predicted cosmetic drop must be replaced or removed if server spawns a real dropped actor.
```

---

## Predicted Object Reconciliation

Owning client prediction can create temporary visuals.

On server confirmation:

```text
if predicted visual matches server state:
  keep visual and align to replicated phase.

if server provides authoritative object actor:
  bind visual to server actor or replace predicted actor.

if server rejects:
  destroy predicted visual or blend it back to body/weapon state.
```

Do not leave both predicted and authoritative magazines visible.

---

## Object Identity

Recommended state fields:

```text
ReloadObjectType
GameplayObjectId optional
VisualObjectActor optional
SourceBodySlotId optional
TargetWeaponPoint optional
```

`GameplayObjectId` is needed when inventory contains multiple distinct magazines with different ammo counts.

If all magazines are abstract ammo pools, `GameplayObjectId` may be omitted for MVP.

---

## Body Slot Visuals

Body slot visuals should be decoupled from inventory authority.

Examples:

```text
inventory says 3 magazines available
body slot visuals show 1 or more visible pouches/magazines
reload fetch hides or swaps a visual magazine
server confirms inventory change at commit point
```

If body slot visuals are purely cosmetic, they can be locally updated and corrected on inventory replication.

---

## Attachment Implementation

Recommended attach targets:

```text
weapon socket
left hand socket
right hand socket
body slot socket
world transform
```

Attach rules:

```text
attach with SnapToTarget for socket-accurate objects
use KeepWorldTransform when dropping to world
use object InsertTipSocket for alignment, not actor root
use object HandGripSocket for hand attachment, not actor root
```

---

## Alignment Rule

Magazine/round alignment uses object socket transforms.

```text
DesiredObjectWorldTransform =
  TargetWeaponPointWorldTransform
  * Inverse(ObjectInsertTipLocalTransform)
```

Hand attachment uses hand grip point:

```text
DesiredObjectWorldTransform =
  HandSocketWorldTransform
  * Inverse(ObjectHandGripLocalTransform)
```

This avoids requiring actor root to be placed at a convenient point.

---

## Server Authority Rules

Server owns:

```text
inventory object existence
ammo count in magazine/object
magazine locked into weapon
round inserted/chambered
lootable dropped object spawning
object pickup eligibility
```

Client owns:

```text
visual interpolation
predicted visual object spawn
cosmetic-only drop effects
hand attachment display
```

---

## Failure Cases

Implementation must handle:

```text
server rejects predicted reload after visual object spawned
server accepts but different magazine object is selected
remote client becomes relevant mid reload
old magazine is dropped but not lootable
old magazine is dropped and lootable
object actor is destroyed before visual sequence ends
body slot becomes invalid during reload
weapon is switched mid reload
```

Every case must recover to a valid visual and gameplay state.

---

## MVP Object Lifecycle

Minimum implementation:

```text
1. Visual magazine actor can attach to weapon magazine socket.
2. Visual magazine actor can attach to left/right hand socket.
3. Old magazine can be hidden or cosmetically dropped.
4. New magazine can be spawned visually from selected body slot.
5. Server commits MagazineDetached and MagazineLocked.
6. Client removes duplicate predicted visuals after server confirmation.
7. Remote client reconstructs object visual state from ReloadInstance.
```

---

## Debug Requirements

Debug should show:

```text
visual object actor
reload object type
gameplay object id
source body slot
target weapon point
current visual state
server commit state
predicted vs confirmed visual
attachment parent/socket
alignment transform
```

---

## Final Formula

```text
Reload object lifecycle =
  server-owned gameplay object state
  + locally animated visual object state
  + explicit attachment transitions
  + commit-point reconciliation
  + duplicate visual cleanup.
```
