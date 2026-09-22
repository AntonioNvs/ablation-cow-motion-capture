# Ablation Cow Motion Capture - Documentation

This repository contains the MkDocs source files for the documentation and experiment logs regarding the multi-view 3D reconstruction of Holstein dairy cows.

**Live Site**: [https://antonionvs.github.io/ablation-cow-motion-capture/](https://antonionvs.github.io/ablation-cow-motion-capture/)

## Overview
The report outlines the full methodology, setup codebase, and the 60+ extensive ablation experiments investigating camera rig reductions (K Contiguous, K Uniform, and specific camera omission). 

## Local Development

To run this documentation site locally:

1. Create a Python virtual environment:
```bash
python -m venv venv
source venv/bin/activate
```

2. Install MkDocs and the Material theme:
```bash
pip install mkdocs mkdocs-material
```

3. Serve the site locally:
```bash
mkdocs serve
```
Then open `http://127.0.0.1:8000` in your browser.

## Deployment

To deploy updates to GitHub Pages, run:
```bash
mkdocs gh-deploy
```
