# KTX 2.0 tools

[KTX-Software](https://github.com/KhronosGroup/KTX-Software/) provides official C/C++/WASM libraries for reading, writing, and transcoding KTX files, Includes [downloadable binary packages](https://github.com/KhronosGroup/KTX-Software/releases) and on-line [documentation](https://github.khronos.org/KTX-Software/).

*For other projects intended for specific tasks related to KTX 2.0, see functional sections below.*

## Read/Write KTX textures

- [Binomial Encoder](https://github.com/BinomialLLC/basis_universal): Binomial C/C++, WASM, and CLI encoder, for Basis UASTC and ETC1S texture modes.
- [KTX-Parse](https://github.com/donmccurdy/KTX-Parse): Lightweight JavaScript/TypeScript/Node.js library for reading and writing KTX files. Transcoding to a GPU compressed format must be handled separately.
- [Kram](https://github.com/alecazam/kram) can be used to encode and bundle BC/ETC2/ASTC files as KTX/KTX2. Scripted support for KTX2 and Basis files using libktx tools. Windows/macOS with binary releases.

## Transcode KTX to GPU formats

- [Binomial Transcoder](https://github.com/BinomialLLC/basis_universal): Binomial C/C++ and WASM transcoder, for Basis UASTC and ETC1S texture modes.
- [Khronos Transcoders](https://github.com/KhronosGroup/Basis-Universal-Transcoders): Lightweight WASM transcoders, for Basis UASTC texture mode.

## View KTX textures

- [BabylonJS Sandbox](https://sandbox.babylonjs.com/): allows users to view standalone KTX textures, or glTF models using embedded KTX textures.
- [Gestaltor Editor](https://gestaltor.io/): Visual glTF editor with KTX read/write support.
- [Kramv](https://github.com/alecazam/kram): Viewer for BC/ASTC/ETC2 and HDR KTX textures. Windows/macOS with binary releases.
- [Image Viewer and Tonemapper](https://github.com/kopaka1822/ImageViewer): Can be used for viewing, image comparisons and statistics, tonemapping and mipmap generation. This has experimental KTX 2 support. Windows only.

## KTX textures in glTF 3D models

- [Gestaltor Editor](https://gestaltor.io/): Visual glTF editor with KTX read/write support.
- [gltf-transform](https://gltf-transform.donmccurdy.com/cli.html): CLI tool that allows converting a glTF model’s textures to KTX.
- [gltfpack](https://github.com/zeux/meshoptimizer/tree/master/gltf): CLI tool that allows converting a glTF model’s textures to KTX.
- [RapidCompact](https://rapidcompact.com/): Online platform for optimization of 3D data.
