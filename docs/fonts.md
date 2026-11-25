# Where DIE's own fonts are

`Book::load(Some(dir))` takes them from `dir`.

`Book::load(None)` finds them as DIE does: `$DIE_FONT_DIR`, else the app bundle's
`Contents/Resources/fonts` when the executable is inside `DIE.app`.

Outside DIE's checkout die-draw can't find DIE's `fonts/` from its own manifest, as
die-render does, so its last resort is the `$DIE_FONT_DIR` it was built with (a dev config's
`[env]` sets it; see [Development](development.md)), else `fonts/` in the working directory.

The system's fonts are found by looking, at run time, wherever they are.
