# Setup & Codebase

## Overview
The primary codebase is located in the `motion-capture` repository, which operates independently from upstream EasyMocap. It contains the following core components grouped by role:

### Pipeline Stages
- **`scripts/preprocess/`**: Dataset preparation (sync gate $\rightarrow$ match $\rightarrow$ extract $\rightarrow$ QC previews).
- **`apps/calibration/`**: Chessboard detection $\rightarrow$ intrinsics $\rightarrow$ extrinsics $\rightarrow$ COLMAP BA.
- **`apps/preprocess/` & `apps/demo/`**: Human pose (2D keypoints $\rightarrow$ triangulation $\rightarrow$ SMPL-X).
- **`apps/handfix_hamer/`**: HaMeR-based hand de-twisting / place / smooth after the SMPL-X fit.
- **`apps/reconstruction/`**: Object (cow) MVS pipeline (sparse SfM $\rightarrow$ dense $\rightarrow$ visual-hull filter).

### Support Components
- **`easymocap/`**: The core python package (loaders, SMPL fitting, camera I/O). Imported directly as `easymocap`.
- **`config/`**: Configuration yaml files for pipelines and per-rig visualization presets (`config/viz/`).
- **`scripts/`**: SLURM launchers, demo production, dataset publication, and post-hoc utilities.
- **`data/`**: Model weights + sample datasets (bulk data lives on cluster storage).
- **`doc/`**: Report sources (session summaries, sync evals), dated research notes, and skeleton reference images.
- **`share/`**: Shareable summaries (two-page PDF overviews).
- **`tests/`**: Unit tests (`pytest tests/`).

> **Note on Fork Independence**: This repository began as an EasyMocap fork but no longer tracks upstream. The upstream config-driven framework and unused apps were removed to maintain a lean, specialized pipeline.

---

## Environment & HPC Setup
This project was largely executed on the Digital Research Alliance of Canada (Alliance) cluster. 

Key notes for reproducing on the cluster:
- **Python**: Use `module load python` and standard virtual environments (`venv`). Python 3.8+, PyTorch, and OpenCV are required.
- **External Dependencies**: You will need MMPose, COLMAP (built with CUDA), SMPL-X model files, and Segment Anything 3 (SAM3).
- **Dense MVS Backends**: Alternative dense backends (ACMMP, DPE-MVS, DVP-MVS) require specific Alliance build patches.
- **Slurm Gotchas**: Beware of Slurm gotchas when requesting GPUs (e.g., specifying the exact memory and GPU type). Custom CUDA extensions might require careful Wheelhouse configurations.
- **Camera Synchronization**: External capture-time synchronization relies on **Argus** (a separate in-house repository).

---

## Task Guide (Running the Pipeline)

Here is a quick reference for running the various pipelines:

| Task | Entry Script / Path |
|---|---|
| **Preprocess new capture session** | `run_sync_pipeline.py <raw> --threshold 16ms` |
| **Camera Calibration** | `detect_calibration_board.py <clip> --mode charuco` |
| **Reconstruct Cow (Object MVS)** | `preprocess_segment_sam3.py <obj> --frames a:b:s` |
| **Fit Human Pose (SMPL-X)** | `extract_keypoints.py <data> --mode mmpose` |
| **Fix Twisted SMPL-X Hands** | See extract $\rightarrow$ fuse $\rightarrow$ place $\rightarrow$ smooth flow in `apps/handfix_hamer/` |
| **Render 3D Results** | `export_scene_ply` $\rightarrow$ headless Blender |
| **Produce Demo Videos** | `launch_cow_1_demo.sh` in `scripts/slurm/demo_production/` |
| **Publish Dataset** | `build_deliverable.sh` in `scripts/publish/dataset/` |

### Detailed Object Reconstruction Commands
Canonical commands for the cow reconstruction pipeline:

```bash
# 1. Foreground masks
python apps/reconstruction/preprocess_segment_sam3.py <obj_path> --frames 0:1426:5

# 2. Per-frame MVS (SLURM array)
./scripts/slurm/launch_4d_array.sh <obj_path> 0:1426:5

# 3. Visual-hull post-filter
python -m apps.reconstruction.tools.visual_hull_filter_pcd \
    <fused.ply> <fused_filtered.ply> \
    --sparse <stage_a>/work/frame_X/sparse/0 --masks <dataset>/masks \
    --frame <frame_id> --erode 8 --min-cams 9
```
*(Resource sizing: ~10–15 min per frame on an H100 MIG 3g.40gb node for the 11-camera 4K cow rig).*
