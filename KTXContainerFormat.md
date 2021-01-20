# Reading KTX 2.0 Containers

Universal Textures are comprised of two formats, written to a single file:

1. **Compression format:** A compact representation of pixel data, in a layout designed for very efficient transcoding to one or more GPU compression formats. See ETC1S and UASTC, below.
2. **Container format:** An explanatory wrapper around the compression format, describing how that data should be used by the client software.

Without a container format, compressed data cannot be correctly moved between software applications. By default, the `basisu` utility produces `.basis` files, a custom container format understood by the Basis Universal reference transcoder. For more general interoperability, and particularly for use within portable 3D formats like [glTF 2.0](https://github.com/KhronosGroup/glTF), we instead recommend using the [KTX 2.0 Container Format](http://github.khronos.org/KTX-Specification/) (`.ktx2`). KTX supports the same underlying compression formats, but provides a more fully-specified container description, license protections under the Khronos IP Framework, and the likelihood of wider support from 3D authoring software over time.

KTX 2.0 is a relatively simple binary data format, and can be read or written without the use of existing software, by referencing the format specification. Several implementations are available, however, and are particularly recommended for texture authoring software.

- [KTX-Software](https://github.com/KhronosGroup/KTX-Software/): Official C/C++ libraries for reading, writing, and transcoding KTX files, with optional functionality for instantiating textures in various graphics APIs. Includes [downloadable binary packages](https://github.com/KhronosGroup/KTX-Software/releases) and WebAssembly builds.
- [KTX-Parse](https://github.com/donmccurdy/KTX-Parse): Lightweight JavaScript/TypeScript/Node.js library for reading and writing KTX files. Provides all data required by the Basis transcoder, but transcoding the attached compressed format must be handled separately by the application.
