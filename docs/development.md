# Developing die-draw beside DIE

die-draw's only DIE crate is a dev-dependency: `die-test-cap`, the memory limit inside every
test binary, from DIE's `main`.

## The dev config

To build against a DIE checkout instead, and to give the tests DIE's fonts, copy the dev
config. Its paths are relative, for a DIE checkout beside this one (`../die`), and cargo reads
them from this repository's root, so change them only if your checkout is elsewhere:

```sh
mkdir -p .cargo
cp dev-config.example.toml .cargo/config.toml   # gitignored, so local paths stay local
```

It patches `die-test-cap` to `../die/tools/test-cap` and sets `DIE_FONT_DIR` to `../die/fonts`
(`cargo xtask fonts` in DIE's checkout fetches them).

An extension developed against this checkout of die-draw patches it the same way in its own
dev config:

```toml
[patch."https://github.com/0xDEADBEE7/die-draw"]
die-draw = { path = "../die-draw" }
```

## The checks

**The tests need `DIE_FONT_DIR`**: without it they pass but skip DIE's Ioskeley Mono among
the families they set text in. Run the checks with the Rust that `rust-toolchain.toml` pins
(1.96.0):

```sh
cargo fmt --check
cargo clippy --all-targets
DIE_UNSEEN=1 cargo test
```

No test puts anything on screen, touches the real pasteboard or opens a file.
