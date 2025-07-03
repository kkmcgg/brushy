# Strategy Document – Multi-Source Data Fusion for Brush

This note outlines **three advanced extensions** to Brush's current pipeline and proposes concrete implementation paths.

1. **LiDAR-assisted initial splats** – fuse dense point clouds with COLMAP's sparse reconstruction.  
2. **Thermal imagery integration** – train joint RGB-T splats.  
3. **Spatial focus & constraints** – guide optimisation using DEMs, control points, or user-defined areas of interest.

Each section lists motivations, file-format expectations, algorithmic changes, and required code touch-points.

---

## 1. LiDAR-Assisted Initial Splats

### 1.1 Motivation
* Photogrammetry struggles in texture-less areas (water, asphalt, vegetation canopies).  
* LiDAR offers precise geometry—even in low-texture regions.  
* Combining both yields better coverage and faster convergence.

### 1.2 Data flow
```
LiDAR (LAS/LAZ) → pre-filter → PLY → Brush Loader
                      ↘ (optional ICP) ↗
COLMAP sparse            alignment      ┘
```

### 1.3 Pre-processing steps
1. **Filter & down-sample** LiDAR:  
   `las2las -i in.laz -o lidar.ply --thin 0.1` (PDAL, CloudCompare,…)
2. **Coordinate alignment** (if necessary):  
   *Option A* – run ICP between LiDAR ↔ COLMAP points (Open3D).  
   *Option B* – exploit RTK/PPK GNSS metadata if cameras and LiDAR share a geodetic frame.
3. **Colourisation** (optional):  
   Copy RGB from overlapping photos (`pdal colorization`).  Brush will fall back to SH DC=white if absent.

### 1.4 Integration paths
1. **Direct replacement**  
   • Drop `points3D.*` from `sparse/0/`.  
   • Place `lidar_init.ply` at dataset root ➜ Brush automatically uses it as **initial splat cloud** (see loader logic in `formats::load_dataset`).
2. **Hybrid augmentation**  
   • Keep COLMAP points for texture-rich regions.  
   • Merge LiDAR PLY; ensure no duplicate IDs.  
   • Implement `merge_point_clouds()` utility in `brush_dataset` that concatenates positions & colours.

### 1.5 Code changes
* `brush_dataset::splat_import` – extend PLY parser to respect `classification` column; ignore 'noise' returns.  
* `RandomSplatsConfig` – add flag `use_external_init`.  
* UI/CLI – new checkbox `--use-lidar-init`.

---

## 2. RGB + Thermal (RGB-T) Training

### 2.1 Motivation
* Thermal cameras reveal heat signatures invisible to RGB.  
* Useful for inspection, search-and-rescue, agriculture.

### 2.2 Data acquisition
* Use a **dual-sensor rig** with a factory-calibrated extrinsic matrix *(R, t)* between RGB ↔ T.  
* Trigger both shutters simultaneously or log timestamps for time-sync.

### 2.3 File-format proposal
```
my_dataset/
├── images_rgb/
│   ├── frame_0001.jpg
│   └── …
├── images_t/
│   ├── frame_0001.png   # 16-bit, linear temp values or emissivity-mapped 8-bit
│   └── …
└── thermo_extrinsics.json  # 4×4 matrices per frame OR global rig file
```

### 2.4 Renderer & data-model changes
1. **Multi-spectral SH**  
   Extend `gaussian_splats::Splats` to hold an **N×C** matrix where C ≥ 4 (RGB + Thermal).  
   Thermal can be treated as an extra DC coefficient (no need for SH order >0).
2. **Loss function**  
   • Keep existing RGB SSIM/L1 loss.  
   • Add **L1/Huber** between predicted T and ground truth temp map.  
   • Weight via new hyper-parameter `thermal_weight`.
3. **Camera model**  
   • Thermal lens has different intrinsics; add `CameraSpec::Thermal` variant.

### 2.5 Training loop adjustments
* `SceneLoader` – emit batches containing `(rgb_img, thermal_img)` pairs.  
* `SplatTrainer::step()` – compute compounded loss.

### 2.6 UI/CLI
* New flag `--with-thermal` (bool) and slider `--thermal-weight` (0-1).

---

## 3. Spatial Focus & External Constraints

### 3.1 Use-cases
* Prioritise urban zones in large-scale scenes.  
* Constrain splats within DEM elevation range.  
* Enforce accuracy near survey control points.

### 3.2 Input conventions
```
constraints/
├── focus_mask.tif       # GeoTIFF, float 0-1 per pixel in ground plane
├── dem.tif              # Elevation raster (EPSG:xyz)
└── gcp.csv              # x,y,z,weight for control points
```

### 3.3 Algorithmic hooks
1. **Sampling bias**  
   • `SceneLoader::next_batch()` – weight image probability by average focus value.  
2. **Loss regularisation**  
   • Add term `w_gcp * ||splat_pos - gcp||²` for nearest N splats.  
   • Clamp `z` (height) using `dem_min/max ± margin`.
3. **Growth pruning**  
   • Override `growth_select_fraction` locally: higher inside focus mask.

### 3.4 Implementation steps
* Create `crate brush_constraints` handling raster & CSV parsing (GDAL, `csv` crate).  
* Expose `ConstraintsConfig` inside `ProcessArgs`.  
* Modify `optimizer.rs` to read per-splat weights.

### 3.5 GUI
* Extra panel tab "Focus / Constraints" with file pickers and sliders.

---

## Milestone Roadmap
| Sprint | Deliverable |
|--------|-------------|
| **S1** | LiDAR PLY import & hybrid merge (CLI flag + docs) |
| **S2** | Thermal channel in Splats, loss term, dataset loader |
| **S3** | Constraint crate, DEM z-clamp, GCP loss |
| **S4** | GUI/CLI polish, public examples, integration tests |

---

### Open Questions & Risks
1. **Memory footprint** – extra channels increase tensor size; consider storing temperature in half precision.  
2. **Alignment errors** – inaccurate LiDAR ↔ RGB alignment can hurt convergence; maybe run pose optimisation jointly.  
3. **DEM resolution** – coarse grids may over-constrain; allow per-splat sigma on elevation.

---

*Prepared for Brush Contributors • v0.1 – July 2025*