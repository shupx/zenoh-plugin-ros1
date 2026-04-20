FROM ubuntu:20.04
# FROM docker.1ms.run/ubuntu:20.04

USER root

ENV DEBIAN_FRONTEND=noninteractive \
    RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static \
    RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup \
    CARGO_HOME=/root/.cargo \
    RUSTUP_HOME=/root/.rustup \
    CARGO_TARGET_DIR=/opt/zenoh-plugin-ros1/target \
    PATH=/root/.cargo/bin:${PATH}

WORKDIR /opt/zenoh-plugin-ros1

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        clang \
        curl \
        git \
        pkg-config \
        python3 \
    && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain 1.85.0 --profile default \
    && rustc --version \
    && cargo --version \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# # 配置 Git 代理以加速依赖下载
# RUN git config --global url."https://gh-proxy.org/https://github.com/".insteadOf "https://github.com/" && git config --global fetch.depth 1

# Use the Git CLI for Cargo git dependencies so any global Git configuration
# applied by image users is honored.
RUN mkdir -p /root/.cargo \
    && printf '%s\n%s\n' '[net]' 'git-fetch-with-cli = true' > /root/.cargo/config.toml

# Copy only manifests and lockfiles first so dependency downloads are cached
# independently from normal source edits.
COPY Cargo.toml Cargo.lock rust-toolchain.toml LICENSE ./
COPY zenoh-bridge-ros1/Cargo.toml zenoh-bridge-ros1/Cargo.toml
COPY zenoh-plugin-ros1/Cargo.toml zenoh-plugin-ros1/Cargo.toml
COPY rosrust/ rosrust/

RUN mkdir -p \
        zenoh-bridge-ros1/src \
        zenoh-plugin-ros1/src \
    && printf '%s\n' 'fn main() {}' > zenoh-bridge-ros1/src/main.rs \
    && printf '%s\n' 'pub fn cargo_cache_probe() {}' > zenoh-plugin-ros1/src/lib.rs \
    && printf '%s\n' 'fn main() {}' > zenoh-plugin-ros1/build.rs

# Fetch and precompile the release dependency graph with stub sources. The real
# nightly build mounts the repository at /workspace but reuses this target dir,
# so Cargo only needs to rebuild crates whose source actually changed.
RUN cargo fetch --locked \
    && cargo build --release --locked -p zenoh-bridge-ros1 -p zenoh-plugin-ros1

CMD ["bash"]
