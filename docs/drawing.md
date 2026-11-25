# What die-draw draws

The drawing DIE's extensions share: the system's fonts set as lines of text, a premultiplied
RGBA8 blit with clipping, PNG in and out, and the command lines that put a PNG on the
pasteboard or open a file. It's a library, with no keys and no commands of its own, and it
depends on none of DIE's crates, so any extension may use it. It was lifted from
[die-markdown](https://github.com/0xDEADBEE7/die-markdown)'s typesetter once
[die-fig](https://github.com/0xDEADBEE7/die-fig) needed the same text.

## Text in the system's fonts

On a Mac the text is San Francisco (`SFNS.ttf`), at its weight (400, 510 or 590) and its
optical size at the size asked for, tracked by its `trak` table and kerned by its GPOS pairs,
which reproduces CoreText's advances; the monospace is SF Mono.

Without those it takes the first text family and the first monospace the machine has:
Helvetica Neue, Helvetica or Arial, and Menlo; DejaVu, Noto or Liberation on Linux; DIE's own
Ioskeley Mono last of both. A character a face lacks comes from the regular face, then the
symbol faces.

One glyph a character: there's no shaper, so no ligatures, no combining marks placed by a
shaper, and no right-to-left text. Glyphs are grown a little as a Mac's font smoothing grows
them.

## Tabular figures

Where a column of numbers should line up, each digit is the face's `tnum` glyph with no
kerning either side, or centred in the widest digit's width in a face with none.

## A line cut to fit

A line too long for a width becomes its longest start that fits, followed by `…`.

## Pixels

Coverage in an ink at an alpha, pictures over what's there, and filled rectangles, on DIE's
premultiplied RGBA8 buffers or tiny-skia's, clipped to a rectangle.

Sizes are logical (CSS px, points) with a scale beside them; what's drawn is physical.

## PNG

Read into premultiplied RGBA8 (refusing a picture past 256 MiB before making room for it),
written from straight RGBA8.

## The pasteboard and the opener

die-draw builds the command that puts a PNG on the pasteboard (`osascript` on macOS,
`wl-copy` or `xclip` on Linux, whichever is on the PATH) and the one that opens a file
(`open` or `xdg-open`). The caller runs it, in a job.

## An example

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
