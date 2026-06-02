---
id: weapon-editor-preview-and-validation-for-unreal-engine
title: Weapon Editor Preview and Validation for Unreal Engine
status: draft
version: 26.602.1414
tags: [ weapon, unreal-engine, ue5.7, editor, validation, preview, debug ]
---

# Weapon Editor Preview and Validation for Unreal Engine

## Purpose

This document defines the Unreal Engine editor preview and validation tools required for weapon interaction profiles and reload sequences.

The weapon interaction system depends heavily on authored socket transforms, local axes, interaction points, reload objects, and hand assignment rules. Without editor preview, most authoring errors will appear as runtime animation bugs.

Profile authoring is defined in [Weapon Interaction Profile for Unreal Engine](./weapon-interaction-profile-ue.md). Reload sequence authoring is defined in [Weapon Reload Sequence Profile for Unreal Engine](./weapon-reload-sequence-profile-ue.md).

---

## Tool Ownership

Editor preview owns:

```text
profile validation visualization
socket axis visualization
interaction point inspection
reload sequence preview
right/left shoulder preview
hand assignment preview
pre-insert/final pose preview
object alignment preview
rejected reason display
```

It does not own runtime authority, replication, or actual gameplay state.

---

## Required Editor Tools

Recommended tools:

```text
AWeaponInteractionPreviewActor
UWeaponInteractionPreviewComponent
SWeaponInteractionProfilePreview or Editor Utility Widget
custom details panel optional
asset validation commandlet optional
```

MVP may use an Editor Utility Widget and preview actor placed in an editor preview map.

---

## Preview Actor

```cpp
UCLASS()
class AWeaponInteractionPreviewActor : public AActor
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Category="Preview")
    TObjectPtr<USkeletalMeshComponent> WeaponMesh;

    UPROPERTY(EditAnywhere, Category="Preview")
    TObjectPtr<UWeaponInteractionProfile> InteractionProfile;

    UPROPERTY(EditAnywhere, Category="Preview")
    TObjectPtr<UReloadSequenceProfile> ReloadSequenceProfile;

    UPROPERTY(EditAnywhere, Category="Preview")
    TObjectPtr<AActor> PreviewReloadObject;

    UPROPERTY(EditAnywhere, Category="Preview")
    EShoulderSide PreviewShoulderSide;

    UPROPERTY(EditAnywhere, Category="Preview")
    EReloadIntent PreviewIntent;

    UFUNCTION(CallInEditor)
    void ValidateProfile();

    UFUNCTION(CallInEditor)
    void PreviewHandAssignment();

    UFUNCTION(CallInEditor)
    void PreviewReloadSequence();

    UFUNCTION(CallInEditor)
    void DrawInteractionDebug();
};
```

---

## Preview Component

```cpp
UCLASS(ClassGroup=(Weapon), meta=(BlueprintSpawnableComponent))
class UWeaponInteractionPreviewComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    bool ValidateProfileAgainstMesh(
        const UWeaponInteractionProfile* Profile,
        const USkeletalMeshComponent* Mesh,
        FWeaponProfileValidationReport& OutReport) const;

    bool ValidateReloadSequenceAgainstProfile(
        const UReloadSequenceProfile* Sequence,
        const UWeaponInteractionProfile* Profile,
        FWeaponProfileValidationReport& OutReport) const;

    void DrawSocketAxes(
        const UWeaponInteractionProfile* Profile,
        const USkeletalMeshComponent* Mesh) const;

    void DrawMagazineAlignmentPreview(
        const UWeaponInteractionProfile* Profile,
        const AActor* ReloadObject) const;

    void SimulateHandAssignmentPreview(
        const UWeaponInteractionProfile* Profile,
        EShoulderSide ShoulderSide,
        FWeaponPreviewHandAssignmentReport& OutReport) const;
};
```

---

## Validation Report

```cpp
USTRUCT(BlueprintType)
struct FWeaponProfileValidationReport
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly)
    bool bValid = true;

    UPROPERTY(BlueprintReadOnly)
    TArray<FText> Errors;

    UPROPERTY(BlueprintReadOnly)
    TArray<FText> Warnings;

    UPROPERTY(BlueprintReadOnly)
    TArray<FName> MissingSockets;

    UPROPERTY(BlueprintReadOnly)
    TArray<FName> MissingBones;

    UPROPERTY(BlueprintReadOnly)
    TArray<FName> InvalidAxisPoints;

    UPROPERTY(BlueprintReadOnly)
    TArray<FGameplayTag> RejectedReasons;
};
```

Validation report should be readable by humans and by automated tests.

---

## Profile Validation Checks

Validate:

```text
profile assigned
weapon mesh assigned
all required sockets exist
all required moving bones exist
all required follow sockets exist
axis vectors are non-zero
feature flags match required interaction points
stock feature has stock shoulder socket
pump feature has pump moving part and pump point
bolt feature has bolt/charging point and operate axis
magazine feature has magazine well and insert/extract axes
all interaction point names are unique
all moving part names are unique
```

---

## Reload Sequence Validation Checks

Validate:

```text
sequence id valid
step ids unique
step durations > 0
phase alpha values ordered
referenced interaction points exist
referenced moving parts exist
commit points are valid gameplay tags
visual object states are compatible with step type
recovery policy exists if step can interrupt
```

Cross-validation requires both sequence profile and weapon profile.

---

## Debug Drawing

Preview should draw:

```text
interaction point name
socket local +X/+Y/+Z axes
insert direction
extract direction
operate direction
pre-insert pose
final insert pose
rock-in pivot and arc
moving part axis
hand target preview
elbow pole preview
body slot reach preview
rejected hand reasons
```

Axis color convention can follow project debug conventions.

---

## Hand Assignment Preview

Preview both shoulders:

```text
RightShoulder preview
LeftShoulder preview
```

For each interaction point show:

```text
preferred hand
resolved hand
cost
requires regrip
requires weapon pose adjustment
rejected left hand reasons
rejected right hand reasons
required stabilization contacts
```

This catches incorrect access regions and mirrored shoulder bugs before runtime.

---

## Magazine Alignment Preview

Preview should compute:

```text
PreInsertPose
FinalInsertPose
InsertDirection
ExtractDirection
DesiredMagazineWorldTransform
```

The preview must use object `InsertTipSocket`, not actor root.

If the object lacks an insert tip, validation should fail for magazine/round insertion preview.

---

## Reload Sequence Preview

Sequence preview should step through:

```text
step index
step id
step type
phase
duration
commit point
visual object state
moving part alpha
hand assignment
```

Controls:

```text
play
pause
scrub alpha
next step
previous step
preview right shoulder
preview left shoulder
show debug axes
show rejected reasons
```

Preview is local/editor-only and does not mutate gameplay inventory.

---

## Automated Validation

Recommended automation:

```text
validate all weapon profiles
validate all reload sequence profiles
validate profile + sequence combinations
fail build or content validation if required profile data is invalid
```

This can run as:

```text
editor utility
commandlet
data validation pass
content validation test
```

---

## Common Authoring Errors

The preview tool should explicitly detect:

```text
MagazineWell socket exists but +X points wrong direction
InsertAxis is zero
ExtractAxis is same as insert when weapon expects custom extraction
Stock feature enabled but stock socket missing
Pump feature enabled but pump moving part missing
Bolt point references missing socket
Reload sequence references missing interaction point
Magazine object has no InsertTipSocket
Hand grip socket is at actor root by mistake
Left shoulder preview uses wrong hand
```

---

## MVP Editor Tool

Minimum implementation:

```text
1. Preview actor with weapon mesh/profile/sequence fields.
2. Validate Profile button.
3. Draw Interaction Axes button.
4. Preview Magazine Alignment button.
5. Preview Hand Assignment for both shoulders.
6. Preview Reload Sequence step list.
7. Human-readable error/warning report.
```

---

## Final Formula

```text
Weapon editor preview =
  profile validation
  + sequence validation
  + socket axis debug
  + object alignment preview
  + hand assignment simulation
  + reload step preview.
```

The editor tool prevents authored data errors from becoming runtime animation bugs.
