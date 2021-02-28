# Managing VRM assets via the low-level JSON API

A low level api for traversing glTF assets json fields is exposed (both to C++ and blieprints). It is a low level api, so consider building your own abstractions over it.

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
