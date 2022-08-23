# Loading Skeletal Meshes over an existing Skeleton Asset

In this tutorial we are going to reuse the Mannequin Animation Blueprint with a GLTF SkeletalMesh (A ReadyPlayerMe character)

The goal is to completely substitute the original Unreal model with the glb one without losing animations: this involves assigning a specific Skeleton asset to the ReadyPlayerMe SkeletalMesh

![Result](SkeletonRemapping_Data/Intro.PNG?raw=true "Result")

## Step 0: Loading the ReadyPlayerMe model into the Character's SkeletalMeshComponent

## Step 1:

## Step 2:

## Step 3:

## Step 4: Remapping bones with JSON
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
