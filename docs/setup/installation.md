## Installation Steps

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

The following installation flow is a Python 3.8 compatibility recipe for HA-VLN 2.0.

The original repository README documents its Python 3.7 environment.

### 1. Clone Repository

```bash
git clone https://github.com/UWMILab/HA-VLN.git
cd HA-VLN
```

### 2. Create the Runtime Environment

```bash
conda create -n havlnce python=3.8 gcc_linux-64=11 gxx_linux-64=11 sysroot_linux-64=2.17 -c conda-forge -y
conda activate havlnce

conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit -y
pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --index-url https://download.pytorch.org/whl/cu118

export CC=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-gcc
export CXX=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-g++
export CUDA_HOME=$CONDA_PREFIX
```

### 3. Install Habitat-Sim and Habitat-Lab

```bash
conda install -c conda-forge python-lmdb libxcrypt libffi -y
conda install -c aihabitat -c conda-forge "habitat-sim=0.1.7=*headless*" -y

git clone --branch v0.1.7 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab
pip install msgpack "numpy<1.24.0" tensorboard
sed -i 's/tensorflow==1.13.1/# tensorflow==1.13.1/g' habitat_baselines/rl/requirements.txt
pip install -r requirements.txt
pip install -r habitat_baselines/rl/requirements.txt
python setup.py develop --all
cd ..
```

### 4. Install GroundingDINO

```bash
cd HASimulator
git clone https://github.com/IDEA-Research/GroundingDINO.git
cd GroundingDINO

rm -rf build/ dist/ *.egg-info
pip install "safetensors==0.3.1" "huggingface-hub==0.16.4" \
    "tokenizers==0.13.3" "transformers==4.30.2" "timm==0.9.2"
sed -i 's/supervision.*/supervision==0.11.1/g' requirements.txt
pip install ninja
pip install -e .
cd ../..
```

If GroundingDINO compilation hits `__int128` typedef errors, patch the conda sysroot header before retrying:

```bash
SYSROOT_TYPES="$CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/include/linux/types.h"
chmod +w "$SYSROOT_TYPES"
sed -i 's/typedef __signed__ __int128/\/\/ typedef __signed__ __int128/g' "$SYSROOT_TYPES"
sed -i 's/typedef unsigned __int128/\/\/ typedef unsigned __int128/g' "$SYSROOT_TYPES"
```

### 5. Install Agent-Side Python Requirements

```bash
pip install -r agent/VLN-CE/requirements.txt
pip install "Pillow<9.0.0" setuptools webdataset==0.1.40

# Reassert the expected PyTorch stack in case pip changed it
pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --index-url https://download.pytorch.org/whl/cu118
```

### 6. Apply the Gym Compatibility Patch

Modern versions of `gym` do not allow `Discrete(0)` initialization in the old code path used here.

```bash
sed -i 's/spaces.Discrete(0)/spaces.Discrete(4)/g' habitat-lab/habitat/tasks/vln/vln.py
```

### 7. Verify Runtime

At minimum, verify these checks:

```bash
python -c "import torch; print(torch.__version__)"
python -c "import habitat_sim; print('habitat-sim OK')"
```

### FAQ: Why does this page use Python 3.8 instead of the original README's Python 3.7?

The original README reflects an older environment. This compatibility recipe uses Python 3.8 because it can be more practical for newer GPUs and CUDA/PyTorch combinations.

Keep the environment versions consistent within either recipe; do not combine Habitat 0.1.7 with a newer Habitat runtime.
