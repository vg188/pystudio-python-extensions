# PyStudio Python Extensions

This repository builds optional Python/Pip extension package repositories for
PyStudio.

The app should keep the normal user-facing flow as `pip install ...`. Extension
packages are provided in two tracks:

1. `pip-build-core` plus `native-libs-*` profiles provide compilers, build
   tools, headers, and native libraries so Android pip builds have a better
   chance to succeed. Heavy Rust-based build helpers live in `pip-build-rust`
   and are kept out of the default profile.
2. `prebuilt-python-*` profiles remain available as fallback packages for
   common modules that are too slow, too large, or too fragile to compile on
   device.

## Profiles

Profiles live in `profiles/python-extensions/*.txt`.

| Profile | Purpose |
| --- | --- |
| `pip-build-core` | Pip, compilers, make, CMake, Ninja, pkg-config, pybind11, and common headers. |
| `pip-build-rust` | Heavy Rust and uv tooling for Rust-backed Python packages. Keep this explicit and out of default builds. |
| `native-libs-scientific` | Native OpenBLAS, FFTW, SuiteSparse, HDF5, NetCDF, and numeric dependencies for pip builds. |
| `native-libs-data` | Native DuckDB, Arrow, ORC, HDF5, SQLite, YAML, XML, and compression libraries. |
| `native-libs-image` | Native JPEG, PNG, TIFF, WebP, OpenJPEG, FreeType, Fontconfig, GraphicsMagick, Leptonica, and Tesseract dependencies. |
| `native-libs-visualize` | Native Cairo, Pango, Graphviz, and Gnuplot dependencies. |
| `native-libs-markup` | Native XML, HTML, YAML, XML security, cleanup, and tree-sitter parser dependencies. |
| `native-libs-crypto-network` | Native OpenSSL, libffi, libsodium, SSH, curl, HTTP/2, protobuf, gRPC, Kerberos, and c-ares dependencies. |
| `native-libs-media` | Native ffmpeg, FLAC, Ogg/Vorbis/Opus, TagLib, SoX, and media codec dependencies. |
| `prebuilt-python-scientific` | Prebuilt NumPy/SciPy-style fallback packages. |
| `prebuilt-python-data` | Prebuilt data processing, serialization, storage, and compression fallback packages. |
| `prebuilt-python-image` | Prebuilt image processing fallback packages such as Pillow. |
| `prebuilt-python-visualize` | Prebuilt plotting and visualization fallback packages. |
| `prebuilt-python-markup` | Prebuilt XML/HTML parsing fallback packages such as lxml. |
| `prebuilt-python-crypto-network` | Prebuilt cryptography, bcrypt, PyCryptodomeX, and gRPC fallback packages. |
| `prebuilt-python-media` | Prebuilt media-related fallback packages. |
| `prebuilt-python-ai-ml` | Heavy AI/ML fallback packages. Keep this explicit and out of default builds. |

## GitHub Actions

Run **Build PyStudio Python Extensions** manually.

Inputs:

- `profile`: one profile above or one grouped profile.
- `profile=standard` builds the recommended pip path:
  `pip-build-core` plus all `native-libs-*` profiles. It does not include
  `pip-build-rust`.
- `profile=prebuilt` builds non-AI `prebuilt-python-*` fallback profiles.
- `profile=all` builds `standard` plus non-AI prebuilt fallback profiles.
- `profile=all-with-ai` also includes `prebuilt-python-ai-ml`.
- `architectures`: default is `aarch64,arm,i686,x86_64`.
- `packages`: optional package override for experiments.
- `publish_release`: publish tarballs to a GitHub release.

Outputs per profile and architecture:

- `pystudio-python-extensions-<profile>-repo-<arch>.tar.gz`
- `pystudio-python-extensions-<profile>-debs-<arch>.tar.gz`
- `SHA256SUMS-<profile>-<arch>.txt`

The `repo` tarball is the app-facing apt repository artifact. The `debs`
tarball is kept for debugging or direct fallback installation.

## App Integration

After a profile build succeeds, add its `repo` tarball URLs to the PyStudio
runtime package manifest as optional extension profiles.

Recommended install flow inside the app:

1. Pull and extract the selected extension repository.
2. Write `deb [trusted=yes] file:<repoDir> stable main` into
   `$PREFIX/etc/apt/sources.list.d/pystudio-extension-<profile>.list`.
3. Run `apt-get update` scoped to that source list.
4. Run `apt-get install -y <profile packages>`.
5. Let users continue with normal `pip install ...`.

Prefer `standard` for the first pass. Use `prebuilt` only when a user needs a
known fallback package, and keep `prebuilt-python-ai-ml` as an explicit advanced
download because it is expected to be large and slower to build.
