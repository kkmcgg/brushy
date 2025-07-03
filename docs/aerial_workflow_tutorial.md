# End-to-End Tutorial – Aerial Images ➜ COLMAP ➜ Brush ➜ PLY

This guide walks you through **every step** required to turn a set of drone photographs into a 3-D Gaussian-splat model (gSplat) using *Brush* and finally export it as a `.ply` file that other viewers can load.

| Phase | Goal | Tools |
|-------|------|-------|
| 1. Capture | Take overlapping photos | Drone / DSLR |
| 2. SfM / MVS | Recover camera poses & sparse points | [COLMAP](https://colmap.github.io/) |
| 3. Train | Optimise a Gaussian-splat representation | `brush` CLI |
| 4. Export | Save to gSplat-compatible file | Built-in exporter |

---

## 1. Capturing aerial imagery

* **Overlap:** Aim for **70 % forward** and **60 % side** overlap so that COLMAP can match features reliably.
* **Exposure:** Shoot RAW or high-quality JPEG; avoid auto-exposure flicker.
* **Numbering:** Name files sequentially (`IMG_0001.JPG`, …) – this makes life easier later.
* **Masks (optional):** If parts of the frame should be ignored (e.g. propellers), place **PNG masks** with the *same basename* in a folder called `masks/` (next to `images/`).

Resulting tree:
```
my_flight/
└── images/
    ├── img0001.jpg
    ├── img0002.jpg
    └── …
```

---

## 2. Structure-from-Motion with COLMAP

You can run COLMAP through its GUI **or** via the command-line. Below is a *fully automated* CLI pipeline that works well for small/medium datasets (change paths accordingly):

```bash
# 2.1 Initialise the database & extract features
echo "==> Feature extraction"
colmap feature_extractor \
    --database_path colmap.db \
    --image_path images \
    --ImageReader.single_camera 1

# 2.2 Match features (exhaustive matcher is robust but O(n²));
# for >2 000 images use sequential/approximate matching instead.

echo "==> Matching"
colmap exhaustive_matcher \
    --database_path colmap.db

# 2.3 Sparse reconstruction (SfM)
echo "==> Mapping"
colmap mapper \
    --database_path colmap.db \
    --image_path images \
    --output_path sparse

# 2.4 (Optional) Undistort & dense MVS – not needed by Brush
# colmap image_undistorter --image_path images --input_path sparse/0 --output_path dense
```

Key output files are generated in `sparse/0/`:

```
my_flight/
├── images/
│   └── …
└── sparse/
    └── 0/
        ├── cameras.txt  (or .bin)
        ├── images.txt   (or .bin)
        └── points3D.txt (or .bin – optional)
```

Brush understands **both `.txt` and `.bin`** variants. If COLMAP produced multiple models (`sparse/1`, `sparse/2`, …) pick the best one and rename/keep it as `sparse/0`.

### Alternatives & tweaks

1. **GUI workflow:** Use *`Reconstruction ➜ Automatic Reconstruction`* – it performs all four stages for you.
2. **Sequential matcher:** `colmap sequential_matcher` is faster when images are captured in order (e.g. drone orbits).
3. **Mask support:** Set `--ImageReader.mask_path mask.jpg` if you have per-image masks outside of Brush; otherwise rely on Brush's mask convention (see next section).

---

## 3. Training a gSplat with Brush

Install the CLI (once):
```bash
cargo install brush-cli  # or build from source with `cargo run --release -- <args>`
```

Now point Brush at the folder **containing `sparse/0/` and `images/`**:

```bash
brush my_flight \
      --total-steps 30000      \  # optimisation iterations
      --eval-every 2000        \  # run validation every N steps
      --export-every 5000      \  # save spline cloud periodically
      --export-path out        \  # target directory for PLYs
      --with-viewer            # launch interactive viewer (omit on servers)
```

Important command-line knobs:

| Flag | Purpose | Default |
|------|---------|---------|
| `--subsample-frames N` | Use every *N*-th image to speed up training. | off |
| `--subsample-points N` | Use every *N*-th COLMAP point for initialisation. | off |
| `--sh-degree D`        | Spherical-harmonics order (0 = colour only; 3≈NeRF quality). | 3 |
| `--mask-alpha`         | Treat alpha channel of PNGs as segmentation mask. | off |
| `--export-name TEMPLATE` | Filename pattern (`export_{iter}.ply`) | `export_{iter}.ply` |
| `--eval-save-to-disk`    | Store rendered validation images under `export-path/` | off |

All flags come from `ProcessArgs`; run `brush --help` for a full reference.

---

## 4. Automatic PLY export

During training Brush writes snapshots to disk every `--export-every` steps **and** at the final iteration. Files are placed in `--export-path` and are named with the pattern from `--export-name` where `{iter}` is replaced by the current step number (zero-padded to the total number of digits).

Example directory after training 30 000 steps:
```
out/
├── export_005000.ply
├── export_010000.ply
├── export_015000.ply
├── export_020000.ply
├── export_025000.ply
└── export_030000.ply  # final result
```

Each `.ply` follows the [Inria gSplat format](https://github.com/graphdeco-inria/gaussian-splatting) and can be viewed in:

* The **Brush** desktop viewer (`brush file.ply --with-viewer`)
* [gsplat.studio](https://gsplat.studio/) or other third-party gSplat viewers
* Blender with the [gSplat addon](https://github.com/0x00019913/BlenderGaussianSplats)

### Manual export

If you paused training early or want a custom export interval, run the small Rust snippet from the [examples](examples.md#4-export-gaussians-to-a-ply-file) or use the `brush-dataset` crate programmatically.

---

## File & folder conventions recap

| Path | Required | Description |
|------|----------|-------------|
| `images/`                | ✔ | All input photographs (JPEG, PNG, …). |
| `masks/`                 | ✖ | **Optional.** PNGs with the *same basename*; alpha channel determines transparency. |
| `sparse/0/`              | ✔ | COLMAP model (either `*.txt` or `*.bin`). |
| `dense/`                 | ✖ | Ignored by Brush (produced by COLMAP's MVS). |
| Any `.ply` in root       | ✖ | If present, is used as *initial* point cloud. |

Brush performs a **recursive search** for the required files, so deeper nesting (zip archives, tarballs, web-served datasets) also works as long as the names stay intact.

---

## What's next?

* **Refinement:** Increase `--total-steps` or run with a higher `--sh-degree` for more detail.
* **Compression:** Post-process the `.ply` with the `gsplat --compress` utility to shrink file size by 10×.
* **Web demo:** Convert the final PLY to a binary blob and host it with the `brush_nextjs` viewer for instant sharing.

Happy splatting! :sparkles: