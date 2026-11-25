# die-draw

Shared drawing for [DIE](https://github.com/0xDEADBEE7/die)'s extensions: text in the
system's fonts, pixels and PNGs.

- **Sets text** in the system's fonts, kerned and tracked like the Mac's own.
- **Lines up numbers** with tabular figures.
- **Cuts a line to fit** a width, ending in `…`.
- **Draws pixels**: text, pictures and rectangles, clipped, on RGBA8 buffers.
- **Reads and writes PNG**, and builds the command to copy one or open a file.

It's a library with no keys or commands, and it needs none of DIE's crates.
[die-markdown](https://github.com/0xDEADBEE7/die-markdown) and
[die-fig](https://github.com/0xDEADBEE7/die-fig) draw with it.

```rust
use die_draw::{Book, Pixels, Rgb, TextLook, Weight};

let mut book = Book::load(None)?;
let mut pass = book.pass();
let look = TextLook::new(Weight::Medium, 13.0).tabular();
let line = pass.set_within("Revenue 1,204,551", look, 2.0, 240.0);
let mut bytes = vec![0; 240 * 40 * 4];
let mut px = Pixels::new(&mut bytes, 240, 40).unwrap();
let (clip, baseline) = (px.whole(), line.extent.baseline_in(0.0, 40.0));
pass.draw(&line, &mut px, 0.0, baseline, Rgb(230, 230, 230), 255, clip);
```

## Use it

An extension that draws depends on it:

```toml
[dependencies]
die-draw = { git = "https://github.com/0xDEADBEE7/die-draw", branch = "main" }
```

## Docs

- [What it draws](docs/drawing.md): fonts, figures, pixels, PNG, the pasteboard
- [Fonts](docs/fonts.md): where die-draw finds DIE's fonts
- [Using it](docs/usage.md): the dependency and why it's on `main`
- [Development](docs/development.md): building beside DIE and the checks

## License

MIT or Apache-2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
