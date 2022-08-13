
# Retarging Mixamo Animations for ReadyPlayerMe Avatars

![MixamoResult](RetargetingRPMAndMixamo_Data/MixamoResult.PNG?raw=true "MixamoResult")

This Tutorial will guide you into loading a ReadyPlayerMe character at runtime and applying a Mixamo Animation over it (included root motion).

Everything will be done with Blueprints but you can obviously accomplish the same result with C++ too.

We will not use the provided glTFRuntimeAssetActor but we will load everything manually to have full control (and better understanding) over the workflow.

## Step 0: Loading the RPM Avatar

This is simple, we are going to use the ```LoadSkeletalMeshRecursiveAsync``` to compact the whole RPM Avatar in a single SkeletalMesh (instead of a hierarchy of multiple SkeletalMeshes) and we assign it to an empty Character placed in the level:

![Step0](RetargetingRPMAndMixamo_Data/Step0.PNG?raw=true "Step0")

The result is good but no "cool":

* The root bone is mapped to "Hips" instead of the middle of the feet

![Step0_Skeleton](RetargetingRPMAndMixamo_Data/Step0_Skeleton.PNG?raw=true "Step0_Skeleton")

* The pivot of the SkeletalMesh does not map to the base of the Character's CapsuleCollision

![Step0_Viewport](RetargetingRPMAndMixamo_Data/Step0_Viewport.PNG?raw=true "Step0_Viewport")





