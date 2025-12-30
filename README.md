# ONNX Runtime with ACL for Android

Build ONNX Runtime with ARM Compute Library (ACL) support for Android arm64-v8a.

## Quick Start

This repository is configured to build ONNX Runtime with ARM Compute Library (ACL) for Android entirely on GitHub Actions. Local build scripts have been removed — CI runs are the supported and tested path.

### How to trigger builds
- Push to `main` or `develop`
- Create a tag matching `v*` (releases)
- Run the workflow manually via `workflow_dispatch` from the Actions tab

The CI workflow is: `.github/workflows/build-with-acl.yml`

**Outputs produced by CI:**
- Cached ACL build artifacts
- ONNX Runtime `.so` files as workflow artifacts
- Automatic GitHub release when pushing `v*` tags

## ACL Execution Provider Availability

**Status:** Community-maintained execution provider

**Important:** There are no official prebuilt binaries or wheels that include the ACL Execution Provider. Unlike other execution providers (CUDA, TensorRT, CoreML), ACL EP is not available through pip, Conda, Maven, or any package manager.

**To use ACL EP, you must build ONNX Runtime from source** as demonstrated in this repository's CI workflow.

**Why no prebuilt binaries?**
- ACL EP is community-maintained, not officially packaged by Microsoft
- ARM Compute Library must be built for specific target hardware configurations
- Typical prebuilt mobile packages include NNAPI, CoreML, or XNNPACK instead

**Analogy:** Think of ONNX Runtime as a universal appliance, and execution providers as specialized attachments. ACL is a high-performance ARM attachment you need to build and install yourself—it's not included in the default package.

For official documentation, see:
- [ACL Execution Provider Documentation](https://onnxruntime.ai/docs/execution-providers/ACL-ExecutionProvider.html)
- [Supported Execution Providers List](https://onnxruntime.ai/docs/execution-providers/)

### Release

Tag and push:
```bash
git tag v1.0.0
git push origin v1.0.0
```

GitHub Actions will:
1. Build ACL (latest tag)
2. Build ONNX Runtime (latest tag)
3. Create release with .so files

## Project Structure

```
ONNX_ACL/
├── .github/workflows/
│   └── build-with-acl.yml     # GitHub Actions CI/CD
├── (build logic is in `.github/workflows/build-with-acl.yml`)
├── .gitignore
└── README.md
```

## Configuration

| Setting | Value |
|---------|-------|
| **Android API** | 24+ |
| **ABI** | arm64-v8a |
| **ACL Version** | Latest tag |
| **ONNX Runtime** | Latest tag |
| **Output** | .so files |

## Output Files

```
artifacts/
├── libonnxruntime.so
├── libonnxruntime_c.so
└── libonnxruntime_providers_acl.so
```

## Build Flags and Configuration

### ARM Compute Library (ACL) Build

The workflow builds ACL using SCons with the following parameters:

```bash
scons \
  os=android \              # Target Android OS
  arch=arm64-v8a \          # 64-bit ARM architecture
  neon=1 \                  # Enable ARM Neon SIMD instructions
  opencl=0 \                # Disable OpenCL (CPU-only build)
  build=cross_compile \     # Cross-compilation mode
  build_dir=../acl-build \ # Output directory
  -j$(nproc)                # Parallel build
```

**Key decisions:**
- **Neon enabled:** Leverages ARM SIMD for acceleration
- **OpenCL disabled:** CPU-only build, no GPU acceleration (may be enabled in future)
- **arm64-v8a only:** 64-bit ARM devices (Android 5.0+)

### ONNX Runtime Build

ONNX Runtime is built using the official `build.py` script with these flags:

```bash
python3 tools/ci_build/build.py \
  --android \                              # Android cross-compilation
  --android_sdk_path $ANDROID_SDK_ROOT \   # Android SDK location
  --android_ndk_path $ANDROID_NDK \        # NDK r26b
  --android_abi arm64-v8a \                # Target ABI
  --android_api_level 24 \                 # Minimum Android 7.0
  --android_toolchain $ANDROID_NDK/build/cmake/android.cmake \
  --use_acl \                              # Enable ACL Execution Provider
  --acl_home /path/to/acl-build \          # ACL library location
  --build_shared_lib \                     # Build .so files
  --cmake_extra_defines "onnxruntime_PREFER_SYSTEM_LIB=OFF" \
  --config Release \                       # Release build
  --update \                               # Update dependencies
  --build                                  # Execute build
```

**Environment:**
- **Android NDK:** r26b (LTS release)
- **Min API Level:** 24 (Android 7.0+)
- **Build Tools:** 34.0.0
- **Architecture:** arm64-v8a only
- **ACL Version:** Latest stable tag from [ARM-software/ComputeLibrary](https://github.com/ARM-software/ComputeLibrary)
- **ONNX Runtime Version:** Latest stable tag from [microsoft/onnxruntime](https://github.com/microsoft/onnxruntime)

## Notes

- First build takes 1-2 hours (compiling ACL + ONNX Runtime)
- Subsequent builds cached automatically
- Both ACL and ONNX Runtime built with Release optimization
- ACL uses Neon SIMD, no OpenCL

## References

### Official ONNX Runtime Documentation

- **Execution Providers Overview:** [https://onnxruntime.ai/docs/execution-providers/](https://onnxruntime.ai/docs/execution-providers/)
- **ACL Execution Provider:** [https://onnxruntime.ai/docs/execution-providers/ACL-ExecutionProvider.html](https://onnxruntime.ai/docs/execution-providers/ACL-ExecutionProvider.html)
- **Building with Execution Providers:** [https://onnxruntime.ai/docs/build/eps.html#arm-compute-library](https://onnxruntime.ai/docs/build/eps.html#arm-compute-library)

### ARM Compute Library

- **ACL GitHub Repository:** [https://github.com/ARM-software/ComputeLibrary](https://github.com/ARM-software/ComputeLibrary)
- **ACL Documentation:** [https://arm-software.github.io/ComputeLibrary/latest/](https://arm-software.github.io/ComputeLibrary/latest/)

### Usage and Integration

For code examples and integration guidance, refer to the official ONNX Runtime documentation:
- C++ API: [Session Configuration](https://onnxruntime.ai/docs/api/c/struct_ort_api.html)
- Java/Android: [Android Integration Guide](https://onnxruntime.ai/docs/tutorials/mobile/)

## Future Work

Potential enhancements planned for future releases:

- **OpenCL GPU Acceleration:** Enable `opencl=1` in ACL build to leverage GPU compute on supported devices
- **32-bit ARM Support:** Add `armeabi-v7a` build variant for older Android devices and broader compatibility
- **Multi-ABI Builds:** Package both arm64-v8a and armeabi-v7a in a single release artifact
- **Performance Benchmarks:** Add automated benchmarking with sample models to track optimization gains
