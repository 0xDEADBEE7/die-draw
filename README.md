# die-draw

The drawing [DIE](https://github.com/0xDEADBEE7/die)'s extensions share: the system's fonts
set as lines of text, a premultiplied RGBA8 blit with clipping, PNG in and out, and the command
lines that put a PNG on the pasteboard or open a file. It's a library, with no keys and no
commands of its own, and it depends on none of DIE's crates, so any extension may use it. It
was lifted from die-markdown's typesetter once die-fig needed the same text. die-markdown and
die-fig draw with it.

## What it draws

- **Text in the system's fonts.** On a Mac the text is San Francisco (`SFNS.ttf`), at its
  weight (400, 510 or 590) and its optical size at the size asked for, tracked by its `trak`
  table and kerned by its GPOS pairs, which reproduces CoreText's advances; the monospace is SF
  Mono. Without those it takes the first text family and the first monospace the machine has:
  Helvetica Neue, Helvetica or Arial, and Menlo; DejaVu, Noto or Liberation on Linux; DIE's own
  Ioskeley Mono last of both. A character a face lacks comes from the regular face, then the
  symbol faces. One glyph a character: no shaper, so no ligatures, combining marks placed by a shaper,
  or right-to-left text. Glyphs are grown a little as a Mac's font smoothing grows them.
- **Tabular figures**, where a column of numbers should line up: each digit in the face's `tnum`
  glyph with no kerning either side, or centred in the widest digit's width in a face with none.
- **A line cut to fit** a width, its longest start that fits followed by `…`.
- **Pixels**: coverage in an ink at an alpha, pictures over what's there and filled rectangles,
  on DIE's premultiplied RGBA8 buffers or tiny-skia's, clipped to a rectangle.
- **PNG**: read into premultiplied RGBA8 (refusing a picture past 256 MiB before making room
  for it), written from straight RGBA8.
- **The pasteboard and the opener**: the command that puts a PNG on the pasteboard
  (`osascript` on macOS, `wl-copy` or `xclip` on Linux, whichever is on the PATH) and the one
  that opens a file (`open` or `xdg-open`). The caller runs it, in a job.

Sizes are logical (CSS px, points) with a scale beside them; what's drawn is physical.

```rust
use die_draw::{Book, Pixels, Rgb, TextLook, Weight};

let mut book = Book::load(None)?; // the fonts, DIE's own where die-draw finds them
let mut pass = book.pass();
let look = TextLook::new(Weight::Medium, 13.0).tabular();
let line = pass.set_within("Revenue 1,204,551", look, 2.0, 240.0);

let (w, h) = (240, 40);
let mut bytes = vec![0; w * h * 4];
let mut px = Pixels::new(&mut bytes, w as u32, h as u32).unwrap();
let clip = px.whole();
let baseline = line.extent.baseline_in(0.0, h as f32);
pass.draw(&line, &mut px, 0.0, baseline, Rgb(230, 230, 230), 255, clip);
```

### Where DIE's own fonts are

`Book::load(Some(dir))` takes them from `dir`. `Book::load(None)` finds them as DIE does:
`$DIE_FONT_DIR`, else the app bundle's `Contents/Resources/fonts` when the executable is inside
`DIE.app`. Outside DIE's checkout die-draw can't find DIE's `fonts/` from its own manifest, as
die-render does, so its last resort is the `$DIE_FONT_DIR` it was built with (a dev config's
`[env]` sets it, below), else `fonts/` in the working directory. The system's fonts are found
by looking, at run time, wherever they are.

## Installing it in die-config

There's nothing to install: die-draw isn't an extension, and die-config lists none of it. An
extension that draws depends on it:

```toml
[dependencies]
die-draw = { git = "https://github.com/0xDEADBEE7/die-draw", branch = "main" }
```

Every repository names it on `branch = "main"`, never a `rev`, so one build resolves one
die-draw however many extensions use it, and each repository's `Cargo.lock` pins the commit.

## Developing it beside DIE

die-draw's only DIE crate is a dev-dependency: `die-test-cap`, the memory limit inside every
test binary, from DIE's `main`. To build against a DIE checkout instead, and to give the tests
DIE's fonts, copy the dev config and write its `~` out as your home directory (cargo doesn't
expand it):

```sh
mkdir -p .cargo
cp dev-config.example.toml .cargo/config.toml   # gitignored, so local paths stay local
```

It patches `die-test-cap` to `/abs/path/to/die/tools/test-cap` and sets `DIE_FONT_DIR` to
`/abs/path/to/die/fonts` (`cargo xtask fonts` in DIE's checkout fetches them). An extension
developed against this checkout of die-draw patches it the same way in its own dev config:

```toml
[patch."https://github.com/0xDEADBEE7/die-draw"]
die-draw = { path = "/abs/path/to/die-draw" }
```

**The tests need `DIE_FONT_DIR`**: without it they pass but skip DIE's Ioskeley Mono among
the families they set text in. The checks, with the Rust that `rust-toolchain.toml` pins (1.96.0):

```sh
cargo fmt --check
cargo clippy --all-targets
DIE_UNSEEN=1 cargo test
```

No test puts anything on screen, touches the real pasteboard or opens a file.

## License

MIT or Apache-2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
