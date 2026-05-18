# Quick Start (Modern GPU Adapted Version)

This guide targets a Python 3.8 and CUDA 11.8 stack that works on modern hardware such as the NVIDIA H100 (sm_90 architecture). The following steps address dependency conflicts, deprecated functions, and compilation issues so the project can run successfully on modern GPUs with PyTorch 2.0+.


## 1. Create the Environment & Install Compilers

Create a Python 3.8 environment and install the GCC 11 compiler toolchain natively within Conda. This prevents compilation conflicts with newer system-level compilers on modern Linux distributions. 

```bash
conda create -n havln python=3.8 gcc_linux-64=11 gxx_linux-64=11 sysroot_linux-64=2.17 -c conda-forge -y
conda activate havln

# Install full CUDA Toolkit 11.8 (provides nvcc compiler for sm_90 architecture)
conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit -y

# Install H100 compatible PyTorch and Torchvision
pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --index-url https://download.pytorch.org/whl/cu118
```

*Note: Before proceeding to Step 4 to compile GroundingDINO, ensure you export the Conda compilers so the build script uses them instead of the system default:*
```bash
export CC=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-gcc
export CXX=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-g++
export CUDA_HOME=$CONDA_PREFIX
```

## 2. Install Habitat-Sim (Headless)

Install the headless version of `habitat-sim` strictly via Conda. We use the build string wildcard `*headless*` to prevent package managers from silently downgrading to a physical display version.

```bash
# Install required system dependencies
conda install -c conda-forge python-lmdb libxcrypt libffi -y

# Force install the pure EGL headless build
conda install -c aihabitat -c conda-forge "habitat-sim=0.1.7=*headless*" -y
```

## 3. Install Habitat-Lab

Install `habitat-lab` with modern dependency replacements. `numpy` is capped to avoid the deprecated `np.float` issue, and legacy `tensorflow` is replaced with `tensorboard`.

```bash
git clone --branch v0.1.7 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab

# Prevent deprecated np.float error and install basic tools
pip install msgpack "numpy<1.24.0" tensorboard

# Remove outdated tensorflow requirement
sed -i 's/tensorflow==1.13.1/# tensorflow==1.13.1/g' habitat_baselines/rl/requirements.txt

pip install -r requirements.txt
pip install -r habitat_baselines/rl/requirements.txt
python setup.py develop --all
cd ..
```

## 4. Compile GroundingDINO

Clean any old compilation caches and build the CUDA extensions using the current environment's PyTorch. `ninja` is used to accelerate the build process.

```bash
cd HASimulator
git clone https://github.com/IDEA-Research/GroundingDINO.git
cd GroundingDINO/

# Clean old build files to prevent "undefined symbol" errors
rm -rf build/ dist/ *.egg-info

# Install dependencies
pip install "safetensors==0.3.1" "huggingface-hub==0.16.4" "tokenizers==0.13.3" "transformers==4.30.2" "timm==0.9.2"
sed -i 's/supervision.*/supervision==0.11.1/g' requirements.txt

# Compile extension
pip install ninja
pip install -e .

# Download weights
mkdir weights
cd weights
wget -q --no-check-certificate https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha/groundingdino_swint_ogc.pth
cd ../../../
```

## 5. Install Agent Requirements & Fix Dependencies

Install the requirements for the VLN-CE agent. This step includes strict versioning for `Pillow` and forcibly reinstalls `torchvision` to ensure pip hasn't overwritten the CUDA 11.8 version.

```bash
pip install -r agent/VLN-CE/requirements.txt

# Fix Pillow version to retain PILLOW_VERSION constant
pip install "Pillow<9.0.0" setuptools webdataset==0.1.40

# Crucial: Re-assert H100 PyTorch to prevent pip requirements from downgrading torchvision
pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --index-url https://download.pytorch.org/whl/cu118
```

## 6. Code Modification (Gym Compatibility)

Modern versions of `gym` no longer allow `Discrete(0)` initialization. We need to patch the observation space initialization in the `habitat-lab` package. 

Since this specific initialization occurs only once in the codebase, we can automatically patch it using `sed`. Run the following command from your project root:

```bash
sed -i 's/spaces.Discrete(0)/spaces.Discrete(4)/g' habitat-lab/habitat/tasks/vln/vln.py
```

## 7. Download Datasets and Weights

Follow the dataset extraction procedure below. Note the modified extraction step for DD-PPO models to prevent directory nesting errors.

```bash
# 1. Download and extract Matterport3D Dataset (Requires download_mp.py)
python3 download_mp.py -o Data/scene_datasets --task habitat
unzip Data/scene_datasets/v1/tasks/mp3d_habitat.zip -d Data/scene_datasets/

# 2. Download and extract HA-R2R and HAPS 2.0 datasets
bash scripts/download_data.sh

# 3. Download DD-PPO model weights
wget --no-check-certificate https://dl.fbaipublicfiles.com/habitat/data/baselines/v1/ddppo/ddppo-models.zip

# Extract directly to Data/ to avoid nested 'data/ddppo-models/ddppo-models' directories
unzip -j ddppo-models.zip "data/ddppo-models/*" -d Data/ddppo-models/
rm ddppo-models.zip
```

*(Note: Ensure Human-Scene Fusion assets and NavMesh-related settings are configured correctly for your local data layout before training or evaluation.)*

## 8. Setup GPU & Run 

Before running training or inference, inject the CUDA library paths and set GPU-related environment variables in a more portable way. The defaults below keep the current single-GPU behavior, while still allowing you to override the visible GPUs or render device without editing the document.

```bash
cd agent

# Inject CUDA environments
export CUDA_HOME=$CONDA_PREFIX
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib:$LD_LIBRARY_PATH

# GPU selection defaults
# Keep a single GPU visible by default; override as needed, for example:
#   export HAVLN_CUDA_VISIBLE_DEVICES=0,1
# To keep all GPUs visible, skip the export below.
export CUDA_VISIBLE_DEVICES=${HAVLN_CUDA_VISIBLE_DEVICES:-0}

# Render on the first visible GPU by default.
# If you want a different render GPU among the visible devices, override for example:
#   export HAVLN_EGL_DEVICE_ID=1
export EGL_DEVICE_ID=${HAVLN_EGL_DEVICE_ID:-0}
export DISPLAY=""

# Training
python run.py --exp-config config/cma_pm_da_aug_tune.yaml --run-type train

# Evaluation
python run.py --exp-config config/cma_pm_da_aug_tune.yaml --run-type eval

# Inference
python run.py --exp-config config/cma_pm_da_aug_tune.yaml --run-type inference
```

## 9. Frequently Asked Questions (FAQ)

### Q1: `ImportError: cannot import name 'PILLOW_VERSION' from 'PIL'`
**Cause:** Older versions of `habitat-lab` rely on the `PILLOW_VERSION` constant, which was completely removed in Pillow >= 9.0.0. If you need to run with a newer Pillow for some reason, follow the solution below to monkey-patch the constant.  
**Solution:** Instead of downgrading Pillow, monkey-patch the constant in your script. Add the following lines at the very beginning of your main training/inference scripts (`run.py` or `demo.py`):
```python
import PIL
if not hasattr(PIL, 'PILLOW_VERSION'):
    PIL.PILLOW_VERSION = PIL.__version__
```

### Q2: GroundingDINO compilation fails with `__int128` typedef errors
**Cause:** Even with the Conda GCC toolchain, strict type definitions in some modern Linux sysroots clash with older PyTorch extension C++ headers.  
**Solution:** You need to comment out the conflicting `__int128` definitions in the Conda environment's `types.h` file. Run this before recompiling:
```bash
SYSROOT_TYPES="$CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/include/linux/types.h"
chmod +w $SYSROOT_TYPES
sed -i 's/typedef __signed__ __int128/\/\/ typedef __signed__ __int128/g' $SYSROOT_TYPES
sed -i 's/typedef unsigned __int128/\/\/ typedef unsigned __int128/g' $SYSROOT_TYPES
```

### Q3: Habitat-Sim throws EGL Initialization Errors on H100
**Cause:** The NVIDIA H100 is a pure datacenter compute card with no display controllers. Sometimes, standard EGL struggles to find a valid rendering device if the environment isn't strictly isolated.
**Solution:** Ensure you are using the `headless` build of `habitat-sim` as instructed in Step 2. Additionally, strictly define the GPU and display environments before running your agent:
```bash
export CUDA_VISIBLE_DEVICES=${HAVLN_CUDA_VISIBLE_DEVICES:-0}
export EGL_DEVICE_ID=${HAVLN_EGL_DEVICE_ID:-0}
export DISPLAY=""
```
If you need a different setup, override the helper variables before running, for example:
```bash
export HAVLN_CUDA_VISIBLE_DEVICES=1,2
export HAVLN_EGL_DEVICE_ID=0
```

### Q4: SSL Certificate errors when downloading DD-PPO weights
**Cause:** Modern Ubuntu environments have updated their OpenSSL configurations, which can reject the legacy SSL certificates from older `fbaipublicfiles.com` servers.  
**Solution:** Bypass the SSL check manually during the download step:
```bash
wget --no-check-certificate https://dl.fbaipublicfiles.com/habitat/data/baselines/v1/ddppo/ddppo-models.zip
```

### Q5: Installation fails with `invalid command 'bdist_wheel'` or `use_2to3 is invalid` during `pip install -e .`
**Cause:** Modern versions of `setuptools` (>= 58.0.0) have completely removed support for the `use_2to3` flag. However, the `setup.py` scripts of older repositories like `habitat-lab` or legacy RL baselines still rely on this deprecated feature.
**Solution:** You must strictly pin the `setuptools` version before compiling these older repositories. The setup instructions handle this by executing `pip install setuptools==59.5.0` (which retains legacy compatibility for this specific build tree). Do not upgrade `setuptools` locally when working with the `habitat-lab` source.
