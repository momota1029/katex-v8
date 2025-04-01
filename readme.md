# katex-v8

A Rust library that utilizes KaTeX (v0.16.21) through the V8 engine to render LaTeX math expressions to HTML.

## Features

* **Fast Processing**: Rapid initialization and rendering using V8 snapshots
* **Single Instance**: Reuse of a single KaTeX instance to minimize loading delays (though not optimized for parallel processing)
* **Macro Support**: Collect and reuse macros defined with `\gdef` and similar commands (Note: depends on KaTeX v0.16.21 internal representation)
* **Caching Capability**: Cache V8 snapshots to the filesystem to reduce startup time

## Installation

Add this to your Cargo.toml:

```toml
[dependencies]
katex-v8 = "0.1.0"
```

## Usage

### Basic Example

```rust
use katex_v8::render;

fn main() {
    // KaTeX is initialized automatically on first call
    let html = render(r"E = mc^2").unwrap();
    println!("{}", html);
}
```

### Using Options and Macros

```rust
use katex_v8::{render_with_opts, Options, KatexOutput};
use std::collections::BTreeMap;
use std::borrow::Cow;

fn main() {
    let mut macros = BTreeMap::new();

    // Set custom options
    let options = Options {
        display_mode: true,
        output: KatexOutput::HtmlAndMathml,
        error_color: Cow::Borrowed("#ff0000"),
        ..Default::default()
    };

    // Render first equation (defining macros)
    let html1 = render_with_opts(
        r"\gdef\myvar{x} \myvar^2 + \myvar = 0",
        &options,
        &mut macros
    ).unwrap();
    println!("HTML 1: {}", html1);

    // Use previously defined macros in second equation
    let html2 = render_with_opts(
        r"\myvar^3",
        &options,
        &mut macros
    ).unwrap();
    println!("HTML 2: {}", html2);
}
```

### Setting Up Cache

```rust
use katex_v8::{set_cache, render};
use std::path::Path;

fn main() {
    // Set path to cache V8 snapshot
    set_cache(Path::new("./katex-cache"));

    // Subsequent renderings will be faster
    let html = render(r"E = mc^2").unwrap();
    println!("{}", html);
}
```

## Comparison with `katex-rs`

* **Macro collection and reuse**: Ability to reuse macros defined in equations in subsequent renderings (main differentiating feature)
* **Caching capability**: Fast initialization with V8 snapshots
* **Single-thread optimization**: Shared KaTeX instance in one worker thread (though not suitable for parallel processing)

Note that `katex-rs` supports more JavaScript engines (duktape, wasm-js, etc.), making it more versatile in that respect.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.