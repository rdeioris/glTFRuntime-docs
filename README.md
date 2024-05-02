# glTFRuntime
Unreal Engine Plugin for loading glTF assets at runtime

![Megagrant](Epic_MegaGrants_Recipient_logo_horizontal_black.png?raw=true#gh-light-mode-only "Megagrant")
![Megagrant](Epic_MegaGrants_Recipient_logo_horizontal_white.png?raw=true#gh-dark-mode-only "Megagrant")

Join the Discord Channel: https://discord.gg/DzS7MHy

Check the Features Showcase: https://www.youtube.com/watch?v=6058JA8wX8I

Buy it from the Marketplace to support the development: https://www.unrealengine.com/marketplace/en-US/product/gltfruntime

Official sources available at https://github.com/rdeioris/glTFRuntime

...and please read this before packaging your project! https://github.com/rdeioris/gltfruntime-docs#notes-when-packaging-a-game

![glTFRuntime](Docs/Screenshots/glTFRuntime512.jpg?raw=true "glTFRuntime")

# Features

* Allows to load Static Meshes, Skeletal Meshes, Animations, Cameras, Lights, Hierarchies, Materials and Textures from glTF 2.0 Embedded (.gltf) or Binary (.glb) files
* Extensive API accessible from both C++ and Blueprints
* Pluggable support for more file formats (OBJ, STL, FBX and MagicaVoxel extensions already available at https://github.com/rdeioris/glTFRuntime/blob/master/README.md#plugins)
* Assets can be loaded on the fly both in PIE and Packaged Games
* Assets can be loaded from the filesystem, http servers, the clipboard, commandlines output or raw json strings.
* Assets can be compressed with gzip or lz4 (will be decompressed on the fly)
* Assets can be archived in zip files (they will be extracted and decompressed on the fly)
* Supports generating ad-hoc Skeletons or reusing already existing ones (a gltf Exporter for Skeletons is included too)
* Non skeletons/skins-related animations are exposed as Curves
* Full support for PBR Materials.
* Support for the Specular/Glossiness extension (KHR_materials_pbrSpecularGlossiness)
* Support for glTF 2.0 Sparse Accessors.
* Support for multiple texture coordinates/channels/uvs.
* Allows to define Static Meshes collisions (Spheres, Boxes, Complex Meshes) at runtime as well as PhysicsAssets for SkeletalMeshes
* StaticMeshes can be imported as SkeletalMeshes (with a single root bone) and the opposite
* Support for MorphTargets
* Support for merging multiple meshes on the same skeleton
* Support for Vertex Colors
* Support for VRM extensions
* Preliminary support for Audio Emitters (MSFT_audio_emitter, wav files supported, check https://github.com/rdeioris/glTFRuntimeAudio for Opus, Vorbis and MP3)
* Support for KHR_mesh_quantization
* Async StaticMesh and SkeletalMesh loading
* Support for KHR_materials_variants, KHR_materials_clearcoat, KHR_materials_unlit, KHR_materials_ior, KHR_materials_emissive_strength
* Support for EXT_mesh_gpu_instancing
* Support for MSFT_lod (https://github.com/rdeioris/glTFRuntime-docs/blob/master/LOD.md)
* Support for EXT_meshopt_compression and KHR_mesh_quantization
* Support for DDS textures
* Support for KHR_lights_punctual
* Support for Texture Streaming

Platforms supported are: Win64, Linux, Mac, Android, iOS, Linux Arm64 and Hololens2

Both UnrealEngine 4 and 5 are supported.

# Quickstart

Consider buying the plugin from the Epic Marketplace (https://www.unrealengine.com/marketplace/en-US/product/gltfruntime), you will get automatic installation and you will support the project.

If you want to build from sources, just start with a C++ project, and clone the master branch into the Plugins/ directory of your project, regenerate the solution files and rebuild the project.

Once the plugin is enabled you will get a set of C++/Blueprint functions for loading assets at runtime from various sources:

![image](https://github.com/rdeioris/glTFRuntime-docs/assets/2234592/f29a45a3-f731-47fb-aabd-2e765a20767a)

Let's start with remote asset loading (we will use the official glTF 2.0 samples), open your level blueprint and on the BeginPlay Event, trigger
the runtime asset loading:

![UrlDuck](Docs/Screenshots/UrlDuck.PNG?raw=true "UrlDuck")

A bunch of notes:

* We are loading the asset from the https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/Duck/glTF-Embedded/Duck.gltf url
* The glTFRuntimeAssetActor is a ready-to-use Actor class included in the plugin. It is perfect for testing, but you are encouraged to implement more advanced structures.
* The glTFLoadAssetFromUrl function is an asynchronous one, the related event will be triggered when the asset is loaded.
* The glTFRuntimeAssetActor requires a glTFRuntimeAsset to correctly spawn.

If all goes well you should see the Collada Duck:

<img src="Docs/Screenshots/ColladaDuck.PNG?raw=true" alt="ColladaDuck" width="50%" />

# Loading Scenes

Time to run your favourite DCC to create a glTF file.

Here i am using Blender 2.83, and i will create a simple scene with Suzanne and a Hat (well a cone) on the center:

(you can download the asset from here if you do not want to build it by yourself:

https://raw.githubusercontent.com/rdeioris/glTFRuntime/master/Docs/Assets/SuzanneWithHat.gltf)

<img src="Docs/Screenshots/SuzanneWithHat.PNG?raw=true" alt="SuzanneWithHat" width="50%" />

Now select both Suzanne and the Cone/Hat and select the File/Export/glTF2.0 menu option

In the export dialog ensure to select the gltf 2.0 Embedded format and to include the selected objects:

<img src="Docs/Screenshots/BlenderExport.PNG?raw=true" alt="BlenderExport" width="50%"/>

(I have saved it as D:/SuzanneWithHat.gltf)

Now back to the Level Blueprint:

![LoadSuzanneWithHat](Docs/Screenshots/LoadSuzanneWithHat.PNG?raw=true "LoadSuzanneWithHat")

Notes:

* This time, as we are loading from the filesystem, we have a synchronous function.
* Note the transform with the increased Z value (if you get suzanne below the floor, you now know why it is happening ;)
* Ensure the path is valid (use the absolute one for being safe, use the 'Path Relative to Content' flag if you need to force users to place assets into the Content/ directory)

The result:

<img src="Docs/Screenshots/RunSuzanneWithHat.PNG?raw=true" alt="RunSuzanneWithHat" width="50%"/>

By running the project in PIE, you will be able to see the hierarchy generated by the glTFRuntimeAssetActor:

![DetailsTree](Docs/Screenshots/DetailsTree.PNG?raw=true "DetailsTree")

This is managed automatically by the glTFRuntimeAssetActor implementation, but you are free to manipulate the glTF hierarchy as you need (included completely ignoring it)

# Loading Static Meshes

Til now, we have used the glTFRuntimeAssetActor commodity class for loading and showing our glTF Assets.

To get the best from the Plugin, you have access to lower level functions allowing you to load specific assets from the glTF scene.

In the following example we will load a single StaticMesh (the Suzanne part of the previous asset, without the Hat) and we will assign it to an already existent
StaticMesh actor in the scene (i have dragged a classic Cube here):

![LoadStaticMesh](Docs/Screenshots/LoadStaticMesh.PNG?raw=true "LoadStaticMesh")

The LoadStaticMesh function expects the index of a mesh into the glTF file. We are lucky as we know the index 0 is Suzanne, but you can load StaticMeshes by name too (the same name you set in Blender):

![LoadStaticMeshByName](Docs/Screenshots/LoadStaticMeshByName.PNG?raw=true "LoadStaticMeshByName")

Notes:

* As we are reusing an already configured StaticMesh Actor, Suzanne will inherit the previously set material slot
* If you change Suzanne with 'Cone', you will get the Cone mesh (obviously) but without the transform applied (read: no rotation, no scale, no translation), transforms are set into the glTF Node (more on nodes traversing later)

<img src="Docs/Screenshots/SuzanneOnly.PNG?raw=true" alt="SuzanneOnly" width="30%"/> <img src="Docs/Screenshots/ConeOnly.PNG?raw=true" alt="ConeOnly" width="30%"/>

# Materials

Materials, included PBR values and Textures as well as Slots, are automatically managed. All of the materials dynamically inherit from a so called 'UberMaterial'
included in the plugin (albeit you are free to use another one by simply setting options, more on this below)

In this example we will load the 'Damaged Helmet' asset available here:
https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/DamagedHelmet/glTF-Embedded/DamagedHelmet.gltf

<img src="Docs/Screenshots/DamagedHelmet.PNG?raw=true" alt="ConeOnly" width="50%"/>

By using the glTFRuntimeAssetActor we can load the whole scene and check the final result:

![PBRMaterial](Docs/Screenshots/PBRMaterial.PNG?raw=true "PBRMaterial")

As you can see, no special options are required to build a standard PBR Material.

Let's change the Level Blueprint to use another approach:

This time we dynamically create a StaticMeshComponent and we assign the StaticMesh to it (note again the wrong transform):

![PBRMaterial2](Docs/Screenshots/PBRMaterial2.PNG?raw=true "PBRMaterial2")

What if we want to change the UberMaterial ? glTFRuntime expects 6 different Material Types: Opaque, TwoSided, Translucent, TwoSidedTranslucent, Masked and TwoSidedMasked

If you want to completely change the rendering mode of glTF assets you need to define all of them.

But le'ts focus on the 'Opaque' one (the one used by DamagedHelmet). Create a new 'dummy' Material (no nodes in it) and use it as an override for the 'Opaque' UberMaterial:

![UberMaterialOverride](Docs/Screenshots/UberMaterialOverride.PNG?raw=true "UberMaterialOverride")

The result will be something like this:

<img src="Docs/Screenshots/UberMaterialOverrideDamagedHelmet.PNG?raw=true" alt="UberMaterialOverrideDamagedHelmet" width="50%"/>

Obviously sooner or later you will want to get the material PBR parameters (included textures) from the glTF asset, in such a case you need to create a bunch of parameters
in your material asset:

* baseColorFactor (vector3)
* roughnessFactor (scalar)
* metallicFactor (scalar)
* emissiveFactor (vector3)
* alphaCutoff (scalar)
* baseColorTexture (texture2d)
* normalTexture (texture2d)
* metallicRoughnessTexture (texture2d)
* emissiveTexture (texture2d)

Check the M_glTFRuntimeBase material into the plugin Content directory for more infos.

Eventually you can completely bypass the UberMaterial logic and force a common base material by setting the "ForceMaterial" field in the MaterialsConfig structure.

More info about textures: https://github.com/rdeioris/glTFRuntime-docs/blob/master/Textures.md

# Collisions

By default, generated StaticMeshes have no collisions. You can assign collision boxes and spheres using the configuration structure (the one you already used for materials):

![Collisions](Docs/Screenshots/Collisions.PNG?raw=true "Collisions")

The BuildSimpleCollision flag, generates an automatic collision based on the mesh bounding box.

You can even set a complex collision by changing the collision complexity field, in such a case ensure to enable the AllowCPUAccess flag and an Outer set (generally the StaticMeshComponent will be enough), otherwise the physics engine will not be able to generate the related shape.

If you prefer convex collisions check https://github.com/rdeioris/glTFRuntimeConvexCollisions

# Loading Skeletal Meshes

glTF Meshes can be combined with a so called 'Skin' (the equivalent of Unreal Engine Skeleton).
glTFRuntime supports unlimited influences per vertex (but if you need more than 12 influences you need to enable the feature in the Unreal Project Settings).

Download the CesiumMan asset: https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/CesiumMan/glTF-Embedded/CesiumMan.gltf

The glTFRuntimeAssetActor will automatically manage Skeletal Meshes, but as with StaticMeshes you can manually load them:

![SkeletalMesh](Docs/Screenshots/SkeletalMesh.PNG?raw=true "SkeletalMesh")

Note how you need to specify the skin index too, and check the component transform rotation (often the skeleton has a strange orientation generated by the DCC software fixed by the glTF node transform)

<img src="Docs/Screenshots/CesiumMan.PNG?raw=true" alt="CesiumMan" width="50%"/>

If you plan to share the same skeleton between multiple assets, consider giving the skeleton asset to your users as a gltf file. glTFRuntime includes an exporter for skeleton assets that will generate a structure that you can use in your favourite DCC software (like blender). Just right click the skeleton asset, choose 'Asset Actions', then 'Export' and you should see glTF as one of the export options.

You can see a skeleton exported from Unreal Engine as gltf below (yes, it is the standard UE4 Mannequin imported into Blender):

<img src="Docs/Screenshots/Mannequin.PNG?raw=true" alt="Mannequin" width="50%"/>

By default glTFRuntime will create a new Skeleton for each SkeletalMesh, you can force a specific Skeleton (useful for sharing animation) by setting the specific options:

![SkeletonOverride](Docs/Screenshots/SkeletonOverride.PNG?raw=true "SkeletonOverride")

The 'Skeleton' field will assign the specific Skeleton to the SkeletalMesh, while the 'OverwriteRefSkeleton' flag, will copy the bone poses to the SkeletalMesh (this will avoid annoying retargeting issues, especially if the bones rotations have been changed by your DCC software). 

# Skeletal Animations

You can extract Skeletal Animations using the LoadSkeletalAnimation, LoadSkeletalAnimationByName and LoadNodeSkeletalAnimation functions:

![SkeletalAnimations](Docs/Screenshots/SkeletalAnimations.PNG?raw=true "SkeletalAnimations")

All of them require a SkeletalMesh to extract the Skeleton (this is technically not required as you could specify the Skeleton asset directly, but this will allow to set a preview asset in the editor, future glTFRuntime releases will allow to specify only a Skeleton too).

Note that as long as different SkeletalMeshes use the same Skeleton, you can share AnimationBlueprint too.

MorphTarget curves are loaded too.

# Nodes Animations

The glTF format, supports generic animation of nodes (read: changing their transforms over time).

Albeit this is not a form of animation supported out of the box in Unreal Engine, glTFRuntime can export them as simple Curve Assets.

Get the https://github.com/rdeioris/glTFRuntime-docs/blob/master/Docs/Assets/SuzanneWithHatAnimated.gltf asset and load it using the classic glTFRuntimeAssetActor. You will see the Hat of Suzanne moving vertically.

This is accomplished by generating a curve from the asset and applying it at every tick.

You can directly get a curve for a node animation by using the LoadNodeAnimationCurve function:

![LoadNodeAnimationCurve](Docs/Screenshots/LoadNodeAnimationCurve.PNG?raw=true "LoadNodeAnimationCurve")

Check the code in the Tick method of glTFRuntimeAssetActor for an example:

https://github.com/rdeioris/glTFRuntime/blob/master/Source/glTFRuntime/Private/glTFRuntimeAssetActor.cpp

Note: rotations are generally managed as Euler rotations in Unreal Curves, so beware of the Gimbal Lock! To reduce the impact, Quaternions are exposed too.

# glTF Hierarchy

A glTF Asset is a collection of nodes grouped into scenes.

glTFRuntime exposes an api to navigate the nodes tree.

The first thing we can do is getting the list of all of the nodes in the asset. This is useful to understand how a node is made:

![Nodes](Docs/Screenshots/Nodes.PNG?raw=true "Nodes")

As you can see, each node has an index, a name, a transform, a parent index, a list of children indices as well as an optional mesh and skin. Using this data you can rebuild scenes.

Next step, getting the asset scenes:

![Scenes](Docs/Screenshots/Scenes.PNG?raw=true "Scenes")

Each scene as am index, a name, and the list of root nodes. Let's traverse them:

![SceneNodes](Docs/Screenshots/SceneNodes.PNG?raw=true "SceneNodes")

And their children:

![NodeChildren](Docs/Screenshots/NodeChildren.PNG?raw=true "NodeChildren")

Obviously all of this stuff should be made recursive, check the glTFRuntimeAssetActor implementation for some idea:

https://github.com/rdeioris/glTFRuntime/blob/master/Source/glTFRuntime/Private/glTFRuntimeAssetActor.cpp

# Loading Zip archives

If you specify a zip archive as the asset to load, glTFRuntime will use it as the source for all of the file/uri references in the json tree.

The default behaviour is to search for the first file with .glb, .gltf, .json or .js extension in the archive, but you can force the file to use as the entry point
by defining the ArchiveEntryPoint string in the LoaderConfig.

# glTF JSON low-level api (A.K.A. managing VRM assets)

Check https://github.com/rdeioris/glTFRuntime-docs/blob/master/VRM.md for an example

# Errors Management

The loader functions (glTFLoadAssetFromFilename, glTFLoadAssetFromString, glTFLoadAssetFromUrl, ...) simply returns Null in case the file is not available
or the asset structure is not valid. No more checks are made by the three functions. All of the other checks are made by the glTFRuntimeAsset returned by them.

The glTFRuntimeAsset instances triggers an event whenever an error is found during parsing or assets generation. You can subscribe to this event easily:

![OnError](Docs/Screenshots/OnError.PNG?raw=true "OnError")

Remember, when allowing users to mod your game, spit out any error generated by glTFRuntime. It will help them in fixing their assets.

# Integration with LuaMachine

If you need modding for your projects, consider combining glTFRuntime with the LuaMachine Plugin:

https://github.com/rdeioris/LuaMachine/

You will be able to govern asset loading from lua scripting

# Notes when Packaging a Game

All of the glTFRuntime features are available both in editor and in packaged builds.

Just remember a bunch of notes:

* Users can have the most bizarre filesystem layout, avoids using absolute paths, the Content directory is a good candidate for dynamically loading assets
* Always expose errors feedback
* Always check for invalid return value of the three main functions (glTFLoadAssetFromFilename, glTFLoadAssetFromString, glTFLoadAssetFromUrl). NULL check in C++ will be good enough as well as the IsValid node on Blueprints.
* If you get an error about materials, it could be that your packaging procedure is not including the plugin's Content directory. Go to Project Settings/Packaging, and expand the main section for accessing advanced options. Now just add the glTFRuntime Content directory to 'Additional Asset Directories to Cook' ('to Package' in UE4). You could be forced to do this even if you are using your customized materials. (note: if you get the same problem when hitting 'launch' in the editor, just run the level one time in PIE mode to force loading of the material asset)
* If you do not see the /glTFRuntime folder into the content browser (or packaging settings), ensure to enable (in 'view options') both Engine and Plugins content.

# Slow HTTP downloads

If your HTTP downloads are slow, ensure to add those line to DefaultEngine.ini

```
[HTTP]
HttpThreadActiveFrameTimeInSeconds=0
```

The default behaviour is to 'tick' the HTTP client thread 200 times per second (that could result in very bad download performance)

# TODO/WIP

* Import Scenes as Sequencer Assets
* More mesh-merge features
* Automatic support for all of the Microsoft glTF-Toolkit extensions
* Propose a GLTF extension for Signed Distance Fields (this will allow to support Lumen, and maybe Nanite)
* Virtual Texture Streaming (still investigating if it is worthy)
* glTF low-level api for extending and exporting assets
* Hooks for JSON extras objects
* Improve point clouds support
* Investigate URL chain load support (for loading assets with multiple parts stored remotely)
* Support for glXF format

# Commercial Support

Commercial support is offered by Blitter (Rome, Italy).
Just enter the discord server and direct message the admin, or drop a mail to info at blitter dot studio

# Credits

The glTF Assets shown are the official glTF Sample Models available from this repository

https://github.com/KhronosGroup/glTF-Sample-Models

DamagedHelmet:  theblueturtle_ Creative Commons Attribution-NonCommercial license

BoxAnimated, CesiumMilkTruck, CesiumMan: Donated by Cesium, Creative Commons Attribution 4.0 International License

AlphaBlendModeTest: Copyright 2018 Analytical Graphics, Inc. CC-BY 4.0

BrainStem: Created by Keith Hunter and owned by Smith Micro Software, Inc, Poser Pro EULA

Collada Duck: SCEA Shared Source License, Version 1.0

# Thanks

Silvia Sicks (https://silviasicks.wordpress.com/) for the glTFRuntime Logo.
