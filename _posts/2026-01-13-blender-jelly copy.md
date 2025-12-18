---
title: "[Blender] Jell"
excerpt: "본문의 주요 내용을 여기에 입력하세요"

categories:
  - Blender
tags:
  - []

permalink: /swift/concurrency/

toc: true
toc_sticky: true

date: 2025-12-23
last_modified_at: 2025-12-23
---


# Subdivide

Edit Mode > Subdivide

<img width="431" height="316" alt="Image" src="https://github.com/user-attachments/assets/f9c9d664-0168-4820-91e5-efad9b60e164" />

Face, edge 를 더 작은 단위로 나눠서 mesh 의 해상도를 높임

https://docs.blender.org/manual/en/latest/modeling/meshes/editing/edge/subdivide.html

* Difference between subivision surface?
  * subdivide : destructive action
  * subdivision surface modifier : non-destructive action


# Soft Body

https://docs.blender.org/manual/en/latest/physics/soft_body/introduction.html#typical-scenarios-for-using-soft-bodies

Used for simulating soft deformable(변형가능) objects. Designed primarily for adding secondary motion to animation. (e.g. jiggle)

Simulating **soft objects** that bend, deform, react to forces like gravity, wind

## Goal

Use motions from animations in the simulation.

"Desired end poisition for vertices" based on animation. Lets control which parts of the object stay rigid and which parts can move and deform freely.

Disable goal for jelly to prevent floating around.

https://www.youtube.com/watch?v=JsWvv0N1i8g

## Stiffness

Edges : Make mesh act like spring

Soft Body > Edges > Stiffness 

## Bending

Soft Body > Edges > Bending

Controls how much bending occurs in simulation

## Plasticity

Soft Body > Edges > Plasticity

Allows to deform the object after a collision permanently.

High value > more permanent deformation
