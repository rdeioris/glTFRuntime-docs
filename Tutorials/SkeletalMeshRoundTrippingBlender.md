
# Skeletal Mesh Roundtripping between Blender and Unreal

This Tutorial will guide you into exporting a skeletal mesh from Unreal Engine to GLTF, from GLTF to Blender, from Blender to GLTF, and from GLTF back into Unreal Engine.

This allows an art asset pipeline between Blender and Unreal at runtime and without going through the editor.

Everything will be done with Blueprints but you can obviously accomplish the same results with C++ too.

We will not use the provided glTFRuntimeAssetActor but we will load everything manually to have full control (and better understanding) over the workflow.


# Unreal Engine Skeletal Mesh to GLTF

## Step 1: Exporting the Skeletal Mesh

Right click your Skeletal Mesh in the Content Browser and select Asset Actions > Export

![Step1 Exporting](SkeletalMeshRoundTrippingBlender_Data/Step1_Exporting.png?raw=true "Step1 Exporting")

Give it a useful filename and save as one of the GL types (.glb or .gltf)

![Step1 Export Filename](SkeletalMeshRoundTrippingBlender_Data/Step1_ExportFileName.png?raw=true "Step1 Export Filename")

Setup your export settings to your liking, default settings were used in this tutorial.

![Step1 Export Settings](SkeletalMeshRoundTrippingBlender_Data/Step1_ExportOptions.png?raw=true "Step1 Export Settings")

## Step 2: Loading the same GLTF file at runtime

Now we load the GLTF file and provide it the same skeleton the original Skeletal Mesh uses in Unreal (in this case SKEL_base) via the glTFRuntimeSkeletalMeshConfig

We spawn an actor, provide it a skeletal mesh component and assign it the loaded skeletal mesh from the GLTF file we exported and give it an animation to play to test if it works.

![Step2 Load GLTF](SkeletalMeshRoundTrippingBlender_Data/Step2_LoadGLTF.png?raw=true "Step2 Load GLTF")

The skeletal animation will appear to be walking sideways, which isn't what we want.

![Step2 Fail Anim](SkeletalMeshRoundTrippingBlender_Data/Step2_FailAnim.png?raw=true "Step2 Fail Anim")

To fix this we have to set the Transform Base Type to YForward when we load the asset:

![Step2 Fix](SkeletalMeshRoundTrippingBlender_Data/Step2_Fix.png?raw=true "Step2 Fix")

With that, the walk animation appears to be correct and we have successfully roundtripped the skeletal mesh from Unreal Engine to GLTF and back.

![Step2 Fix Result](SkeletalMeshRoundTrippingBlender_Data/Step2_FixResult.png?raw=true "Step2 Fix Result")

# Importing/Exporting GLTF with Blender

Now let's try to bring the GLTF file into Blender, make any desired changes, export it to GLTF, and bring it back into Unreal.

## Step 1: Import the GLTF file

Select File > Import > glTF 2.0 (.glb/.gltf)

![Step1 Blender Import glTF](SkeletalMeshRoundTrippingBlender_Data/Step1_BlenderImport.png?raw=true "Step1 Blender Import glTF")

Make any desired changes to the mesh/materials/animations without changing the skeleton.

## Step 2: Export the GLTF file

Select File > Export > glTF 2.0 (.glb/.gltf) to select your export options. Default settings with +Y Up are what I used.

Before exporting either: 
* Remove everything else in the scene

or

* Ensure only the mesh and skeleton are selected and that the export option Include > Limit to "Selected Objects" is enabled.

![Step2 Blender Export glTF](SkeletalMeshRoundTrippingBlender_Data/Step2_BlenderExport.png?raw=true "Step2 Blender Export glTF")

## Step 3: Loading the GLTF file at runtime

If you exported with Blender 4.2, you should be done, loading the .gltf file the same way we did with the Unreal exported file works right out of the box.


### Blender 3.4 Fix

If you used Blender 3.4 to export (and perhaps earlier and later builds, it's not clear when this was resolved) you will see the mesh is mangled.

![Step3 Blender3.4 Fail](SkeletalMeshRoundTrippingBlender_Data/Step3_Blender34Fail.png?raw=true "Step3 Blender3.4 Fail")

To fix this, set the Overwrite Ref Skeleton to Enabled in the glTFRuntimeSkeletalMeshConfig:

![Step3 Blender3.4 Fix](SkeletalMeshRoundTrippingBlender_Data/Step3_Blender34Fix.png?raw=true "Step3 Blender3.4 Fix")

And we get a successful result:

![Step2 Fix Result](SkeletalMeshRoundTrippingBlender_Data/Step2_FixResult.png?raw=true "Step2 Fix Result")


# The Morph Target problem

As of Unreal Engine 5.4.2 the built in gltf exporter does not support exporting morph targets. With some legwork it's still possible to roundtrip the skeletal mesh with morph targets though, the simplified version is:

* In Unreal, export the skeletal mesh as FBX - this supports morph targets but you lose materials
* In Blender, import the FBX mesh and fix the materials either manually recreating them or importing the glTF and assigning the materials from that
* In Blender, export the mesh  to .glTF with shapekeys enabled
* In Unreal, load the skeletal mesh (as described in the earlier section) and in the glTFRuntimeSkeletalMeshConfig ensure you set the Morph Targets Duplicate strategy is set to Merge. Without doing this it will import the morph targets but setting their values does not  have an effect.