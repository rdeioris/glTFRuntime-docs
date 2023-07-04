# Managing Textures at runtime

glTFRuntime supports loading textures from various sources:

* from "image" objects in a GLTF file (you reference them by the index)
* from "blob" data (taken from files, strings or urls)

Images data can be loaded into the following types of Unreal assets

* UTexture2D (single image)
* UTexture2DArray (multiple images)
* UTextureCube (6 square images)

Images data can be compressed in various formats, each one with a specific PixelFormat:

* PNG (natively supported, BGRA8)
* JPEG (natively supported, BGRA8)
* EXR (natively supported in Unreal 5, Float_RGBA)
* TIFF (natively supported, BGRA8)
* TGA (natively supported, BGRA8)
* DDS (implemented by glTFRuntime, DXT1, DXT3, DXT5, R16F, G16R16F, Float_RGBA, BC7, BGRA8)
* WebP (implemented by https://github.com/rdeioris/glTFRuntimeWebP, BGRA8)
* HDR (implemented by https://github.com/rdeioris/glTFRuntimeSTBImage, Float_RGBA)
* KTX2 (immplemented by https://github.com/rdeioris/glTFRuntimeKTX2, BGRA8, DXT5)

If you are loading a GLTF scene (or a mesh) textures will be automatically extracted and applied to the specific object, but you can load them manually
using one of the provided UFUNCTIONs of the UglTFRuntimeAsset class:

```cpp
UTexture2D* LoadImage(const int32 ImageIndex, const FglTFRuntimeImagesConfig& ImagesConfig);
UTextureCube* LoadCubeMap(const int32 ImageIndexXP, const int32 ImageIndexXN, const int32 ImageIndexYP, const int32 ImageIndexYN, const int32 ImageIndexZP, const int32 ImageIndexZN, const bool bAutoRotate, const FglTFRuntimeImagesConfig& ImagesConfig);
UTexture2DArray* LoadImageArray(const TArray<int32>& ImageIndices, const FglTFRuntimeImagesConfig& ImagesConfig);
UTexture2D* LoadImageFromBlob(const FglTFRuntimeImagesConfig& ImagesConfig);
UTexture2D* LoadMipsFromBlob(const FglTFRuntimeImagesConfig& ImagesConfig);
UTextureCube* LoadCubeMapFromBlob(const bool bSpherical, const bool bAutoRotate, const FglTFRuntimeImagesConfig& ImagesConfig);
UTexture2DArray* LoadImageArrayFromBlob(const FglTFRuntimeImagesConfig& ImagesConfig);
```

Image indices are related to the gltf asset (so you may want to iterate them, the ```int32 GetNumImages() const`` method will help in this).
Blob is meant for assets directly loaded from a file or a url:


