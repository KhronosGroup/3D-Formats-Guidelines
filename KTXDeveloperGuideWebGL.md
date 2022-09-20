# KTX 2.0 / Basis Universal Textures — WebGL Tips

To ensure that transcoding from the transmission texture formats (i.e. ETC1S or UASTC) to GPU compressed formats works as expected, developers should test their WebGL applications on a wide range of platforms due to differences in compressed format support. Still, there are two setups that provide exceptionally high level of compatibility: modern Intel GPUs (Windows and Linux) and Apple M1 and newer SoC (macOS).

> **Warning**: It is highly recommended to use Chromium Dev builds instead of altering the main Chrome or Edge installation because using non-default ANGLE backend may decrease stability or security.

## Windows and Linux

Modern Intel GPUs, namely Intel HD Graphics 5xx ("Skylake") or newer, have the best compressed format coverage on Windows and Linux. OpenGL or Vulkan are required to get access to ASTC and ETC formats.

1. Consider using the latest GPU driver. On Windows, it could be downloaded from the [Intel website](https://downloadcenter.intel.com/); on Linux, it is usually provided and maintained by OS distribution vendors.

2. Install [Google Chrome Dev](https://www.google.com/chrome/dev/) or [Microsoft Edge Dev](https://www.microsoftedgeinsider.com/en-us/download/).

3. On Windows, open `about://flags/#use-angle` page, select OpenGL as ANGLE graphics backend, and restart the browser.

4. Confirm enabled WebGL extensions here: https://webglreport.com/, the list should include:
   - [`EXT_texture_compression_bptc`](https://www.khronos.org/registry/webgl/extensions/EXT_texture_compression_bptc)
   - [`EXT_texture_compression_rgtc`](https://www.khronos.org/registry/webgl/extensions/EXT_texture_compression_rgtc)
   - [`WEBGL_compressed_texture_astc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_astc)
   - [`WEBGL_compressed_texture_etc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_etc)
   - [`WEBGL_compressed_texture_s3tc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_s3tc)
   - [`WEBGL_compressed_texture_s3tc_srgb`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_s3tc_srgb)

## macOS

Apple M1 and newer SoC support all possible compressed transcode targets: BC*, ETC*, ASTC, and PVRTC1. Most of them are available only via Metal API. Intel-based Macs support only BC* formats.

### Chromium-based browsers

1. Install [Google Chrome Dev](https://www.google.com/chrome/dev/) or [Microsoft Edge Dev](https://www.microsoftedgeinsider.com/en-us/download/).

2. Open `about://flags/#use-angle` page, select Metal as ANGLE graphics backend, and restart the browser.

3. Confirm enabled WebGL extensions here: https://webglreport.com/, the list should include:
   - [`EXT_texture_compression_bptc`](https://www.khronos.org/registry/webgl/extensions/EXT_texture_compression_bptc)
   - [`EXT_texture_compression_rgtc`](https://www.khronos.org/registry/webgl/extensions/EXT_texture_compression_rgtc)
   - [`WEBGL_compressed_texture_astc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_astc)
   - [`WEBGL_compressed_texture_etc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_etc)
   - [`WEBGL_compressed_texture_pvrtc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_pvrtc)
   - [`WEBGL_compressed_texture_s3tc`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_s3tc)
   - [`WEBGL_compressed_texture_s3tc_srgb`](https://www.khronos.org/registry/webgl/extensions/WEBGL_compressed_texture_s3tc_srgb)

### Safari

1. Ensure that macOS 12 Monterey or newer is used.

2. Ensure that Safari 16 or newer is used.

3. Confirm enabled WebGL extensions here: https://webglreport.com/, the list should be the same as with Chromium-based browsers running on Metal.

## Platform Support Table

|                          | AMD                                         | Apple Ax     | Apple M1             | ARM                 | Intel                             | NVIDIA Desktop                           | NVIDIA Tegra | Qualcomm             |
|:------------------------:|---------------------------------------------|--------------|----------------------|---------------------|-----------------------------------|------------------------------------------|--------------|----------------------|
|           ASTC           | ❌                                           | A8 and newer | ✅<sup><b>2</b></sup> | Mali-T620 and newer | Gen9 and newer<sup><b>3</b></sup> | ❌                                        | ✅            | Adreno 3xx and newer |
| BC1<br>BC3<br>BC4<br>BC5 | ✅                                           | ❌            | ✅                    | ❌                   | ✅                                 | ✅                                        | ✅            | ❌                    |
|            BC7           | Radeon HD 5000 and newer<sup><b>1</b></sup> | ❌            | ✅<sup><b>1</b></sup> | ❌                   | Gen7 and newer<sup><b>1</b></sup> | GeForce 400 and newer<sup><b>1</b></sup> | ✅            | ❌                    |
|           ETC1           | ❌                                           | A7 and newer | ✅<sup><b>2</b></sup> | Mali-300 and newer  | Gen8 and newer<sup><b>3</b></sup> | ❌                                        | ✅            | Adreno 2xx and newer |
|        ETC2 / EAC        | ❌                                           | A7 and newer | ✅<sup><b>2</b></sup> | Mali-T6xx and newer | Gen8 and newer<sup><b>3</b></sup> | ❌                                        | ✅            | Adreno 3xx and newer |
|          PVRTC1          | ❌                                           | ✅            | ✅<sup><b>2</b></sup> | ❌                   | ❌                                 | ❌                                        | ❌            | ❌                    |

<p><sup>1</sup> On macOS, only Metal API supports BC7.</p>
<p><sup>2</sup> Use of Metal API is required.</p>
<p><sup>3</sup> Available only on Windows and Linux, requires use of OpenGL or Vulkan.</p>
