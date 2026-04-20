# Docker build image

This directory contains the Dockerfile for the nightly build environment image.
The image pre-fetches Cargo registry and Git dependencies and precompiles the
release dependency graph for `zenoh-plugin-ros1`. The `rosrust` workspace is
copied as real source, while the top-level bridge and plugin crates use minimal
stub sources during image build. It is not a runtime image for
`zenoh-bridge-ros1`.

Build and publish the multi-architecture image from the repository root:

```bash
docker login

docker buildx create --name zenoh-ros1-builder --use --bootstrap

docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -f docker_dev_instruction/Dockerfile \
  -t shupeixuan/zenoh-plugin-ros1-build:latest \
  --push \
  .
```

The pushed tag is a multi-arch manifest, so the amd64 and arm64 GitHub runners
will pull the native image variant automatically.

The image stores precompiled release artifacts in
`/opt/zenoh-plugin-ros1/target`. The nightly workflow sets `CARGO_TARGET_DIR` to
that path so the mounted source tree can reuse the cached dependency build.

For a local single-architecture smoke test without pushing:

```bash
docker buildx build \
  --platform "$(docker version -f '{{.Server.Os}}/{{.Server.Arch}}')" \
  -f docker_dev_instruction/Dockerfile \
  -t shupeixuan/zenoh-plugin-ros1-build:local \
  --load \
  .
```

The build context must be the repository root so the root manifests, workspace
members, and the `rosrust` submodule are available. Make sure the submodule is
checked out before building:

```bash
git submodule update --init --recursive
```

For more details on how to use the image for building the bridge and plugin, see [readme_build_with_docker.md](readme_build_with_docker.md).
