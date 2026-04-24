# Objaverse-XL Rendering Setup Guide (No Sudo)

This guide explains how to set up the Objaverse-XL rendering environment on a server where you do not have `sudo` privileges. We use **Conda** to manage both Python dependencies and the system-level libraries required for headless rendering.

## 1. Prerequisites
- **Conda** installed and initialized.
- **Git** installed.

## 2. Environment Setup

### Create Conda Environment
We use Python 3.10 as it's the most stable version for the current requirements.
```bash
conda create -y -n objaverse-xl python=3.10
conda activate objaverse-xl
```

### Install Python Dependencies
Install the package-wide dependencies and the `objaverse` package in editable mode:
```bash
pip install -r requirements.txt
pip install -e .
```

### Install Headless Rendering Libraries
Since we cannot use `sudo apt install xserver-xorg`, we install the required X11 and OpenGL libraries directly into the Conda environment from `conda-forge`:
```bash
conda install -y -c conda-forge \
    xorg-libx11 xorg-libxext xorg-libxrender \
    xorg-libxi xorg-libxfixes xorg-libxcursor \
    xorg-libxinerama xorg-libxrandr xorg-libxcomposite \
    xorg-libxdamage xorg-libxxf86vm xorg-libsm xorg-libice \
    mesa-libgl-devel-cos7-x86_64 mesa-libglapi-cos7-x86_64 libglvnd-cos7-x86_64
```

## 3. Blender Installation
The scripts require a specific version of Blender (**3.2.2**).

```bash
cd scripts/rendering
wget https://download.blender.org/release/Blender3.2/blender-3.2.2-linux-x64.tar.xz
tar -xf blender-3.2.2-linux-x64.tar.xz
cd ../..
```

## 4. Running the Renderer

### Library Path Configuration
Blender needs to find the libraries installed in your Conda environment. You must set the `LD_LIBRARY_PATH` to include the Conda environment's library directories.

### Using the Helper Script
A helper script `run_render.sh` has been provided to automate the path configuration and environment activation.

**Execute a test render:**
```bash
./run_render.sh --num_renders 12 --gpu_devices 0
```
*(Use `--gpu_devices 0` to force CPU rendering if no GPUs are available or configured).*

### Output
By default, the rendered files are saved to the `renders` directory in the project root.

Each object is saved as a ZIP containing:
- **12 PNG images** (randomized camera views).
- **12 NPY files** (3x4 camera matrices).
- **metadata.json** (object statistics).

## 5. Troubleshooting
- **Missing `.so` libraries:** If Blender fails with a "cannot open shared object file" error, ensure your `LD_LIBRARY_PATH` includes `$CONDA_PREFIX/lib` and `$CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib64`.
- **GPU Rendering:** If you have GPUs but they are not being used, check your CUDA version compatibility with Blender 3.2.2.
