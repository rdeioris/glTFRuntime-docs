# Loading Skeletal Meshes over an existing Skeleton Asset

In this tutorial we are going to reuse the Mannequin Animation Blueprint with a GLTF SkeletalMesh (A ReadyPlayerMe character)

The goal is to completely substitute the original Unreal model with the GLTF/GLB one without losing animations: this involves assigning a specific Skeleton asset to the ReadyPlayerMe SkeletalMesh

![Result](SkeletonRemapping_Data/Intro.PNG?raw=true "Result")

## Step 0: Loading the ReadyPlayerMe model into the Character's SkeletalMeshComponent

This is probably a very easy step (something you may have done dozens of times while playing with glTFRuntime): you get a reference to the character in your level blueprint and assign a new SkeletalMesh to it.

The SkeletalMesh is a gltf asset loaded recursively (to end with a single SkeletalMesh)

![Step0_BP](SkeletonRemapping_Data/Step0_BP.PNG?raw=true "Step0_BP")

The model will be loaded correctly:

![Step0_SM](SkeletonRemapping_Data/Step0_SM.PNG?raw=true "Step0_SM")

...but in the viewport it is in A-Pose (instead of being animated) and it is facing the wrong direction

![Step0_Viewport](SkeletonRemapping_Data/Step0_Viewport.PNG?raw=true "Step0_Viewport")

## Step 1: Fixing the SkeletalMesh Forward

The wrong facing issue is caused by the Unreal convention of having the Character's forward vector pointing along the Y (instead of the X). This is a bit annoying but lucky enough we can quickly instruct the glTFRuntime loader to automatically assume the Y is the forward vector:

![Step1_BP](SkeletonRemapping_Data/Step1_BP.PNG?raw=true "Step1_BP")

Now the Character should point in the right direction (but still in A-Pose)

![Step1_Viewport](SkeletonRemapping_Data/Step1_Viewport.PNG?raw=true "Step1_Viewport")

## Step 2: Assigning the Mannequin Skeleton

it is important to always keep in mind that each SkeletalMesh in Unreal is technically mapped to two skeleton hierarchies: the SkeletonRef and the Skeleton Asset.

The SkeletonRef defines the Mesh specific bones with their positions and rotations and it is part of the SkeletalMesh Asset (something you can access via C++). The Skeleton Asset instead is used by the animation system to recognize if a SkeletalMesh is "compatible" with a specific animation (animations themselves are assigned to a Skeleton). This means that if two different SkeletalMeshes (read: with two different SkeletonRefs) ae mapped to the same Skeleton Asset they can use the same animations.

Now our Character has its own SkeletonRef (generated by glTFRuntime based on the asset information) and its own Skeleton Asset (generated automatically from the SkeletonRef).

Obviously this means the Mannequin animations cannot be used with our own ReadyPlayerMe Character.

Let's start by assigning it the Mannequin Skeleton Asset:

![Step2_BP](SkeletonRemapping_Data/Step2_BP.PNG?raw=true "Step2_BP")

OOps something bad happened!

![Step2_Viewport](SkeletonRemapping_Data/Step2_Viewport.PNG?raw=true "Step2_Viewport")

(please notice that the head is animated!)

The problem here is that the ReadyPlayerMe model has the root bone in the pelvis, while the Mannequin in the middle of the feet.

Let's fix this by adding a root bone:

![Step2_BP_Fix](SkeletonRemapping_Data/Step2_BP_Fix.PNG?raw=true "Step2_BP_Fix")

Yep!

![Step2_Viewport_Fix](SkeletonRemapping_Data/Step2_Viewport_Fix.PNG?raw=true "Step2_Viewport_Fix")

(please continue noticing the head is animated!)

## Step 3: Fixing the SkeletonRef

Unfortunately we are in a pretty broken situation: the SkeletonRef has a completely different hierarchy in respect to the one of the Mannequin and, more important, from the Skeleton Asset we assigned. But why the head is animated?

Let's look at the Mannequin structure:

![Step3_SM](SkeletonRemapping_Data/Step3_SM.PNG?raw=true "Step3_SM")

it looks pretty different compared to the ReadyPlayerMe, but there is a bone with a matching name: the head!

Yes, the animation curves will be applied to all of those Bones in the Skeleton Asset that exists in the SkeletonRef.

This means that if we could rename the bones of the ReadyPlayerMe Character we should be able to see it animated.

Let's try remapping 'Hips' to 'pelvis':

![Step3_BP](SkeletonRemapping_Data/Step3_BP.PNG?raw=true "Step3_BP")

Nope:

![Step3_Viewport](SkeletonRemapping_Data/Step3_Viewport.PNG?raw=true "Step3_Viewport")

We have the right name for the bone, but the animations assume the base pose of a model (read: the SkeletonRef) has a very specific rotations configuration (while positions can be different).

Lucky enough we can copy rotations from a Skeleton Asset to our SkeletonRef (withe CopyRotationsFrom field):

![Step3_BP_Fix](SkeletonRemapping_Data/Step3_BP_Fix.PNG?raw=true "Step3_BP_Fix")

The pelvis (and the head) are now correct (given that we have copied the rotations from the Mannequin Skeleton Asset), but all of the other bones of the SkeletonRef are broken!

![Step3_Viewport_BrokenRot](SkeletonRemapping_Data/Step3_Viewport_BrokenRot.PNG?raw=true "Step3_Viewport_BrokenRot")

We need to change the behaviour of the bones remapper to just map 'unmapped' bones to the first valid parent (so the spine_01, spine_02 and so on will be mapped to pelvis given that it is the first valid parent).

We have a simple boolean option for adding this behaviour (Assign Unmapped Bones to Parent):

![Step3_BP_Fix2](SkeletonRemapping_Data/Step3_BP_Fix2.PNG?raw=true "Step3_BP_Fix2")

We will end now with our Character having just a root and a pelvis (the only two 'valid' bones in the hierarchy after the remapping):

![Step3_Viewport_FixedRot](SkeletonRemapping_Data/Step3_Viewport_FixedRot.PNG?raw=true "Step3_Viewport_FixedRot")

## Step 4: Remapping bones with JSON

You could obviously start filling the Blueprint Map for the bones remapping manually, but honestly its a very annoying procedure. Instead I will show you a more 'developer-friendly' approach: let's define the mapping in the asset itself!

Well, instead of editing (and possibly breaking) the ReadyPlayerMe asset, i am going to create a new gltf (json) file with just the information we need. We will load it independently (and at runtime). But feel free to embed the 'extras' object in your asset.

```json
{
	"extras":
	{
		"rpm_retargeting":
		{
			"Hips": "pelvis",
			"Spine": "spine_01",
			"Spine1": "spine_02",
			"Spine2": "spine_03",
			"Neck": "neck_01",
			"Head": "head",
			"LeftShoulder": "clavicle_l",
			"RightShoulder": "clavicle_r",
			"LeftArm": "upperarm_l",
			"RightArm": "upperarm_r",
			"LeftForeArm": "lowerarm_l",
			"RightForeArm": "lowerarm_r",
			"LeftHand": "hand_l",
			"RightHand": "hand_r",
			"LeftUpLeg": "thigh_l",
			"RightUpLeg": "thigh_r",
			"LeftLeg": "calf_l",
			"RightLeg": "calf_r",
			"LeftFoot": "foot_l",
			"RightFoot": "foot_r",
			"LeftToe_End": "ball_l",
			"RightToe_End": "ball_r",

			"LeftHandThumb1": "thumb_01_l",
			"RightHandThumb1": "thumb_01_r",
			"LeftHandThumb2": "thumb_02_l",
			"RightHandThumb2": "thumb_02_r",
			"LeftHandThumb3": "thumb_03_l",
			"RightHandThumb3": "thumb_03_r",

			"LeftHandIndex1": "index_01_l",
			"RightHandIndex1": "index_01_r",
			"LeftHandIndex2": "index_02_l",
			"RightHandIndex2": "index_02_r",
			"LeftHandIndex3": "index_03_l",
			"RightHandIndex3": "index_03_r",

			"LeftHandMiddle1": "middle_01_l",
			"RightHandMiddle1": "middle_01_r",
			"LeftHandMiddle2": "middle_02_l",
			"RightHandMiddle2": "middle_02_r",
			"LeftHandMiddle3": "middle_03_l",
			"RightHandMiddle3": "middle_03_r",

			"LeftHandRing1": "ring_01_l",
			"RightHandRing1": "ring_01_r",
			"LeftHandRing2": "ring_02_l",
			"RightHandRing2": "ring_02_r",
			"LeftHandRing3": "ring_03_l",
			"RightHandRing3": "ring_03_r",

			"LeftHandPinky1": "pinky_01_l",
			"RightHandPinky1": "pinky_01_r",
			"LeftHandPinky2": "pinky_02_l",
			"RightHandPinky2": "pinky_02_r",
			"LeftHandPinky3": "pinky_03_l",
			"RightHandPinky3": "pinky_03_r"
		}
	}
}
````

For glTFRuntime this is a valid GLTF file, so we can load it and use the low-level api for accessing the JSON information:

![Step4_BP](SkeletonRemapping_Data/Step4_BP.PNG?raw=true "Step4_BP")

If the mapping is correct you will see your ReadyPlayerMe Character using the Mannequin animations:

![Step4_Viewport](SkeletonRemapping_Data/Step4_Viewport.PNG?raw=true "Step4_Viewport")

## Step 5: More complex remapping: UE5 Manny

The UE5 Manny Character has a more complex (and somewhat incompatible) Skeleton structure. The main issue is the different hierarchy (check the spine bones).

![Manny](SkeletonRemapping_Data/Step5_Manny.PNG?raw=true "Manny")

Given that 'holes in the hierarchy' are not allowed, we need to map the missing bones (like spine_04 and spine_05) to a valid parent bone (like spine_03) and assign them an identity (empty) Transform:

```json
{
	"extras":
	{
		"rpm_retargeting":
		{
			"Hips": "pelvis",
			"Spine": "spine_01",
			"Spine1": "spine_02",
			"Spine2": "spine_03,spine_04,spine_05",
			"Neck": "neck_01,neck_02",
			"Head": "head",
			"LeftShoulder": "clavicle_l",
			"RightShoulder": "clavicle_r",
			"LeftArm": "upperarm_l",
			"RightArm": "upperarm_r",
			"LeftForeArm": "lowerarm_l",
			"RightForeArm": "lowerarm_r",
			"LeftHand": "hand_l",
			"RightHand": "hand_r",
			"LeftUpLeg": "thigh_l",
			"RightUpLeg": "thigh_r",
			"LeftLeg": "calf_l",
			"RightLeg": "calf_r",
			"LeftFoot": "foot_l",
			"RightFoot": "foot_r",
			"LeftToe_End": "ball_l",
			"RightToe_End": "ball_r",

			"LeftHandThumb1": "thumb_01_l",
			"RightHandThumb1": "thumb_01_r",
			"LeftHandThumb2": "thumb_02_l",
			"RightHandThumb2": "thumb_02_r",
			"LeftHandThumb3": "thumb_03_l",
			"RightHandThumb3": "thumb_03_r",

			"LeftHandIndex1": "index_metacarpal_l,index_01_l",
			"RightHandIndex1": "index_metacarpal_r,index_01_r",
			"LeftHandIndex2": "index_02_l",
			"RightHandIndex2": "index_02_r",
			"LeftHandIndex3": "index_03_l",
			"RightHandIndex3": "index_03_r",

			"LeftHandMiddle1": "middle_metacarpal_l,middle_01_l",
			"RightHandMiddle1": "middle_metacarpal_r,middle_01_r",
			"LeftHandMiddle2": "middle_02_l",
			"RightHandMiddle2": "middle_02_r",
			"LeftHandMiddle3": "middle_03_l",
			"RightHandMiddle3": "middle_03_r",

			"LeftHandRing1": "ring_metacarpal_l,ring_01_l",
			"RightHandRing1": "ring_metacarpal_r,ring_01_r",
			"LeftHandRing2": "ring_02_l",
			"RightHandRing2": "ring_02_r",
			"LeftHandRing3": "ring_03_l",
			"RightHandRing3": "ring_03_r",

			"LeftHandPinky1": "pinky_metacarpal_l,pinky_01_l",
			"RightHandPinky1": "pinky_metacarpal_r,pinky_01_r",
			"LeftHandPinky2": "pinky_02_l",
			"RightHandPinky2": "pinky_02_r",
			"LeftHandPinky3": "pinky_03_l",
			"RightHandPinky3": "pinky_03_r"
		},
	}
}
```

As you can see, the value of each bone mapping can contains multiple comma separated targets. This special feature instructs glTFRuntime to add more bones to the same key.

Unfortunately the Manny animations do not match very well with the ReadyPlayerMe Character, so do not expect a great result.

## Final Notes

* Try to abuse the "extras" object in your GLTF/JSON files. There are nodes for extracting numbers, strings, array of strings and booleans (in addition to the string dictionary we have already seen)
* The Mannequin pose has the arms in a slightly different configuration in respect to the ReadPlayerMe Character (about 10 degrees on the Y axis). You can add two 'Transform Bone' nodes for the upperarms in the Animation Blueprint to fix this (consider adding the 10 degrees information in an 'extras' object)
* The height of the ReadyPlayerMe Character is different, so consider using its bounds to fix the capsule height and the SkeletalMeshComponent Z position
* Probably the feet of the ReadyPlayerMe Character will be a bit under the platforms: add a delta to the SkeletalMeshComponent Z value (you could base it on the shoes height)
