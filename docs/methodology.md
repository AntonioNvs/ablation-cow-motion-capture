# Methodology

This document provides a comprehensive overview of the end-to-end pipeline constructed in this project. While the primary focus revolves around the 3D reconstruction of Holstein dairy cows without predefined body models, the framework incorporates an extensive multi-stage process from synchronization to post-processing.

---

## Stage 1: Dataset Preprocessing & Sync

Before reconstruction can occur, raw multi-view video feeds must be temporally aligned and prepared:

- **Camera Synchronization**: The multi-camera rig is synchronized using the **Argus** framework (developed in-house) during the capture phase. This ensures that frames across all 11 cameras in the rig correspond to the exact same temporal instance.
- **Data Preparation**: The synchronized video feeds are extracted, matched, and processed to create quality-controlled (QC) previews of the dataset.

---

## Stage 2: Camera Calibration

Accurate multi-view geometry relies on precise camera calibration.

- **Chessboard Detection**: The pipeline utilizes ChArUco or standard chessboard patterns to establish the initial camera intrinsics (focal length, principal point) and extrinsics (rotation and translation relative to the world origin).
- **COLMAP Bundle Adjustment (BA)**: To refine these initial estimates, COLMAP's Bundle Adjustment is applied. This step jointly optimizes the camera poses and 3D points across the entire 11-camera rig, minimizing the global reprojection error and ensuring a robust geometric foundation for the subsequent stages.

---

## Stage 3.1: Human Pose Estimation & Refinement

While the primary research subject is dairy cows, the pipeline maintains full support for human pose triangulation and SMPL-X mesh recovery:

- **2D Keypoint Extraction**: 2D joints are extracted using **MMPose** (with fallback support for OpenPose or HRNet).
- **Triangulation & SMPL-X**: The 2D keypoints are triangulated into 3D space, and the SMPL-X parametric body model is fitted to the triangulated skeleton.
- **HaMeR Hand Refinement**: Because standard triangulation-driven SMPL-X hands often fail on this rig (resulting in twisted or mislocated hands due to the ~69 pixel resolution of the hands), a custom **HaMeR** (Hand Mesh Recovery) refinement pipeline is deployed. It:
  1. Fuses per-camera HaMeR MANO predictions.
  2. Grafts them into the SMPL-X fit.
  3. Applies an arm-chain placement fix to correct relative positioning.
  4. Utilizes a One-Euro temporal filter to guarantee smooth kinematics across frames.

---

## Stage 3.2: Arbitrary Object Reconstruction (Cow MVS)

To reconstruct dynamic objects like Holstein dairy cows where parametric models (like SMPL) do not exist, a classical (optimization-based) Multi-View Stereo (MVS) pipeline is used.

### 1. Foreground Masking
- **Segment Anything 3 (SAM3)**: SAM3 is employed to isolate the cow from the background in all calibrated views. 
- **Dilation**: A mask dilation of approximately 8 pixels is applied to give a margin against under-segmentation (e.g., preventing chopped hooves/ears). While this introduces a minor halo of background depths, it is resolved in the post-filtering stage.

### 2. Sparse SfM (Structure from Motion)
- **Feature Extraction & Matching**: COLMAP extracts SIFT features confined within the foreground masks. These features are exhaustively matched using FLANN nearest-neighbor search.
- **RANSAC & Triangulation**: A two-view RANSAC filter eliminates false correspondences by fitting a fundamental matrix (typical survival rates are extremely healthy, around ~92%). The verified matches are then triangulated into a sparse point cloud.

### 3. Dense MVS
- **PatchMatch Stereo**: A dense point cloud is hypothesized by estimating depths and surface normals per pixel. 
- **Stereo Fusion (min_num_pixels=1)**: The standard stereo fusion threshold was intentionally lowered to `--min_num_pixels=1`. Because the cow's textureless white hide only yields a strong depth signal from at most *one* nearby camera per surface patch, lenient fusion prevents these valid points from being discarded. PatchMatch’s internal geometric consistency check suffices to reject outliers.
- **Alternative Backends**: Alongside COLMAP, the pipeline supports modern dense MVS backends like **ACMMP**, **DPE-MVS**, and **DVP-MVS** for improved baseline reconstructions.

### 4. Post-Filtering (Visual-Hull)
- Because of the dilated masks and lenient fusion, a minor halo of background noise (e.g., ground edges) is fused into the PLY.
- **Visual-Hull Intersection**: The final point cloud is passed through a visual-hull filter. This step erodes the SAM masks by the initial dilation amount and discards any 3D points that do not project as "cow" across a minimum consensus of cameras (e.g., $\ge$ 9 out of 11 cameras).

---

## Experimental Design
The ablation studies focus heavily on the Dense MVS reconstruction robustness. We alter available camera views during fusion to measure the degradation of the visual hull and surface reconstruction:
- **K Contiguous/Uniform**: Reducing the camera count ($K$) to simulate a smaller rig, selecting either adjacent (contiguous) or spread-out (uniform) cameras.
- **Omit Camera**: Systematic leave-one-out experiments to measure the localized importance of each specific camera in the rig.
