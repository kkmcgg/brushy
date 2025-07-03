# Public Rust API

Below is a *human-written* overview of the most important public crates and their entry points. For the exhaustive, auto-generated documentation, run

```
cargo doc --workspace --no-deps --open
```

which will build HTML docs for every public item.

---

## `brush_render`
High-performance Gaussian-splat rasteriser built on [Burn](https://github.com/tracel-ai/burn) and `wgpu`.

| Item | Description |
|------|-------------|
| `gaussian_splats::Splats<B>` | Container of gaussians. Generic over a Burn backend `B`. Implements `render`, `num_splats`, `into_autodiff`, … |
| `camera::Camera` | Simple pin-hole camera with left-handed‐Y-up convention. |
| `burn_init_setup()` | Async helper that initialises a default WebGPU device. |
| `burn_init_device()` | Same, but takes an existing adapter/device/queue triple. |

```rust
use brush_render::{gaussian_splats::Splats, camera::Camera};
// See the minimal example in examples.md
```

---

## `brush_train`
Implements optimisation logic and loss functions to convert a posed image dataset into a splat cloud.

| Item | Description |
|------|-------------|
| `train::SplatTrainer` | Complete training loop with Adam optimiser, regularisation, and optional refine stages. |
| `config::TrainConfig` | All hyper-parameters – learning rates, regularisation weights, total steps, etc. |

```rust
let config = TrainConfig::new().with_total_steps(30_000);
let mut trainer = SplatTrainer::new(&config, &device);
let (splats, stats) = trainer.step(dt, iter, &batch, splats);
```

---

## `brush_ui`
Reusable egui components for displaying splats.

| Item | Description |
|------|-------------|
| `burn_texture::BurnTexture` | Wraps a Burn tensor in a GPU texture and exposes an `egui::TextureId`. |
| `ui_process::UiProcess` | High-level coordinator used by the desktop viewer. |
| `create_egui_options()` | Returns sane default `WgpuConfiguration` for egui-with-wgpu. |

---

## `brush_process`
End-to-end pipeline orchestration. Converts CLI arguments into streaming `ProcessMessage`s that can be consumed by the GUI or the CLI progress bar.

| Item | Description |
|------|-------------|
| `config::ProcessArgs` | Deserialisable config that mirrors the CLI. |
| `message::ProcessMessage` | Event stream (dataset loaded, train step, eval result, …). |

---

## `brush_cli`
Entry point for the binary published on crates.io. Uses the crates above but adds the `clap` interface and a TTY UI based on `indicatif`.

---

## `brush_wasm`
Bindings for the WebAssembly target. Exposes `wasm_app(canvas_name, url)` which spawns the viewer in a browser context.

---

## Helper crates

* `brush_vfs` – Abstract file-system + URL loader.
* `brush_dataset` – Dataset format handling, COLMAP/Nerfstudio IO.
* `brush_sort` – GPU radix sort kernels, also available standalone.
* `lpips` – Port of the LPIPS perceptual loss to Burn.
* … and several more low-level crates located in `/crates`.

---

## Versioning & SemVer

Each crate follows semantic versioning. Breaking API changes will result in a new **major** version on crates.io. All internal crates share the same version number.

---

## Feature flags

| Flag | Crate | Effect |
|------|-------|--------|
| `wasm` | `brush_render`, `brush_ui` | Enables WebAssembly-specific code paths. |
| `android` | `rrfd` | JNI bindings for Android. |

Activate with e.g. `brush_render = { version = "1", features = ["wasm"] }` in `Cargo.toml`.