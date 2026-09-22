# Setup & Codebase

## Overview
The primary codebase is located in the `motion-capture` repository. It contains the following core components:
- **`easymocap`**: The core python module for multi-view capture and processing.
- **`apps`**: Various pipeline applications like calibration, annotation, and reconstruction.
- **`scripts`**: Helper scripts for preprocessing, postprocessing, and Slurm cluster submissions.
- **`config`**: Configuration yaml files for the pipelines.

## Environment & HPC Setup
This project was largely executed on the Digital Research Alliance of Canada (Alliance) cluster. 
Key notes from the `drac-harness` wiki:
- Use `module load python` and standard virtual environments (`venv`) for python dependencies.
- Beware of Slurm gotchas when requesting GPUs (e.g., specifying the exact memory and GPU type).
- Custom CUDA extensions might require careful Wheelhouse configurations.

## Running the Pipeline
To run a general motion-capture pipeline, refer to the scripts in `motion-capture/scripts/` or submit a job via `motion-capture/scripts/slurm/`.
