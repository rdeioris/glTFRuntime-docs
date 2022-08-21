
# Retarging Mixamo Animations for ReadyPlayerMe Avatars

![MixamoResult](RetargetingRPMAndMixamo_Data/MixamoResult.PNG?raw=true "MixamoResult")

This Tutorial will guide you into loading a ReadyPlayerMe character at runtime and applying a Mixamo Animation over it (included root motion, if required).

The Animation is converted to a glb file using Blender, so you can load animations at runtime too.

Everything will be done with Blueprints but you can obviously accomplish the same results with C++ too.

We will not use the provided glTFRuntimeAssetActor but we will load everything manually to have full control (and better understanding) over the workflow.

Note: those steps are pretty advanced and generally you do not need to follow them if the animations are fully GLTF compliant and compatible with your mesh skeleton.

Mixamo Note: based on the selected character you may end with a different scheletal structure. Using the "David" character is highly suggested (i always had good results with it)

## Step 0: Loading the RPM Avatar

This is simple, we are going to use the ```LoadSkeletalMeshRecursiveAsync``` to compact the whole RPM Avatar in a single SkeletalMesh (instead of a hierarchy of multiple SkeletalMeshes) and we assign it to an empty Character placed in the level:

![Step0](RetargetingRPMAndMixamo_Data/Step0.PNG?raw=true "Step0")

The result is good but no "cool":

* The root bone is mapped to "Hips" instead of the middle of the feet

![Step0_Skeleton](RetargetingRPMAndMixamo_Data/Step0_Skeleton.PNG?raw=true "Step0_Skeleton")

* The pivot of the SkeletalMesh does not map to the base of the Character's CapsuleCollision (it is flying!)

![Step0_Viewport](RetargetingRPMAndMixamo_Data/Step0_Viewport.PNG?raw=true "Step0_Viewport")


## Step 1: Adding a true Root Bone

Let's configure the glTFRuntime Skeleton loader to automatically add a root bone:

![Step1](RetargetingRPMAndMixamo_Data/Step1.PNG?raw=true "Step1")

Now the result is way better:

![Step1_Skeleton](RetargetingRPMAndMixamo_Data/Step1_Skeleton.PNG?raw=true "Step1_Skeleton")

The root bone is "valid" (no more red bone in the Skeleton Editor)

## Step 2: Fixing Character position and CapsuleCollision height using Mesh Bounds

We are going to fix the "flying avatar issue" by using the model bounds.

Be prepared for a bit of spaghetti: 

![Step2](RetargetingRPMAndMixamo_Data/Step2.PNG?raw=true "Step2")

Thanks to the bounds height (the Z value, rememebr to break/split the Extent bounds pin of the GetBounds node) we can set the capsule 'Half Height' and move the SkeletalMeshComponent to the right vertical offset

![Step2_Viewport](RetargetingRPMAndMixamo_Data/Step2_Viewport.PNG?raw=true "Step2_Viewport")

Ok, we are good. Time for Mixamo!

## Step 3: Downloading the Mixamo animation and Converting it to GLTF

Go to Mixamo and download your favourite animation. We are going to use the "manga-like" Run:

![Step3_Mixamo](RetargetingRPMAndMixamo_Data/Step3_Mixamo.PNG?raw=true "Step3_Mixamo")

Download it as FBX and ensure to not export the skin (we are just interested in skeleton structure and animation curves)

![Step3_MixamoDownload](RetargetingRPMAndMixamo_Data/Step3_MixamoDownload.PNG?raw=true "Step3_MixamoDownload")

Now import the FBX in blender (ensure to set the "Automatic Bone Orientation" flag):

![Step3_BlenderImport](RetargetingRPMAndMixamo_Data/Step3_BlenderImport.PNG?raw=true "Step3_BlenderImport")

And now slide the animation timeline to check that everythng is fine (it is obviously hard to understand what is going on given that we have only bones as visual reference).

![Step3_Blender](RetargetingRPMAndMixamo_Data/Step3_Blender.PNG?raw=true "Step3_Blender")

If your animation has root motion, ensure that the skeleton moves too while sliding the timeline.

Now export the object as gltf/glb from blender (you can use the default export settings) and prepare for the next step.

## Step 4: Loading the Animation

Add a new pin to the Sequence node, load the GLTF file and extract the animation from it:

![Step4](RetargetingRPMAndMixamo_Data/Step4.PNG?raw=true "Step4")

Oops! something went wrong:

![Step4_Error](RetargetingRPMAndMixamo_Data/Step4_Error.PNG?raw=true "Step4_Error")

The bones names of the Mixamo asset do not match the bones names of the RPM Avatar :(

## Step 5: Bones Names Remapping

This is a pretty advanced procedure you will generally do in C++. Lucky enough the difference is just the '''mixamorig:'' (or '''mixamorig10:''' if you are using David) prefix in each bone, so our remapping function will be super simple (and we can do it with a Blueprint):

![Step5](RetargetingRPMAndMixamo_Data/Step5.PNG?raw=true "Step5")

The Curve Remapper pin allows you to define a delegate/event/filter to rename an animation curve (that is generally named as the bone to which transformations are applied). Ensure to use the "Create Event" node when connecting the red (event) pin (we cannot use the "Add Custom Event" node as our function needs to return a value).

In the "Create Event" node, select the "Create a Matching Function" option to create and access you "remapper" function

![Step5_RemapperBase](RetargetingRPMAndMixamo_Data/Step5_RemapperBase.PNG?raw=true "Step5_RemapperBase")

Let's modify it to remove the '''mixamorig:''' prefix:

![Step5_Remapper](RetargetingRPMAndMixamo_Data/Step5_Remapper.PNG?raw=true "Step5_Remapper")

The errors in the Output log should now disappear, so we can play our animation by setting the AnimationData structure of the SkeletalMeshComponent and its AnimationMode:

![Step5_Play](RetargetingRPMAndMixamo_Data/Step5_Play.PNG?raw=true "Step5_Play")

If we hit play we will see a pretty messy situation with our Avatar being dismembered (or something similar...)

![Step5_Mess](RetargetingRPMAndMixamo_Data/Step5_Mess.PNG?raw=true "Step5_Mess")

## Step 6: Retargeting

Retargeting is the process of converting a bone transformation based on a pose, to another one based on a different pose.

The problem we are facing is that the Mixamo animation curves have been built for a specific pose (different from the RPM Avatar).

Lucky enough we can easily retarget animations in glTFRuntime, but before doing this we need to address the scaling of the animation (unfortunately when exporting from an FBX, blender will scale everything by 100, so we set the SceneScale value to '1')

![Step6_Scale](RetargetingRPMAndMixamo_Data/Step6_Scale.PNG?raw=true "Step6_Scale")

Let's try again:

![Step6_Viewport](RetargetingRPMAndMixamo_Data/Step6_Viewport.PNG?raw=true "Step6_Viewport")

Ok, visible, but definitely wrong, let's retarget the animation!

By connecting the RetargetTo pin to the Skeleton object of the SkeletalMesh (the RPM one) the animation curves will be converted for the new skeleton:

![Step6](RetargetingRPMAndMixamo_Data/Step6.PNG?raw=true "Step6")

 we can now hit play again:

![Step6_ViewportFixed](RetargetingRPMAndMixamo_Data/Step6_ViewportFixed.PNG?raw=true "Step6_ViewportFixed")

## Step 7: Fixing the arms

There is still a problem with the arms: they cross in a pretty weird way.

This is caused by the mixamo animation being created for a model in T-Pose, while the RPM Avatar is in A-Pose.

The fix is again pretty easy, we can add a "transformation" to every relevant curve in our animation to address the difference in default poses:

![Step7](RetargetingRPMAndMixamo_Data/Step7.PNG?raw=true "Step7")

Hit play again:

![Step7_Viewport](RetargetingRPMAndMixamo_Data/Step7_Viewport.PNG?raw=true "Step7_Viewport")

Great, the arms are fixed and theoretically if you animation has no root motion, you could end your worflow here.

## (Optional) Step 8: Root Motion

Currently the root motion in our animation is still mapped to the Hips curve, let's move it to the "root" bone by editing our curve remapper function (the one we previously created for removing the "mixamorig:" prefix):

![Step8](RetargetingRPMAndMixamo_Data/Step8.PNG?raw=true "Step8")

Whenever a "translation" curve is named "Hips", we move it to the "root" one.

We can finally enable "RootMotion" and "Anim First Frame" LockMode in our animation

![Step8_RootMotion](RetargetingRPMAndMixamo_Data/Step8_RootMotion.PNG?raw=true "Step8_RootMotion")

and we can turn root motion "on" in the AnimInstance:

![Step8_AnimInstance](RetargetingRPMAndMixamo_Data/Step8_AnimInstance.PNG?raw=true "Step8_AnimInstance")

## Final Notes

![Final](RetargetingRPMAndMixamo_Data/Final.PNG?raw=true "Final")

* As already said this is an advanced approach mainly useful for showing advanced glTFRuntime features
* You can end your workflow at step 7 if you do not need root motion support
