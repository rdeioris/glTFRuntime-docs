# AssetUserData

AssetUserData are additional informations you can attach to an asset (well to all of the ones inheriting from IInterface_AssetUserData)

You can retrieve those informations using the GetAssetUserDataOfClass() function (from both C++ and Blueprints)

# glTFRuntimeAssetUserData

This is the base class to add AssetUserData to glTFRuntime assets.

This base class allows the user to define how to fill the additional informations by accessing the JSON structure of the glTF asset as well as the index of the specific asset being created.

The created asset is always the outer of the glTFRuntimeAssetUserData instance.

To add logic to a glTFRuntimeAssetUserData class you can just override the ReceiveFillAssetUserData (Fill AssetUserData) UFunction. It receives the index of the specific object (mesh, skin, material, texture, ...) as the only argument.

# Example for adding uri to each Texture

* Create a Blueprint Class inheriting from glTFRuntimeAssetUserData
* Override the FillAssetUserData with the following content:

![FillAssetUserData](Docs/Screenshots/AssetUserData.png?raw=true "FillAssetUserData")
