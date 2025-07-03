# GUI Reference – Pre-Training Settings Window

When you click **Load ➜ File/Directory/URL** in the Brush desktop viewer, a *Settings* popup appears. This window lets you tweak **all hyper-parameters**, dataset options, and logging tools *before* training starts.  This document explains every control, its purpose, valid range, and default value so you can make informed decisions.

---

## 1. Training

| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| **Total steps** | 1 – 50 000 | **30 000** | Number of optimisation iterations. |
| **Max splats** | 1 000 000 – 10 000 000 | **10 000 000** | Upper bound for gaussians after all refine stages. |

### 1.1 Learning rates (expandable)

| Control | Range | Default | Notes |
|---------|-------|---------|-------|
| *Mean LR (start)* | 1 e-7 – 1 e-4 (log) | **4 e-5** | Initial LR for mean (`xyz`) parameters. |
| *Mean LR (end)* | 1 e-7 – 1 e-4 (log) | **4 e-7** | Final LR (linearly decays from start). |
| *Mean noise weight* | 1 e³ – 1 e⁵ | **1 e⁴** | Adds noise to low-opacity gaussians to encourage exploration. |
| *SH coeffs (dc)* | 1 e-4 – 1 e-2 (log) | **3 e-3** | LR for the DC colour of spherical harmonics. |
| *SH division (high orders)* | 1 – 50 | **20** | Higher-order SH LRs are divided by this factor. |
| *Opacity LR* | 1 e-3 – 1 e-1 (log) | **3 e-2** | LR for raw opacity values. |
| *Scale LR (start)* | 1 e-3 – 1 e-1 (log) | **1 e-2** | LR for log-scales. |
| *Scale LR (end)* | 1 e-4 – 1 e-2 (log) | **6 e-3** | Final LR (linear decay). |
| *Rotation LR* | 1 e-4 – 1 e-2 (log) | **1 e-3** | LR for quaternion rotation. |

### 1.2 Growth & Refinement (expandable)

| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| *Refine every* | 50 – 300 steps | **150** | Interval between densify / pruning passes. |
| *Growth grad threshold* | 1 e-4 – 1 e-3 (log) | **8.5 e-4** | Lower = faster growth. |
| *Growth selection frac.* | 1 % – 20 % | **10 %** | Fraction of candidates that are actually split. |
| *Growth stop iter* | 5 000 – 20 000 | **12 500** | Disable growth after this iteration. |

### 1.3 Losses (expandable)

| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| *SSIM weight* | 0 – 1 | **0.2** | Blend between SSIM (structure) and L1. |
| *Opacity loss weight* | 1 e-9 – 1 e-7 (log) | **1 e-8** | Regularises raw opacity. |
| *Alpha-match weight* | 0.01 – 1 | **0.1** | Strength of RGBA alpha match when source images include transparency. |

---

## 2. Model

| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| **SH Degree** | 0 – 4 | **3** | Spherical harmonics order – higher yields more colour detail at extra memory. |

---

## 3. Dataset

| Control | Range / Options | Default | Description |
|---------|-----------------|---------|-------------|
| **Max image resolution** | 32 – 2048 px (long edge) | **1920** | Down-scales larger images to save VRAM. |
| **Limit max frames** | Checkbox + 1 – 256 | *off* | Only load the first *N* frames. |
| **Split dataset for eval** | Checkbox + 2 – 32 | *off* | Uses every *N*-th frame as an evaluation set.  e.g. *1 out of 8 frames*. |
| **Subsample frames** | Checkbox + 2 – 20 | *off* | Load every 1/*N* image to accelerate training. |
| **Subsample points** | Checkbox + 2 – 20 | *off* | Load every 1/*N* SfM point from COLMAP. |

---

## 4. Process

| Control | Range / Options | Default | Description |
|---------|-----------------|---------|-------------|
| **Random seed** | Any u64 | **42** | Initialise RNG for reproducibility. |
| **Start at iteration** | 0 – 10 000 | **0** | Resume training from a checkpoint number (also increases export counter). |

### 4.1 Export (native builds only)

| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| *Export every* | 1 – 15 000 steps | **5 000** | Interval for saving `.ply` snapshots. |
| *Export path* | Text | `.` | Directory where PLYs are written. |
| *Export filename* | Text | `export_{iter}.ply` | `{iter}` is replaced by the step (zero-padded). |

### 4.2 Evaluate

| Control | Range / Options | Default | Description |
|---------|-----------------|---------|-------------|
| *Eval every* | 1 – 5 000 steps | **1 000** | Frequency of validation sweeps. |
| *Save eval images* | Checkbox | *off* | Dump rendered validation views to `export-path/`. |

---

## 5. Rerun (native desktop only)

[Rerun.io](https://rerun.io/) is an optional visualiser.  Controls are hidden on Web / Android builds.

| Control | Range / Options | Default | Description |
|---------|-----------------|---------|-------------|
| **Enable rerun** | Checkbox | *off* | Connect Brush to the Rerun viewer. |
| *Log train stats every* | 1 – 1 000 | **50** | How often to send basic metrics. |
| *Visualise splats* | Checkbox + 1 – 5 000 steps | *off* | Streams the full splat cloud (heavy). |
| *Max image log size* | 128 – 2 048 px | **512** | Down-scale images sent to Rerun. |

---

## 6. Load source

At the top of the panel you'll find three buttons:

1. **File** – Open a single `.ply`, `.zip` or dataset folder.  
2. **Directory** – Open a directory (disabled on web & Android).  
3. **URL** – Stream a remote `.ply` or dataset; enter the full HTTPS URL.

Once a source is selected, the **Start** button at the bottom becomes active.  Clicking it hides the popup and kicks off the training pipeline with the parameters you chose.

---

### Tips & Shortcuts

* **Reset to defaults:** Close the popup without pressing *Start*; reopening loads default config.  
* **Numeric fields:** Hold **Ctrl** while dragging sliders for finer increments.  
* **Log-scale sliders:** Appear in scientific notation (`1e-4`); useful for tiny learning rates and loss weights.

---

That's every knob available in the pre-training GUI.  Combine them to balance speed, memory, and quality for your specific dataset!