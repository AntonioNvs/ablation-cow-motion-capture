# Executive Summary

## Problem Statement
This project extends multi-view motion capture methodologies to both human subjects and arbitrary moving objects, with a particular research focus on the 3D reconstruction of Holstein dairy cows. The challenge is to achieve accurate, high-fidelity 3D reconstruction from calibrated multi-view camera rigs without predefined body models for the cows, overcoming issues related to occlusion, camera synchronization, and environment noise.

## Objectives
- Implement a robust multi-view stereo (MVS) pipeline capable of reconstructing dynamic objects like dairy cows.
- Perform ablation studies (e.g., omitting specific cameras, altering contiguous/uniform camera subsets) to determine the minimal optimal rig setup.
- Integrate human pose estimation (MMPose, SMPL-X) and refine extremities (HaMeR hand refinement).

## Key Findings & Results
The ablation experiments on the Holstein dairy cow dataset (`cow-2026-07-29`) yielded insights into camera placement importance and reconstruction volume. Detailed plots and metric summaries for camera subset ablations and individual camera omissions can be found in the [Experiments](experiments/ablation_k_contiguous.md) section.

> [!tip] Quick Link
> For instructions on setting up the environment, compiling dense MVS backends, and reproducing these results, see [Setup & Codebase](setup_and_codebase.md).
