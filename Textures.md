# Managing Textures at runtime

glTFRuntime supports loading textures from various sources:

* from "image" objects in a GLTF file (you reference them by the index)
* from "blob" data (taken from files, strings or urls)

Images data can be loaded into the following types of Unreal assets

* UTexture2D (single image)
* UTexture2DArray (multiple images)
* UTextureCube (6 square images)

Images data can be compressed in various formats, each one with a sepcific PixelFormat:

* PNG (natively supported, BGRA8)
* JPEG (natively supported, BGRA8)
* EXR (natively supported in Unreal 5, Float_RGBA)
* TIFF (natively supported, BGRA8)
* TGA (natively supported, BGRA8)
* DDS (implemented by glTFRuntime, DXT1, DXT3, DXT5, R16F, G16R16F, Float_RGBA, BC7, BGRA8)
* WebP (implemented by https://github.com/rdeioris/glTFRuntimeWebP, BGRA8)
