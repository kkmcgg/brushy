# Brush – Comprehensive Documentation

Welcome to the official documentation of **Brush** – a cross-platform Gaussian-splatting engine written in Rust.

Use the table of contents below to quickly navigate to the section you are interested in:

1. [Getting Started](#getting-started)
2. [Command-line Interface](cli.md)
3. [Public Rust API](API.md)
4. [Examples & Tutorials](examples.md)
5. [Building for Every Platform](#building-for-every-platform)
6. [Contributing](#contributing)

---

## Getting Started

Brush can be used in three different ways:

1. **As a desktop application / viewer** – just run `cargo run --release` from the workspace root and load a dataset.
2. **As a command-line tool** – see the [CLI docs](cli.md) to automate training, rendering and evaluation.
3. **As a Rust library** – import the desired crates in your own project. All public items are listed in the [API reference](API.md).

If you simply want to have a look at Brush without compiling anything, try the [online demo](https://arthurbrussee.github.io/brush-demo).

---

## Building for Every Platform

* **Desktop (Windows, macOS, Linux)** – `cargo run --release`
* **WebAssembly** – follow the instructions in `brush_nextjs/README.md` or run `npm run dev` inside that directory.
* **Android** – see the step-by-step guide in the [root README](../README.md#android).

---

## Contributing

Contributions, issues and feature requests are welcome! Please open a pull request or start a discussion on [Discord](https://discord.gg/TbxJST2BbC).

---

© 2025 Brush Contributors – licensed under the Apache 2.0 License.