# Hot patching from WSL2 to a Windows window

The compiler and every tool stay in WSL2. There is no Rust toolchain or MSVC on Windows. The app runs as a real Windows process.

This branch needs Dioxus CLI 0.7.10 with `dioxus-0.7.10.patch`. Unpatched, `dx` fails to link the fat binary for `x86_64-pc-windows-gnu`.

Needs WSL2 with mirrored networking, Rust 1.99.1 with the `x86_64-pc-windows-gnu` target, `llvm-dlltool`, and zig from pip.

```sh
python3 -m pip install --target /tmp/ziglang ziglang
rustup target add x86_64-pc-windows-gnu

git clone --depth 1 --branch v0.7.10 https://github.com/DioxusLabs/dioxus.git /tmp/dioxus
git -C /tmp/dioxus apply "$PWD/scripts/wsl-windows/dioxus-0.7.10.patch"
CARGO_TARGET_DIR=/tmp/dioxus-target cargo build --manifest-path /tmp/dioxus/Cargo.toml --locked -p dioxus-cli

export PATH="$PWD/scripts/wsl-windows:$PATH"
export ZIGLANG_DIR=/tmp/ziglang
export CARGO_TARGET_X86_64_PC_WINDOWS_GNU_LINKER="$PWD/scripts/wsl-windows/zig-cc"
export RUNE_DX=/tmp/dioxus-target/debug/dx
export RUNE_HOT_WSL_ROOT="\\\\wsl.localhost\\$WSL_DISTRO_NAME"
export WSLENV=RUNE_HOT:RUNE_HOT_WSL_ROOT:DIOXUS_DEVSERVER_IP:DIOXUS_DEVSERVER_PORT:DIOXUS_CLI_ENABLED:DIOXUS_BUILD_ID:DIOXUS_SESSION_CACHE_DIR:RUNE_DEV_GUARDED

cd scripts/wsl-windows/app
python3 ../../rune-dev --local --target x86_64-pc-windows-gnu --manifest-path Cargo.toml --patch 'src/*.rs'
```

Edit a string in `view` in `src/main.rs`. The open window changes and keeps its state.

If `dx` then reports `libdeps-*.a: unknown file type` on a later run, delete the empty `libdeps-*.a` and `rlibs-*.txt` from the target's `rune-hot` folder.
