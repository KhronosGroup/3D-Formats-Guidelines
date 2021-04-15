# KTX 2.0 tools

[KTX-Software](https://github.com/KhronosGroup/KTX-Software/) provides official C/C++/WASM libraries for reading, writing, and transcoding KTX files, Includes [downloadable binary packages](https://github.com/KhronosGroup/KTX-Software/releases).

*For other projects intended for specific tasks related to KTX 2.0, see functional sections below.*

## Read/Write KTX textures

- [Basis Universal](https://github.com/BinomialLLC/basis_universal): Binomial C/C++, WASM, and CLI compression tools, for Basis UASTC and ETC1S texture modes.
- [KTX-Parse](https://github.com/donmccurdy/KTX-Parse): Lightweight JavaScript/TypeScript/Node.js library for reading and writing KTX files. Transcoding to a GPU compressed format must be handled separately.

## Transcode KTX to GPU formats

- [Binomial Transcoders](https://github.com/BinomialLLC/basis_universal): Binomial C/C++ and WASM transcoders, for all Basis input formats and transcoding targets.
- [Khronos Transcoders](https://github.com/KhronosGroup/Basis-Universal-Transcoders): Lightweight WASM transcoders, for selected Basis input formats and transcoding targets.

## View KTX textures

- [BabylonJS Sandbox](https://sandbox.babylonjs.com/): allows users to view standalone KTX textures, or glTF models using embedded KTX textures.
- [Gestaltor Editor](https://gestaltor.io/): Visual glTF editor with KTX read/write support.

## KTX textures in glTF 3D models

- [Gestaltor Editor](https://gestaltor.io/): Visual glTF editor with KTX read/write support.
- [gltf-transform](https://gltf-transform.donmccurdy.com/cli.html): CLI tool that allows converting a glTF model’s textures to KTX.
- [gltfpack](https://github.com/zeux/meshoptimizer/tree/master/gltf): CLI tool that allows converting a glTF model’s textures to KTX.
- [RapidCompact](https://rapidcompact.com/): Online platform for optimization of 3D data.
