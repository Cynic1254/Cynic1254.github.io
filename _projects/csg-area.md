---
title: "Dynamic geometry zones"
order: 4
summary: >
  An Unreal Engine plugin that uses real-time constructive solid geometry to clip object meshes and collision to a defined spherical zone, inspired by the Timeshift Stones from Zelda: Skyward Sword.
status: "Open Source"
status_color: "open"
tags:
  - C++
  - Unreal Engine
  - Constructive Solid Geometry
image: "/assets/images/CSG_Demo.webp"
image_alt: "Gameplay demo showing a CSG zone bound to a projectile, cutting through walls in real time"
link_source_label: "View on GitHub"
link_source_url: "https://github.com/Cynic1254/CSGArea"
---

## Overview

CSGArea is an Unreal Engine plugin that clips object meshes and their collision to a defined spherical zone in real time using constructive solid geometry. The mechanic is inspired by the Timeshift Stones in Zelda: Skyward Sword and the bells in A Hat in Time. Objects that straddle the zone boundary are sliced exactly at that boundary, both visually and physically.

The plugin was developed as a standalone block A project and was briefly considered as a mechanic for the Katharsi project.

## The problem

A visibility toggle isn't sufficient for this mechanic. An object that is half inside a zone needs to be half visible and have collision only where it is visible. A simple hide/show approach leaves invisible geometry still blocking the player, or visible geometry the player can walk through.

The solution is to treat the zone boundary as a cutting plane and recompute both the visual mesh and the collision shape every frame to match exactly how much of the object currently overlaps the zone.

## Implementation

The plugin has three main components:

`UCSGAreaComponent` is a sphere component that defines the zone boundary. It operates on a configurable collision channel so it only interacts with objects that have been set up to participate in the system.

`UCSGBaseComponent` is the core, extending `UDynamicMeshComponent`. Every tick it queries which `UCSGAreaComponent`s it is currently overlapping, then uses UE's Geometry Scripting plugin to compute the intersection of its mesh with each overlapping zone sphere. The results are unioned together to produce the final visible portion, and the collision shape is updated to match.

```cpp
void UCSGBaseComponent::RebuildMesh(UDynamicMesh* OutMesh, UDynamicMesh* FullMesh) const
{
    TArray Overlapping;
    GetOwner()->GetOverlappingComponents(Overlapping);

    TArray MeshPieces;

    for (const auto OverlappingComponent : Overlapping)
    {
        if (const auto* Component = Cast(OverlappingComponent))
        {
            UDynamicMesh* TempMesh = MeshPool->RequestMesh();

            UGeometryScriptLibrary_MeshPrimitiveFunctions::AppendSphereBox(
                TempMesh, {}, Component->GetComponentTransform(),
                Component->GetUnscaledSphereRadius());

            UGeometryScriptLibrary_MeshBooleanFunctions::ApplyMeshBoolean(
                TempMesh, {},
                BaseMesh, GetComponentTransform(),
                EGeometryScriptBooleanOperation::Intersection,
                {true, true, 0.01, true});

            MeshPieces.Push(TempMesh);
        }
    }

    DynamicMesh->Reset();

    for (const auto Piece : MeshPieces)
    {
        UGeometryScriptLibrary_MeshBooleanFunctions::ApplyMeshBoolean(
            DynamicMesh, GetComponentTransform(),
            Piece, {},
            EGeometryScriptBooleanOperation::Union,
            {true, true, 0.01, true});
    }
}
```

The component also supports a reverse mode (`bDoReverseCSG`) which subtracts the zone from the mesh instead of intersecting with it — useful for objects that should be visible everywhere *except* inside the zone.

`UCSGStaticMeshComponent` is a concrete implementation that takes a `UStaticMesh` as its input. The base class exposes `GetVisualMesh` and `GetCollisionMesh` as `BlueprintNativeEvent`s, so custom mesh sources can be provided from Blueprint without touching C++. A `UDynamicMeshPool` is used to avoid allocating new mesh objects every tick.

## Performance

The CSG calculation currently runs every tick regardless of whether the zone or the object has moved. For simple geometry this performs acceptably, the initial performance concerns that led to the plugin being dropped from Katharsi turned out to be the result of testing against a high-polygon mannequin mesh rather than the kind of simple level geometry the mechanic was designed for. Limiting the per-tick recalculation to only run when relevant state changes would be the obvious next step if development continued.

## Current status

The plugin is feature-complete and available on GitHub. It supports both intersection and subtractive CSG modes, multiple overlapping zones, separate visual and collision mesh sources, and has editor preview meshes for both the zone and the object to make placement easier.