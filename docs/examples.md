# Examples & Tutorials

This section walks you through the most common tasks when using Brush as a **Rust library**. Every example can be compiled with

```
cargo run --package <crate-name> --example <example-name>
```

or explored directly in the `/examples` folder.

---

## 1. Minimal rendering example

```rust
use brush_render::{camera::Camera, gaussian_splats::Splats, burn_init_setup};
use glam::Vec3;

#[tokio::main]
async fn main() {
    // Initialise a device that works on any WebGPU backend.
    let device = burn_init_setup().await;

    // Create a single white gaussian in the centre of the world
    let mut splats = Splats::from_xyz_rgba(&[[0.0, 0.0, 0.0, 1.0, 1.0, 1.0, 1.0]], &device);

    // Simple pin-hole camera
    let cam = Camera::look_at(Vec3::new(0.0, 0.0, -3.0), Vec3::ZERO);

    // Render a 512×512 RGBA image (Float32 per channel, linear space)
    let (rgba, _aux) = splats.render(&cam, glam::uvec2(512, 512), Vec3::ZERO, None);
    rgba.to_file("out.exr").expect("failed to write file");
}
```

### What's happening?
1. `burn_init_setup()` creates a `WgpuDevice` and picks the best adapter.
2. `Splats::from_xyz_rgba` builds a `Splats` struct from an interleaved slice.
3. `render` projects the gaussians, sorts them on GPU, and rasterises them.

---

## 2. Train on a single image (2-D example)

A fully-fledged training loop with GUI is located in `examples/train-2d`. Run it with

```
cargo run -p train-2d --example train-2d
```

The example shows how to:

* Use `brush_train::SplatTrainer` for optimisation.
* Continuously refine the number of gaussians.
* Visualise training progress in realtime via `egui`.

---

## 3. WebAssembly / NextJS demo

The `brush_nextjs` folder contains a ready-to-go NextJS site that compiles the core engine to WebAssembly and runs entirely in the browser.

```
cd brush_nextjs
npm install
npm run dev
```

Supported browsers: **Chrome 135+** (Firefox/Safari soon).

---

## 4. Export gaussians to a `.ply` file

```rust
use brush_render::gaussian_splats::Splats;
use brush_dataset::splat_export::splat_to_ply;
use brush_render::burn_init_setup;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let device = burn_init_setup().await;

    // Imagine we already trained or created some splats
    let splats = Splats::from_xyz_rgba(
        &[[0.0, 0.0, 0.0, 1.0, 0.2, 0.8, 1.0]],
        &device,
    );

    // Write to the widely-supported gSplat (*.ply) format
    let ply_bytes = splat_to_ply(splats).await?;
    std::fs::write("scene.ply", ply_bytes)?;
    Ok(())
}
```

---

## 5. Embed Brush in an existing **egui** desktop app

```rust
use brush_render::{gaussian_splats::Splats, camera::Camera, burn_init_setup};
use brush_ui::burn_texture::BurnTexture;
use glam::{Vec3, Vec2, Quat};

struct MyViewer {
    cam: Camera,
    splats: Splats<brush_render::MainBackend>,
    tex: BurnTexture,
}

impl MyViewer {
    fn new(renderer: &eframe::wgpu::Renderer, device: &wgpu::Device, queue: &wgpu::Queue) -> Self {
        let burn_dev = brush_render::burn_init_device(
            renderer.adapter().clone(),
            device.clone(),
            queue.clone(),
        );

        let splats = Splats::from_xyz_rgba(&[[0.0, 0.0, 0.0, 1., 1., 1., 1.]], &burn_dev);
        let cam = Camera::look_at(Vec3::new(0.0, 0.0, -4.0), Vec3::ZERO);
        let tex = BurnTexture::new(renderer.clone(), device.clone(), queue.clone());
        Self { cam, splats, tex }
    }

    fn ui(&mut self, ui: &mut egui::Ui) {
        // Render on demand (e.g. every frame)
        let (img, _aux) = self
            .splats
            .render(&self.cam, glam::uvec2(512, 512), Vec3::ZERO, None);
        let tid = self.tex.update_texture(img);
        ui.image(tid, Vec2::splat(512.0));
    }
}
```

The helper type `BurnTexture` converts a Burn tensor into a GPU texture that `egui` can display. The heavy-lifting (projection, sorting, rasterisation) happens entirely on the GPU, so the viewer remains highly interactive.