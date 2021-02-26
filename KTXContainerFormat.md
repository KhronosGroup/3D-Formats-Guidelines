# KTX Container Format

When implementing support for compressed textures, developers should be aware of three conceptual formats:

1. **Container format:** An explanatory wrapper around the transmission format data. Describes image dimensions, compression types, and how to access and transcode that data into a GPU-compatible pixel format. Without a container format, compressed data cannot be correctly moved between software applications.
2. **Transmission format:** A highly compressed representation of pixel data, in a layout designed for very efficient transcoding to one or more GPU compression formats.
	- _Examples: ETC1S and UASTC_
3. **GPU compressed pixel format:** A compressed representation of pixel data, understood by the GPU.
	- _Examples: BCn, ASTC, ETC, and PVRTC1_
	- _Examples: KTX 2.0_

 Portable [glTF 2.0](https://github.com/KhronosGroup/glTF) 3D assets may use compressed textures stored in the [KTX 2.0 Container Format](http://github.khronos.org/KTX-Specification/) (`.ktx2`)<sup>1</sup>, as described by the [`KHR_texture_basisu`](https://github.com/KhronosGroup/glTF/tree/master/extensions/2.0/Khronos/KHR_texture_basisu) glTF extension. KTX 2.0 is a relatively simple binary data format, and can be read or written without the use of existing software by referencing the format specification. Several existing implementations are available:

- [KTX-Software](https://github.com/KhronosGroup/KTX-Software/): Official C/C++ libraries for reading, writing, and transcoding KTX files, with optional functionality for instantiating textures in various graphics APIs. Includes [downloadable binary packages](https://github.com/KhronosGroup/KTX-Software/releases) and WebAssembly builds.
- [KTX-Parse](https://github.com/donmccurdy/KTX-Parse): Lightweight JavaScript/TypeScript/Node.js library for reading and writing KTX files. Transcoding to a GPU compressed format must be handled separately<sup>2</sup>.

<small><sup>1</sup> Note that the `basisu` utility produces `.basis` files, a custom container format understood by the [Binomial C/C++/WASM transcoders](https://github.com/BinomialLLC/basis_universal/). KTX 2.0 (`.ktx2`) supports the same underlying transmission and transcoder target formats, but provides a more fully-specified container description, license protections under the Khronos IP Framework, and the likelihood of wider support from 3D authoring software over time.</small>

<small><sup>2</sup> Transcoders from Basis Universal transmission formats to GPU compression formats are included in [KTX-Software](https://github.com/KhronosGroup/KTX-Software/), or available independently in [Binomial C/C++/WASM transcoders](https://github.com/BinomialLLC/basis_universal/), or [Khronos Group WASM transcoders](https://github.com/KhronosGroup/Basis-Universal-Transcoders).

</small>