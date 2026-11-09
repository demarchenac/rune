# Reproducing hot patching from WSL2 to Windows

Rune cross-compiles to `x86_64-pc-windows-gnu` from WSL2 with no Rust or MSVC tools on Windows. A normal build works. A hot-patch build fails while `dx` 0.7.10 links the fat binary.

Needs WSL2, Rust 1.99.1 with the `x86_64-pc-windows-gnu` target, `llvm-dlltool`, Dioxus CLI 0.7.10 and zig from pip:

```sh
python3 -m pip install --target /tmp/ziglang ziglang
rustup target add x86_64-pc-windows-gnu
cargo install dioxus-cli --version 0.7.10 --locked --root /tmp/dx

export PATH="$PWD/scripts/wsl-windows:$PATH"
export ZIGLANG_DIR=/tmp/ziglang
export CARGO_TARGET_X86_64_PC_WINDOWS_GNU_LINKER="$PWD/scripts/wsl-windows/zig-cc"
export RUNE_DX=/tmp/dx/bin/dx

cd scripts/wsl-windows/app
python3 ../../rune-dev --local --target x86_64-pc-windows-gnu --manifest-path Cargo.toml --patch 'src/*.rs'
```

Expected failure:

```text
ERROR Failed to generate fat binary: error: /OUT:.../wsl-hot-app.exe: unrecognized file extension
```

`zig-cc` drops `-Wl,--export-dynamic-symbol,main`. Without that, the build stops one step earlier, on `unsupported linker arg: --export-dynamic-symbol`.
