# Using die-draw

There's nothing to install: die-draw isn't an extension, and
[die-config](https://github.com/0xDEADBEE7/die-config) lists none of it. An extension that
draws depends on it:

```toml
[dependencies]
die-draw = { git = "https://github.com/0xDEADBEE7/die-draw", branch = "main" }
```

Every repository names it on `branch = "main"`, never a `rev`, so one build resolves one
die-draw however many extensions use it, and each repository's `Cargo.lock` pins the commit.

See [What it draws](drawing.md) for the API in brief and [Fonts](fonts.md) for where it
finds DIE's fonts.
