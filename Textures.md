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
* KTX2 (implemented by https://github.com/rdeioris/glTFRuntimeKTX2, BGRA8, DXT5)

If you are loading a GLTF scene (or a mesh), textures will be automatically extracted and applied to the specific object, but you can load them manually
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

Image indices are related to the gltf asset (so you may want to iterate them, the ```int32 GetNumImages() const``` method will help in this).
Blob is meant for assets directly loaded from a file or a url:

![Blob](Docs/Screenshots/Textures000.png?raw=true "Blob")

## MipMaps

Each texture is composed by 1 ore more mipmaps. Mipmaps (or Mips) are reduced (in size) variants of the same image. Having optimized
variants of the same image based on the required (on screen) size can improve GPU size dramatically (at the cost of additional memory usage).

The default behaviour of glTFRuntime is to have a single Mip for each texture, but you can enable automatic Mips generation or (if the format supports it, like KTX2 or DDS) automatic mips extraction from the image asset.

To automatically generate Mips, you need to enable the ```bGeneratesMipMaps``` flag in the Materials Config structure:

![GenerateMips](Docs/Screenshots/Textures001.png?raw=true "GenerateMips")

To automatically load Mips (if the format exposes them), you need to enable the ```bLoadMipMaps``` flag in the Materials Config structure:

![LoadMips](Docs/Screenshots/Textures002.png?raw=true "LoadMips")

## Texture Compression

While image formats often imply some form of compression (like PNG and JPEG), those data are uploaded uncompressed to the GPU.

A 1024x1024 image in BGRA8 format will occupy 4MBs of precious GPU memory. A 4K texture will use 64MBs!

Lucky enough GPUs supports uploading image data in a (lossy) compressed format. The most common (at least on desktop operating systems) is the DXT/STC3 format.

If your image format supports one of those compression system (DXT5 or BC7 generally, both available in KTX2 and DDS formats) that data will be automatically uploaded to the GPU
(dramatically reducing the amount of used memory).

This is the result of loading the official Khronos sample of the "FlightHelmet" with the KTX2 extension (https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0/FlightHelmet/glTF-KTX-BasisU):

![KTX2](Docs/Screenshots/Textures003.png?raw=true "KTX2")

Eventually you may want to automatically compress linear formats (like BGRA8) to compressed ones. You can do it by using the https://github.com/rdeioris/glTFRuntimeSTBImage plugin and enabling the bCompressMips flag in the Images config structure:

![DXT](Docs/Screenshots/Textures004.png?raw=true "DXT")

The result will be a DXT5 compressed texture.

## Textures array

This is (as the name implies) a list of images in the same texture. From a technical point of view you can "sample" them with 3 values instead of the classic 2 (UV).

The third value is the index in the textures array.

They are used for various tricks, but generally (for lower level implementations) they allow to reduce the impact of switching textures in each shader (the amount of usable textures in a shader can be limited).

## Cubemaps

Cubemaps are a special form of textures array: they are formed by 6 square images.

Each one of the images represents a face of a virtual cube. Those specific kind of textures are used for skyboxes and for images based lighting (included reflection captures).

Unreal expects those 6 images with a very special layout:

![CubeMap](Docs/Screenshots/Textures005.png?raw=true "CubeMap")

Cubemap images can be stored as 6 different images, or as a single one including all of them projected in a single spherical image:

![Spherical](Docs/Screenshots/Textures006.png?raw=true "Spherical")

(thanks to Epic Games for the 2 example images)

glTFRuntime supports loading cubemaps from both of the image types.

This is an example of loading an EXR 2k image from https://polyhaven.com/hdris:

![CubeMapFromBlob](Docs/Screenshots/Textures007.png?raw=true "CubeMapFromBlob")

And the result:

![CubeMapTexture](Docs/Screenshots/Textures008.png?raw=true "CubeMapTexture")

## Streaming 

As we have seen GPU memory usage is one of the most critical part when working with textures. Those problems are often amplified when doing runtime loading.

To reduce the amount of used GPU memory (especially for very big levels with lot of assets) Unreal Engine implements a streaming system where the textures Mips are copied to the GPU only when required (read: when the specific Mip level needs to be rendered).

If you have multiple mips for textures, you can enable glTFRuntime texture streaming by setting the 'bStreaming' flag in the ImagesConfig structure. This will keep a copy of all of the Mips in system memory, and the streamer will copy the required ones on the GPU.

If you want to test texture streaming you can use the 'Statistics' tool under the Tools/Audit menu, and remember that while in editor the streamer will always keep 7 mips level (so you need at least 8 levels for seeing it working).

## WIP: Virtual Texturing

Virtual Texturing has multiple meanings in Unreal Engine, here I am referring to "virtual texture streaming": textures are placed on a virtual grid with a configurable cell size. Instead of loading a whole texture, the streaming system can load the "visible" part of it falling into a specific cell. This potentially results in more efficient memory usage (again at a performance cost).

## Notes

GLTF makes a difference between "textures" and "images":

"images" are the blob containing the images data, while "textures" are the combination of "images" and "samplers" (samplers are the filters used when reading textures, like nearest, bilinear... and the wrapping strategy, like repeat or clamp)

The "compression" setting in the Images config structures has currently no usage (but glTFRuntime plugins developers can make use of it)


