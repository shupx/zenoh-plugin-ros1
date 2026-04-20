# Standalone build with Docker

The nightly workflow uses `shupeixuan/zenoh-plugin-ros1-build:latest` to build
the standalone Linux bridge binary and plugin shared library on native amd64 and
arm64 runners.

The `latest` tag should be published as a multi-architecture image for
`linux/amd64` and `linux/arm64`; see [README.md](README.md) for the `buildx`
publish command. The nightly workflow only pulls and uses the image.

The image includes a precompiled release dependency cache at
`/opt/zenoh-plugin-ros1/target`. Set `CARGO_TARGET_DIR` to this path when using
the image so Cargo can reuse the cached dependencies.

To run the same release build locally from the repository root:

```bash
docker run --rm \
  -v "$PWD:/workspace" \
  -w /workspace \
  -e CARGO_TARGET_DIR=/opt/zenoh-plugin-ros1/target \
  shupeixuan/zenoh-plugin-ros1-build:latest \
  bash -lc 'cargo build --release --locked -p zenoh-bridge-ros1 -p zenoh-plugin-ros1'
```

The expected build outputs are:

```text
/opt/zenoh-plugin-ros1/target/release/zenoh-bridge-ros1
/opt/zenoh-plugin-ros1/target/release/libzenoh_plugin_ros1.so
```

To copy the prebuilt binary and shared library from the image to the host
without running a build:

```bash
mkdir -p dist/local-docker

docker run --rm \
  -v "$PWD:/workspace" \
  -w /workspace \
  -e CARGO_TARGET_DIR=/opt/zenoh-plugin-ros1/target \
  shupeixuan/zenoh-plugin-ros1-build:latest \
  bash -lc '
    set -euo pipefail
    cp "$CARGO_TARGET_DIR/release/zenoh-bridge-ros1" /workspace/dist/local-docker/
    cp "$CARGO_TARGET_DIR/release/libzenoh_plugin_ros1.so" /workspace/dist/local-docker/
  '
```

The files will be available on the host at:

```text
dist/local-docker/zenoh-bridge-ros1
dist/local-docker/libzenoh_plugin_ros1.so
```

The nightly workflow copies these files into `dist/<platform>/` and renames
them with platform suffixes before uploading them to the GitHub `nightly`
release, for example:

```text
zenoh-bridge-ros1-linux-amd64
libzenoh_plugin_ros1-linux-amd64.so
zenoh-bridge-ros1-linux-arm64
libzenoh_plugin_ros1-linux-arm64.so
```
