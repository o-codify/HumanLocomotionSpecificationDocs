---
id: introduction
title: Introduction
status: review
version: 26.604.1324
tags: [ scope, policy, engine-agnostic, human-motion ]
---

# Introduction

## Purpose

Human Locomotion Specification, abbreviated HLS, is a formal description of how the human body moves.

HLS exists for AI systems that need exact, testable movement constraints instead of vague animation instructions such as:

```text
move a little wider
lean slightly left
make the step feel heavier
turn more naturally
```

The specification converts human locomotion knowledge into numeric, phase-based, and constraint-based descriptions.

The target reader is not an animator, engine programmer, or game designer. The target reader is an AI system, simulator, validator, or procedural pose generator that needs to reason about human movement with explicit values.

---

## Scope

HLS describes:

```text
body segment proportions
joint degrees of freedom
joint range limits
walking gait phases
running gait phases
postural alignment
center of mass behavior
base of support
balance constraints
foot contact timing
stride length and cadence ranges
pelvis motion
spine counter-rotation
arm swing coordination
turning mechanics
starting and stopping mechanics
slope and stair adaptation
load carriage effects
weapon or object carry effects
injury and limping effects
```

HLS does not prescribe one implementation method.

---

## Non-Goals

HLS must not contain:

```text
engine-specific implementation instructions
Unreal Engine instructions
Unity instructions
Control Rig instructions
Blueprint or C++ class plans
network replication plans
asset authoring roadmaps
gameplay feature design
visual effect requirements
project management task lists
```

A document may define a mathematical output such as foot target position, pelvis height, trunk angle, or phase timing. It must not describe how a specific engine should compute, store, replicate, or render that output.

---

## Required Style

Every movement description should prefer measurable language.

Bad:

```text
step a bit wider
lean slightly forward
make the stance more aggressive
turn naturally
```

Good:

```text
step width = 0.08-0.16 * bodyHeight
trunk pitch = 4-10 degrees forward
pelvis yaw leads thorax yaw by 5-15 degrees during turn initiation
stance phase = 58-62% of walking gait cycle
```

If exact values are uncertain, the document must mark the value as an approximation and give a usable range.

---

## Coordinate Convention

Unless a document states otherwise, HLS uses a body-relative coordinate frame:

```text
+X = forward
+Y = subject left
+Z = upward
```

Angles use degrees.

Distances may be specified in meters or normalized by body height.

Normalized notation:

```text
H = body height
L_leg = functional leg length from hip joint center to ground contact
BW = body width at shoulders or pelvis, depending on context
```

---

## Human Model Assumption

HLS assumes a bipedal adult human unless a document explicitly defines a child, elderly, pathological, or non-standard case.

Default reference body:

```text
bodyHeight H = 1.70-1.85 m
bodyMass M = 60-90 kg
legLength L_leg = 0.50-0.55 * H
pelvisWidth = 0.14-0.19 * H
shoulderWidth = 0.22-0.28 * H
```

These values are not identity requirements. They are reference ranges for scaling motion.

---

## Quality Target

HLS targets motion that is:

```text
biomechanically plausible
numerically bounded
temporally coherent
phase-continuous
stable under modifier stacking
clear enough for AI generation and validation
```

The output does not need to be a full medical simulation. It must be good enough that an AI can distinguish plausible human movement from impossible, unstable, or visibly wrong movement.

---

## Core Principle

Human locomotion is not a list of poses.

It is a constrained, cyclic transfer of body mass over alternating support contacts:

```text
intent
→ phase timing
→ support state
→ center of mass trajectory
→ pelvis and trunk response
→ limb swing and contact placement
→ balance correction
→ next support state
```

HLS documents this chain in numeric terms.
