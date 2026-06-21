# PyStudio Python Extensions

This repository builds optional Python/Pip extension package repositories for
PyStudio.

The base app should install only the minimal terminal bootstrap and the core
Python/Pip toolchain. These extension profiles are optional downloads that help
`pip install` work better on Android when packages need native libraries,
compilers, or common scientific/media/XML/network dependencies.

## Profiles

Profiles live in `profiles/python-extensions/*.txt`.

| Profile | Purpose |
| --- | --- |
| `build-core` | Compilers, build tools, pkg-config, Rust, pybind11, CMake, and common headers. |
| `scientific` | NumPy/SciPy and numeric libraries such as OpenBLAS, FFTW, SuiteSparse, HDF5, NetCDF. |
| `data` | Data storage/interchange libraries such as DuckDB, Arrow, ORC, HDF5, SQLite, YAML, Zstd. |
| `image` | Pillow and raster/image dependencies such as JPEG, PNG, TIFF, WebP, ImageMagick, Tesseract. |
| `visualize` | Matplotlib, contourpy, Cairo, Pango, Graphviz, and Gnuplot. |
| `markup` | lxml, libxml2/libxslt, XML security, HTML cleanup/parsing, tree-sitter parsers. |
| `crypto-network` | cryptography, bcrypt, PyCryptodomeX, gRPC, OpenSSL, libcurl, SSH, HTTP/2. |
| `media` | Audio/video helpers such as ffmpeg, lame, FLAC, Ogg/Vorbis/Opus, TagLib, SoX. |
| `ai-ml` | Heavy AI/ML packages such as Torch, TorchVision, TorchAudio, TFLite, ONNX Runtime. |

## GitHub Actions

Run **Build PyStudio Python Extensions** manually.

Inputs:

- `profile`: one profile above or `all`.
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

Keep `ai-ml` separate from default downloads because it is expected to be large
and slower to build.
