# Dependencies

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This page outlines the system-level and library prerequisites for running HA-VLN.

---

## 1. Zero-Dependency Option: Docker

If you have Docker and the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) installed, you do not need to install local system development packages or compile C++/CUDA extensions. The official image contains all required runtimes:

```bash
IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4
docker pull "$IMAGE"
```

---

## 2. Native System Packages (Linux & WSL2)

For native installations on modern distributions (Ubuntu 20.04 / 22.04 / 24.04 and WSL2), install the graphics, headless rendering, and compilation libraries:

```bash
sudo apt-get update
sudo apt-get install -y --no-install-recommends \
  libjpeg-dev libglm-dev libgl1 libegl1-mesa-dev mesa-utils \
  xorg-dev freeglut3-dev libcrypt-dev curl unzip git
```

---

## 3. Python & Compiler Toolchain (Native Conda)

We recommend Conda with Python 3.8 and CUDA Toolkit 11.8 for modern GPUs (RTX 30/40 series, A100, H100):

```bash
conda create -n havlnce python=3.8 pip=24.0 -c conda-forge -y
conda activate havlnce

# Install pre-built headless Habitat-Sim and system libraries
conda install -c aihabitat -c conda-forge \
  "habitat-sim=0.1.7=*headless*" "numpy=1.23.5" \
  python-lmdb libxcrypt libopengl libglx -y

# PyTorch with CUDA 11.8 support
python -m pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 \
  --index-url https://download.pytorch.org/whl/cu118
```

For full step-by-step installation instructions including Habitat-Sim and Habitat-Lab, proceed to [Installation Steps](installation.md).
