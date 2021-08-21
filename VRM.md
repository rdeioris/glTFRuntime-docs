# Managing VRM assets via the low-level JSON API

A low level api for traversing glTF assets json fields is exposed (both to C++ and blueprints). It is a low level api, so consider building your own abstractions over it.

VRM (https://vrm.dev/en/) is a glTF extension adding informations for humanoid characters targeted at VR environments.

It is the perfect example for discovering how the low level api work.

The VRM extensions are drafted here: https://github.com/vrm-c/vrm-specification/blob/master/specification/0.0/README.md

# extensions.VRM.meta

This is the simplest extension to grasp: https://github.com/vrm-c/vrm-specification/blob/master/specification/0.0/README.md#vrm-extension-model-information-jsonextensionsvrmmeta

It basically adds metadata to the whole asset (like author, version, title, license...)

If we want to read the 'author' metadata we need to access the 'extensions' JSON object, then the 'VRM' object, then the 'meta' object and finally the 'author' field.

In JSONPath syntax would be: `$.extensions.VRM.meta.author`

While via blueprint you can do something like this:

![VRMmeta](Docs/Screenshots/VRMmeta.PNG?raw=true "VRMmeta")

The key here is understanding the glTFRuntimePathitem structure: if the Index field is -1, then we are asking for an object field, it is higher than -1 then the field is is an array and we are getting the 'Index' element.

As there are no array involved here, we do not need indexes.

# extensions.VRM.humanoid

This is a more interesting one: https://github.com/vrm-c/vrm-specification/blob/master/specification/0.0/README.md#vrm-extension-models-bone-mapping-jsonextensionsvrmhumanoid

it allows to define the joint mapped to a specific (known) humanoid bone.

There is an official list of bones: https://github.com/vrm-c/vrm-specification/blob/master/specification/0.0/README.md#defined-bones

So if you are asset as a skeleton with a bone named FooBar in it, this extension will allow you to recogninze FooBar as a 'jaw'

The mapping is managed by the  `$.extensions.VRM.humanoid.humanBones` array, so the first thing we need to do is knowing the size of the array to start indexing its elements:


![VRMhumanoid](Docs/Screenshots/VRMhumanoid.PNG?raw=true "VRMhumanoid")

The GetArrySizeFromPath will return the size of the specified array.

Now we can iterate the list of bones. The Json is something like this:

```json
"humanoid":
  {"humanBones":
    [
      {"bone":"hips","node":3,"useDefaultValues":true},
      {"bone":"leftUpperLeg","node":123,"useDefaultValues":true},
      {"bone":"rightUpperLeg","node":136,"useDefaultValues":true}
    ]
  }
```

So assuming BonesSize is an integer variable holding the array size we can access all of the 'bone' fields with:

![VRMhumanoidBones](Docs/Screenshots/VRMhumanoidBones.PNG?raw=true "VRMhumanoidBones")

Or if you want to get the joint name from the node id (read: the true bone name of the Unreal skeleton):

![VRMhumanoidJoints](Docs/Screenshots/VRMhumanoidJoints.PNG?raw=true "VRMhumanoidJoints")

# C++/Blueprint UFunctions

```cpp
UFUNCTION(BlueprintCallable, BlueprintPure, Category = "glTFRuntime")
FString GetStringFromPath(const TArray<FglTFRuntimePathItem> Path, bool& bFound) const;

UFUNCTION(BlueprintCallable, BlueprintPure, Category = "glTFRuntime")
int64 GetIntegerFromPath(const TArray<FglTFRuntimePathItem> Path, bool& bFound) const;

UFUNCTION(BlueprintCallable, BlueprintPure, Category = "glTFRuntime")
float GetFloatFromPath(const TArray<FglTFRuntimePathItem> Path, bool& bFound) const;

UFUNCTION(BlueprintCallable, BlueprintPure, Category = "glTFRuntime")
bool GetBooleanFromPath(const TArray<FglTFRuntimePathItem> Path, bool& bFound) const;

UFUNCTION(BlueprintCallable, BlueprintPure, Category = "glTFRuntime")
int32 GetArraySizeFromPath(const TArray<FglTFRuntimePathItem> Path, bool& bFound) const;
```
