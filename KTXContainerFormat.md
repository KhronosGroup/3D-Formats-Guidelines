# KTX Container Format

When implementing support for compressed textures, developers should be aware of three conceptual formats:

1. **Transmission format:** A highly compressed representation of pixel data, in a layout designed for very efficient transcoding to one or more GPU compression formats.
	- _Examples: ETC1S and UASTC_
2. **GPU compressed pixel format:** A compressed representation of pixel data, understood by the GPU.
	- _Examples: BCn, ASTC, ETC, etc._
3. **Container format:** An explanatory wrapper around the transmission format data. Describes image dimensions, compression types, and how to access and transcode that data into a GPU-compatible pixel format. Without a container format, compressed data cannot be correctly moved between software applications.
	- _Examples: KTX 2.0_

 To represent textures within portable [glTF 2.0](https://github.com/KhronosGroup/glTF) 3D models, we use the [KTX 2.0 Container Format](http://github.khronos.org/KTX-Specification/)<sup>1</sup> (`.ktx2`). KTX 2.0 is a relatively simple binary data format, and can be read or written without the use of existing software, by referencing the format specification. Several existing implementations are available:

- [KTX-Software](https://github.com/KhronosGroup/KTX-Software/): Official C/C++ libraries for reading, writing, and transcoding KTX files, with optional functionality for instantiating textures in various graphics APIs. Includes [downloadable binary packages](https://github.com/KhronosGroup/KTX-Software/releases) and WebAssembly builds.
- [KTX-Parse](https://github.com/donmccurdy/KTX-Parse): Lightweight JavaScript/TypeScript/Node.js library for reading and writing KTX files. Provides all data required by the Basis transcoder, but transcoding to a GPU compressed format must be handled separately by the application.

<small><sup>1</sup> Note that the `basisu` utility produces `.basis` files, a custom container format understood by the Basis Universal reference transcoder. KTX 2.0 (`.ktx2`) supports the same underlying transmission and transcoder target formats, but provides a more fully-specified container description, license protections under the Khronos IP Framework, and the likelihood of wider support from 3D authoring software over time.</small>