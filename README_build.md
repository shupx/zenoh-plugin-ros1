# build instructions

Since zenoh-plugin-ros1 is not maintained by the zenoh team since 1.0.0-dev-83-g2b1cfac, Peixuan Shu continues to maintain it in his forked repository to keep it compatible with the latest zenoh releases. The build instructions are as follows:

# Support zenoh 1.7.2

I have modified `Cargo.toml` and replace the git branch `main` with the latest release tag `release/1.7.2` to ensure compatibility with the latest zenoh release. The original `xml-rpc = "0.0.12"` dependency is also updated to `0.1.0`. The `main.rs` and `lib.rs` in `zenoh-plugin-ros1/zenoh-plugin-ros1/src/` is modified to be compatible with the latest zenoh API. 

The build command is:

```bash
cargo +stable build --release
# There are several warnings but no errors.
```

In case of any build errors, ensure you have the latest version of Rust installed and try again.

```bash
# Remove the apt installed Rust if you have it
sudo apt remove rustc cargo

# Reinstall Rust using rustup
curl https://sh.rustup.rs -sSf | sh

cargo --version
# cargo 1.85.0 (d73d2caf9 2024-12-31)

rustc --version
# rustc 1.85.0 (4d91de4e4 2025-02-17)
```

If you modify the `Cargo.toml` file, make sure to update the `Cargo.lock` by:

```bash
# cd this repo and run
cargo update
```