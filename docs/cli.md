# Brush CLI

The command-line interface (CLI) lets you train, evaluate, and export Gaussian-splatting scenes without running the desktop viewer. The binary is called `brush` when installed from crates.io, or you can run it directly from the workspace root with `cargo run --release --`.

```
brush --help
```

### Global options

| Flag | Default | Description |
|------|---------|-------------|
| `--with-viewer` | `true` (desktop) / `false` (headless) | Spawn the egui viewer alongside the process. Automatically disabled when `PATH_OR_URL` is provided. |
| `PATH_OR_URL`   | *(optional)* | A local path or an `https://` URL pointing to a dataset or `.ply` file. |
| All flags from **ProcessArgs** | see below | Fine-grained control over training, refinement, and evaluation. |

<details>
<summary>ProcessArgs (advanced)</summary>

| Flag | Default | Description |
|------|---------|-------------|
| `--total-steps <N>` | 30_000 | Number of optimisation iterations. |
| `--batch-size <N>`  | 1 | Number of views per step. |
| `--eval-every <N>`  | 2_000 | Evaluate the current model every *N* steps. |
| `--mask-alpha` | off | Respect the alpha channel of PNGs as a segmentation mask. |
| … | … | See `crates/brush-process/src/config.rs` for the full list. |

</details>

---

## Typical workflows

### 1. Train a dataset and inspect it live

```
# Assumes a folder formatted like Nerfstudio or COLMAP
brush /path/to/dataset --with-viewer
```

### 2. Headless training on a server

```
brush /data/farm/garden --with-viewer=false --total-steps 60000 \
      --batch-size 4 --eval-every 5000 >log.txt 2>&1
```

### 3. Render an existing `.ply` / `.compressed.ply`

```
brush assets/bonsai.ply --with-viewer
```

### 4. Stream data from the web (viewer only)

```
brush "https://example.com/bicycle.compressed.ply" --with-viewer
```

---

## Installation

```
cargo install brush-cli
```

The CLI is completely self-contained; it only requires a Vulkan / Metal / D3D12 capable GPU driver.