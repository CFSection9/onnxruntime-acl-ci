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

## Notes

- First build takes 1-2 hours (compiling ACL + ONNX Runtime)
- Subsequent builds cached automatically
- Both ACL and ONNX Runtime built with Release optimization
- ACL uses Neon SIMD, no OpenCL
