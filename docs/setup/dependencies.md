## Dependencies

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

This page provides the dependency setup for HA-VLN.

The original repository README was written around a Python 3.7 and CUDA 11.1 era stack. This documentation provides a Python 3.8 option for newer Linux distributions and NVIDIA GPUs; it is a compatibility recipe, not the original project's pinned environment.

### System Packages

Install the common HA-VLN system dependencies first:

```bash
sudo apt-get update
sudo apt-get install -y --no-install-recommends \
    libjpeg-dev libglm-dev libgl1 libegl1-mesa-dev mesa-utils \
    xorg-dev freeglut3-dev libcrypt-dev
```

### Python Environment

Use conda to create and manage the environment:

```bash
conda create -n havlnce python=3.8 gcc_linux-64=11 gxx_linux-64=11 sysroot_linux-64=2.17 -c conda-forge -y
conda activate havlnce
```

If you plan to compile GPU-dependent extensions, export the conda toolchain before continuing:

```bash
export CC=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-gcc
export CXX=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-g++
export CUDA_HOME=$CONDA_PREFIX
```

### Core Python Packages

Install a modern CUDA-compatible stack for participant development:

```bash
# Full CUDA toolkit for compilation and runtime compatibility
conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit -y

# HA-VLN-compatible PyTorch stack
pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --index-url https://download.pytorch.org/whl/cu118
```

### Why does this recipe use Python 3.8 instead of the README's Python 3.7?

The short answer is compatibility.

- the original repository README reflects an older Python 3.7 era environment
- newer GPUs may require a newer CUDA and PyTorch combination than the Python 3.7 path typically supports well
- this recipe therefore uses Python 3.8 for broader compatibility on modern hardware

If you use the original Python 3.7 path, follow the upstream README and keep its corresponding CUDA/PyTorch versions together.
