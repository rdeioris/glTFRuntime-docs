# Managing LODs

There are various ways for managing LODs, you may directly get them using one of the supported extensions (like MSFT_lod), combining multiple meshes from an asset, or combining meshes from multiple assets.

## Quick and dirty way: MSFT_lod (StaticMeshes only)

This is an extension by Microsoft: https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Vendor/MSFT_lod

You can combine assets (as LODs) in a single one by using the glTF-Toolkit tool: https://github.com/microsoft/glTF-Toolkit/releases

Example:

```sh
WindowsMRAssetConverter.exe cube.glb -o lods.glb -lod sphere.glb -screen-coverage 0.4 0.1
```

This will create a new asset called lods.glb with the cube as the LOD0 (with screen percentage between max and 0.4) and sphere.glb as LOD1 (with screen percentage between 0.4 and 0.1).

MSFT_lod is automatically honoured when using the AglTFRuntimeAssetActor

## Combine multiple meshes from the same asset (StaticMeshes and SkeletalMeshes)

Here we are going to use the LoadStaticMeshLODs (or LoadSkeletalMeshLODs) that takes an array of mesh indices as input (you obviously need to know which one to use)

![LoadStaticMeshLODs](Docs/Assets/LoadStaticMeshLODs.png?raw=true "LoadStaticMeshLODs")

Notice how you can specify the LOD ScreenSize using the StaticMeshConfig structure

## Combine multiple meshes from the multiple assets (StaticMeshes and SkeletalMeshes)

Static Meshes (as well as SkeletalMeshes) can be build using an opaque data structure called the RuntimeLOD. You can fill RuntimeLODs from various assets and finally combine them:

![RuntimeLODs](Docs/Assets/RuntimeLODs.png?raw=true "RuntimeLODs")

## Notes:

if a lower LOD does not specify a material, it will inherit the first available one in the same section of a higher LOD (if available)
