# glTF Universal Textures Developer Guide

<!-- Software packages overview TBD -->

## KTX Container Format

<!-- libKTX vs lightweight parsers TBD -->

## ETC1S / BasisLZ Codec

### Overview

ETC1S / BasisLZ is a hybrid compression scheme that features rearranged ETC1S texture block data with a custom LZ-style lossless compression. It achieves very high storage / transmission efficiency by prioritizing luma information. Thus, this codec is more suited for color textures (albedo, base color, etc) rather than arbitrary non-color data like normal maps.

### Runtime usage

Using ETC1S / BasisLZ data comprises three steps:

1. Initializing the transcoder with data common to all texture slices (i.e. faces, array elements, mip levels, etc).
   > **Note**: In KTX v2 container, such data is stored in `supercompressionGlobalData` block. See [BasisLZ Global Data](https://github.khronos.org/KTX-Specification/#basislz_gd) and [BasisLZ Bitstream Specification](https://github.khronos.org/KTX-Specification/#basisLZ) sections for more details.

2. Calling the decoder with per-slice data and the target texture format needed. Applications should choose target format depending on platform capabilities and expected usage.

3. Uploading transcoded data to the GPU.

When the texture data uses non-linear sRGB encoding (virtually all color textures do that), applications should use hardware sRGB decoders to get correct filtering. This could be easily achieved by uploading compressed data with proper texture format enum, see specific values below.

### Transcode Target Selection

- By design, ETC1S is a strict subset of ETC1 so transcoding it to ETC formats is always preferred. Depending on alpha channel presence, applications should use the appropriate format enums.

- On desktop GPUs without ETC support, applications should transcode to BC7.

- On older desktops without BC7 support, applications should transcode to BC1 or BC3 depending on alpha channel presence.

- Transcoding to PVRTC1 is also supported but should be used only if other options are not available. Also note that the reference PVRTC1 transcoder supports only textures with power-of-2 dimensions.

- In an unfortunate situation where the platform supports none of aforementioned compressed texture formats, ETC1S data could be decoded to uncompressed RGBA pixels.

![ETC1S Target Format Selection Flowchart](figures/ETC1S_targets.png)

## UASTC Codec

### Overview

UASTC is a virtual block-compressed texture format that is designed for quick and efficient transcoding (i.e. conversion) to hardware-supported block-compressed GPU formats. Being built on top of state of the art ASTC and BC7 texture compression techniques, it can handle all kinds of 8-bit texture data: color maps, normal maps, height maps, etc. By applying RDO (rate-distortion optimization) during encoding, UASTC output can be optimized for subsequent LZ-style lossless compression for more efficient transmission and storage. KTX v2 container format relies on Zstandard for lossless compression.

### Runtime usage

UASTC textures consist of 4x4 blocks, each block takes exactly 16 bytes. Zstandard compression (when present) must be decoded prior to texture transcoding.

Since UASTC is a "virtual" texture format, it has to be converted to one of hardware-supported formats prior to GPU uploading. Applications should choose target format depending on platform capabilities and expected usage.

When the texture data uses non-linear sRGB encoding (virtually all color textures do that), applications should use hardware sRGB decoders to get correct filtering. This could be easily achieved by uploading compressed data with proper texture format enum, see specific values below.

### Primary Transcode Targets

By design, UASTC is optimized for fast and predictable transcoding to ASTC and BC7. Transcoding to ASTC is always lossless (e.g. it would match decoding to RGBA32), transcoding to BC7 is almost lossless.

ASTC should be the default choice when ASTC LDR support is available.

> **Note**: At the time of writing, GPUs supporting ASTC LDR include:
> * Apple A8 and newer, Apple M1
> * Arm Mali-T620 and newer
> * ImgTec PowerVR Series6 and newer
> * Intel Gen9 ("Skylake") and newer
> * NVIDIA Tegra
> * Qualcomm Adreno 3xx series and newer

BC7 should be the default choice when BC7 is supported but ASTC isn't. Such platforms include virtually all desktop GPUs.

When a high-quality output is required (e.g. for normal or other non-color maps) but neither ASTC nor BC7 are supported, UASTC data should be decoded to uncompressed RGBA values. Even when the texture is known to be opaque, it's still usually better to upload it as RGBA instead of RGB due to GPU implementation details.

![UASTC Target Format Selection Flowchart](figures/UASTC_targets.png)

### Additional Transcode Targets

Although transcoding to the following formats may result in some quality loss, it can sometimes be a better choice than decoding to uncompressed considering reduced GPU memory footprint. Usually, the loss is acceptable for textures containing color data.

#### ETC

Transcoding UASTC to ETC involves decoding the texture to uncompressed values and re-encoding them as ETC. This process is partly accelerated by ETC-specific hints that are present in UASTC data. Refer to UASTC specification for more details.

Depending on alpha channel presence, applications should use the appropriate format enums.

#### S3TC (BC1 / BC3)

Transcoding UASTC to S3TC (aka DXT) formats involves decoding the texture to uncompressed values and re-encoding them as BC1 or BC3. Transcoding of RGB data could be partly accelerated by BC1-specific hints that may be present in UASTC data. Refer to UASTC specification for more details.

Depending on alpha channel presence, applications should use the appropriate format enums.

#### PVRTC1

Transcoding to PVRTC1 involves decoding the texture to uncompressed values and re-encoding them as PVRTC1.

The reference UASTC to PVRTC1 transcoder supports only power-of-2 textures.

The reference UASTC to PVRTC1 transcoder needs to know whether the alpha channel is used.

#### 16-bit packed formats

Sometimes, decoding UASTC to 16-bit packed pixel formats (RGB565 or RGBA4444) may yield better results than transcoding to ETC, BC1/BC3, or PVRTC1 at a cost of increased (about 2x) GPU memory footprint.

## GPU API Support

### Compressed Formats

#### ASTC 4x4

The transcoded data uses 16 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format Enum | Linear Format Enum |
|-|-|-|-|
| Vulkan | `textureCompressionASTC_LDR` device feature | `VK_FORMAT_ASTC_4x4_SRGB_BLOCK` | `VK_FORMAT_ASTC_4x4_UNORM_BLOCK` |
| WebGL | `WEBGL_compressed_texture_astc` extension | `COMPRESSED_SRGB8_ALPHA8_ASTC_4x4_KHR` | `COMPRESSED_RGBA_ASTC_4x4_KHR` |
| OpenGL (ES) | `GL_KHR_texture_compression_astc_ldr` extension | `GL_COMPRESSED_SRGB8_ALPHA8_ASTC_4x4_KHR` | `GL_COMPRESSED_RGBA_ASTC_4x4_KHR` |
| Direct3D | N/A | N/A | N/A |
| Metal | `MTLGPUFamilyApple2`-compatible GPU | `MTLPixelFormatASTC_4x4_sRGB` | `MTLPixelFormatASTC_4x4_LDR` |

When the platform supports setting ASTC decode mode (e.g. via `VK_EXT_astc_decode_mode` or `GL_EXT_texture_compression_astc_decode_mode`), applications should set it to `unorm8`.

#### BC1 (S3TC RGB)

The transcoded data uses 8 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format | Linear Format |
|-|-|-|-|
| Vulkan | `textureCompressionBC` device feature | `VK_FORMAT_BC1_RGB_SRGB_BLOCK` | `VK_FORMAT_BC1_RGB_UNORM_BLOCK` |
| WebGL | `WEBGL_compressed_texture_s3tc` and `WEBGL_compressed_texture_s3tc_srgb` extensions | `COMPRESSED_SRGB_S3TC_DXT1_EXT` | `COMPRESSED_RGB_S3TC_DXT1_EXT` |
| OpenGL | `GL_EXT_texture_compression_s3tc` and `GL_EXT_texture_sRGB` extensions | `GL_COMPRESSED_SRGB_S3TC_DXT1_EXT` | `GL_COMPRESSED_RGB_S3TC_DXT1_EXT` |
| OpenGL ES | `GL_EXT_texture_compression_s3tc` and `GL_EXT_texture_compression_s3tc_srgb` extensions | `GL_COMPRESSED_SRGB_S3TC_DXT1_EXT` | `GL_COMPRESSED_RGB_S3TC_DXT1_EXT` |
| Direct3D | `9_1` feature level or higher | `DXGI_FORMAT_BC1_UNORM_SRGB` | `DXGI_FORMAT_BC1_UNORM` |
| Metal | `MTLGPUFamilyMac1`-compatible GPU | `MTLPixelFormatBC1_RGBA_sRGB` | `MTLPixelFormatBC1_RGBA` |

> **Note**: For Direct3D and Metal, BC1 RGBA enums should be used since these APIs do not expose BC1 RGB. Transcoded blocks produced by the reference transcoder will be sampled correctly anyway.

#### BC3 (S3TC RGBA)

The transcoded data uses 16 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format | Linear Format |
|-|-|-|-|
| Vulkan | `textureCompressionBC` device feature | `VK_FORMAT_BC3_SRGB_BLOCK` | `VK_FORMAT_BC3_UNORM_BLOCK` |
| WebGL | `WEBGL_compressed_texture_s3tc` and `WEBGL_compressed_texture_s3tc_srgb` extensions | `COMPRESSED_SRGB_ALPHA_S3TC_DXT5_EXT` | `COMPRESSED_RGBA_S3TC_DXT5_EXT` |
| OpenGL | `GL_EXT_texture_compression_s3tc` and `GL_EXT_texture_sRGB` extensions | `GL_COMPRESSED_SRGB_ALPHA_S3TC_DXT5_EXT` | `GL_COMPRESSED_RGBA_S3TC_DXT5_EXT` |
| OpenGL ES | `GL_EXT_texture_compression_s3tc` and `GL_EXT_texture_compression_s3tc_srgb` extensions | `GL_COMPRESSED_SRGB_ALPHA_S3TC_DXT5_EXT` | `GL_COMPRESSED_RGBA_S3TC_DXT5_EXT` |
| Direct3D | `9_1` feature level or higher | `DXGI_FORMAT_BC3_UNORM_SRGB` | `DXGI_FORMAT_BC3_UNORM` |
| Metal | `MTLGPUFamilyMac1`-compatible GPU | `MTLPixelFormatBC3_RGBA_sRGB` | `MTLPixelFormatBC3_RGBA` |

#### BC7

The transcoded data uses 16 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format | Linear Format |
|-|-|-|-|
| Vulkan | `textureCompressionBC` device feature | `VK_FORMAT_BC7_SRGB_BLOCK` | `VK_FORMAT_BC7_UNORM_BLOCK` |
| WebGL | `EXT_texture_compression_bptc` extension | `COMPRESSED_SRGB_ALPHA_BPTC_UNORM_EXT` | `COMPRESSED_RGBA_BPTC_UNORM_EXT` |
| OpenGL | `GL_ARB_texture_compression_bptc` extension | `GL_COMPRESSED_SRGB_ALPHA_BPTC_UNORM_ARB` | `GL_COMPRESSED_RGBA_BPTC_UNORM_ARB` |
| OpenGL ES | `GL_EXT_texture_compression_bptc` extension | `GL_COMPRESSED_SRGB_ALPHA_BPTC_UNORM_EXT` | `GL_COMPRESSED_RGBA_BPTC_UNORM_EXT` |
| Direct3D | `11_0` feature level | `DXGI_FORMAT_BC7_UNORM_SRGB` | `DXGI_FORMAT_BC7_UNORM` |
| Metal | `MTLGPUFamilyMac1`-compatible GPU | `MTLPixelFormatBC7_RGBAUnorm_sRGB` | `MTLPixelFormatBC7_RGBAUnorm` |

#### ETC1 RGB

The transcoded data uses 8 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format | Linear Format |
|-|-|-|-|
| Vulkan | `textureCompressionETC2` device feature | `VK_FORMAT_ETC2_R8G8B8_SRGB_BLOCK` | `VK_FORMAT_ETC2_R8G8B8_UNORM_BLOCK` |
| WebGL | `WEBGL_compressed_texture_etc` extension | `COMPRESSED_SRGB8_ETC2` | `COMPRESSED_RGB8_ETC2` |
| OpenGL | `GL_ARB_ES3_compatibility` extension | `GL_COMPRESSED_SRGB8_ETC2` | `GL_COMPRESSED_RGB8_ETC2` |
| OpenGL ES | Version 3.0 or higher | `GL_COMPRESSED_SRGB8_ETC2` | `GL_COMPRESSED_RGB8_ETC2` |
| Direct3D | N/A | N/A | N/A |
| Metal | `MTLGPUFamilyApple1`-compatible GPU | `MTLPixelFormatETC2_RGB8_sRGB` | `MTLPixelFormatETC2_RGB8` |

> **Note**: WebGL contexts backed by OpenGL ES 2.0 may support `WEBGL_compressed_texture_etc1` extension that exposes only linear ETC1 texture format (`ETC1_RGB8_OES`). Applications would need to decode sRGB values with a fragment shader.

> **Note**: OpenGL ES 2.0 contexts may support `GL_OES_compressed_ETC1_RGB8_texture` extension that exposes only linear ETC1 texture format (`GL_ETC1_RGB8_OES`). Applications would need to decode sRGB values with a fragment shader.

> **Note**: The reference transcoder implementation produces blocks that don't use ETC2-specific features thus making it possible to use transcoded data on ETC1 hardware.

#### ETC2 RGBA

The transcoded data uses 16 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format | Linear Format |
|-|-|-|-|
| Vulkan | `textureCompressionETC2` device feature | `VK_FORMAT_ETC2_R8G8B8A8_SRGB_BLOCK` | `VK_FORMAT_ETC2_R8G8B8A8_UNORM_BLOCK` |
| WebGL | `WEBGL_compressed_texture_etc` extension | `COMPRESSED_SRGB8_ALPHA8_ETC2_EAC` | `COMPRESSED_RGBA8_ETC2_EAC` |
| OpenGL | `GL_ARB_ES3_compatibility` extension | `GL_COMPRESSED_SRGB8_ALPHA8_ETC2_EAC` | `GL_COMPRESSED_RGBA8_ETC2_EAC` |
| OpenGL ES | Version 3.0 or higher | `GL_COMPRESSED_SRGB8_ALPHA8_ETC2_EAC` | `GL_COMPRESSED_RGBA8_ETC2_EAC` |
| Direct3D | N/A | N/A | N/A |
| Metal | `MTLGPUFamilyApple1`-compatible GPU | `MTLPixelFormatEAC_RGBA8_sRGB` | `MTLPixelFormatEAC_RGBA8` |

> **Note**: Applications running on OpenGL ES 2.0 contexts or WebGL contexts backed by OpenGL ES 2.0 may transcode data to two ETC1 textures: one for RGB and another for alpha. Refer to the previous section for more notes on using ETC1 hardware.

#### PVRTC1

The transcoded data uses 8 bytes per each 4x4 block.

| API | Feature Detection | sRGB Format | Linear Format |
|-|-|-|-|
| Vulkan | `VK_IMG_format_pvrtc` extension | `VK_FORMAT_PVRTC1_4BPP_SRGB_BLOCK_IMG` | `VK_FORMAT_PVRTC1_4BPP_UNORM_BLOCK_IMG` |
| WebGL | `WEBGL_compressed_texture_pvrtc` extension | N/A | `COMPRESSED_RGBA_PVRTC_4BPPV1_IMG` |
| OpenGL | N/A | N/A | N/A |
| OpenGL ES | `GL_IMG_texture_compression_pvrtc` and `GL_EXT_pvrtc_sRGB` extensions | `GL_COMPRESSED_SRGB_ALPHA_PVRTC_4BPPV1_EXT` | `GL_COMPRESSED_RGBA_PVRTC_4BPPV1_IMG` |
| Direct3D | N/A | N/A | N/A |
| Metal | `MTLGPUFamilyApple1`-compatible GPU | `MTLPixelFormatPVRTC_RGBA_4BPP_sRGB` | `MTLPixelFormatPVRTC_RGBA_4BPP` |

> **Note**: WebGL contexts do not support sRGB PVRTC1 formats. Applications would need to decode sRGB values with a fragment shader.

### Uncompressed Formats

#### RGBA32

The decoded data size must be computed from image dimensions in pixels (not blocks) as
```
width * height * 4
```

| API | sRGB Format | Linear Format |
|-|-|-|
| Vulkan | `VK_FORMAT_R8G8B8A8_SRGB` | `VK_FORMAT_R8G8B8A8_UNORM` |
| WebGL | `SRGB8_ALPHA8` | `RGBA8` |
| OpenGL (ES) | `GL_SRGB8_ALPHA8` | `GL_RGBA8` |
| Direct3D | `DXGI_FORMAT_R8G8B8A8_UNORM_SRGB` | `DXGI_FORMAT_R8G8B8A8_UNORM` |
| Metal | `MTLPixelFormatRGBA8Unorm_sRGB` | `MTLPixelFormatRGBA8Unorm` |

> **Note**: WebGL 1.0 contexts require `EXT_sRGB` extension to be enabled for sRGB filtering.

#### 16-bit packed formats

The decoded data size must be computed from image dimensions in pixels (not blocks) as
```
width * height * 2
```

| API | Feature Detection | RGB | RGBA |
|-|-|-|-|
| Vulkan | Always supported | `VK_FORMAT_R5G6B5_UNORM_PACK16` | `VK_FORMAT_R4G4B4A4_UNORM_PACK16` |
| WebGL | Always supported | `RGB565` | `RGBA4` |
| OpenGL (ES) | Always supported | `GL_RGB565` | `GL_RGBA4` |
| Direct3D | WDDM 1.2 or newer | `DXGI_FORMAT_B5G6R5_UNORM` | N/A |
| Metal | `MTLGPUFamilyApple1`-compatible GPU | `MTLPixelFormatB5G6R5Unorm` | `MTLPixelFormatABGR4Unorm` |

> **Note**: WebGL applications should be cautious with using these formats because they will be emulated as RGBA32 when the underlying platform does not support packed 16-bit formats natively.

> **Note**: There is no hardware sRGB decoding support for packed formats. Applications would need to decode sRGB values with a fragment shader.
