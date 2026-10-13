# Installation Steps

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

Choose between the pre-built **Docker container** (recommended for quick evaluation without dependency compilation) or a **Native Conda** environment (Python 3.8 + CUDA 11.8).

---

## 1. Docker Container Installation (Recommended)

The official Docker image packages all Habitat-Sim 0.1.7 binaries, CUDA 11.8 drivers, PyTorch 2.0.1, and evaluation dependencies.

### Pull Pre-built Container

```bash
IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4
docker pull "$IMAGE"
```

### Interactive Development Container

Clone the repository and mount it to inspect the environment or run interactive commands:

```bash
git clone https://github.com/JostarXiong/HA-VLN.git
cd HA-VLN

DATA_DIR="/absolute/path/to/Data"

docker run --gpus all -it --rm \
  --shm-size 16g \
  --mount type=bind,source="$(pwd)",target=/workspace/HA-VLN \
  --mount type=bind,source="$DATA_DIR",target=/workspace/HA-VLN/Data \
  --mount type=bind,source="$DATA_DIR",target=/data/havln2 \
  --workdir /workspace/HA-VLN \
  "$IMAGE" bash
```

Inside the interactive container, run the baseline setup script before launching agent commands:

```bash
# Inside the container:
bash scripts/setup_docker_cma.sh
```

---

## 2. Native Conda Installation (Python 3.8 + CUDA 11.8)

This stack is adapted for modern GPUs (RTX 30/40 series, A100, H100) using Python 3.8, PyTorch 2.0.1, and CUDA 11.8.

### Step 1: Base Environment & Habitat-Sim

```bash
git clone https://github.com/JostarXiong/HA-VLN.git
cd HA-VLN
export HA_VLN_ROOT="$(pwd)"

# 1. Create and activate conda environment
conda create -n havlnce python=3.8 pip=24.0 -c conda-forge -y
conda activate havlnce

# 2. Install pre-built headless Habitat-Sim and system libraries
conda install -c aihabitat -c conda-forge \
  "habitat-sim=0.1.7=*headless*" "numpy=1.23.5" \
  python-lmdb libxcrypt libopengl libglx -y

# 3. Install PyTorch with CUDA 11.8 support
python -m pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 \
  --index-url https://download.pytorch.org/whl/cu118
```

### Step 2: Install Habitat-Lab 0.1.7

Clone Habitat-Lab 0.1.7 and install it in development mode:

```bash
# 4. Clone and install Habitat-Lab 0.1.7 in development mode
git clone --branch v0.1.7 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab
python setup.py develop --all
cd "$HA_VLN_ROOT"
```

### Step 3: Install Agent Compatibility Dependencies

Install the Python 3.8 compatibility requirements from the repository root:

```bash
# 5. Install Python 3.8 compatibility requirements (from repository root)
python -m pip install -r requirements-py38.txt \
  --extra-index-url https://download.pytorch.org/whl/cu118
```

> [!WARNING]
> Do **not** run `pip install -r agent/VLN-CE/requirements.txt`. The legacy file contains `torchvision==0.2.2.post3`, which would downgrade torchvision and break the CUDA 11.8 environment. All required packages for the agent are already included in `requirements-py38.txt`.

---

## 3. Optional Modules

<details>
<summary><b>Setup GroundingDINO for Human Counting (Optional)</b></summary>
<br>

*Note: GroundingDINO is an optional simulator perception module for online human detection, observation logging, and human counting (`HASimulator/detector.py`). Standard navigation policies (such as HA-VLN-CMA) do not require GroundingDINO.*

```bash
cd "${HA_VLN_ROOT:-$(pwd)}"

# Install C/C++ compilers and CUDA Toolkit required for building DINO
conda install -c nvidia/label/cuda-11.8.0 -c conda-forge \
  cuda-toolkit gcc_linux-64=11 gxx_linux-64=11 sysroot_linux-64=2.17 -y

export CUDA_HOME="$CONDA_PREFIX"
export CC="$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-gcc"
export CXX="$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-g++"
export PATH="$CUDA_HOME/bin:$PATH"

# Install GroundingDINO perception dependencies
python -m pip install -r requirements-dino-py38.txt

git clone https://github.com/IDEA-Research/GroundingDINO.git HASimulator/GroundingDINO
git -C HASimulator/GroundingDINO checkout df5b48a3efbaa64288d8d0ad09b748ac86f22671
MAX_JOBS=2 python -m pip install --no-deps --no-build-isolation \
  -e HASimulator/GroundingDINO

mkdir -p HASimulator/GroundingDINO/weights
curl -fL --retry 3 \
  https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha/groundingdino_swint_ogc.pth \
  -o HASimulator/GroundingDINO/weights/groundingdino_swint_ogc.pth
python -m pip check
```

</details>

<details>
<summary><b>Legacy Python 3.7 Native Environment</b></summary>
<br>

These commands retain the original software stack for historical reference:

```bash
conda create -n havln-agent python=3.7 -y
conda activate havln-agent

# Install Habitat-Sim 0.1.7 (Headless)
conda install -c aihabitat -c conda-forge habitat-sim=0.1.7=py3.7_linux_headless_da39a3ee5e6b4b0d3255bfef95601890afd80709 -y

# Clone and install Habitat-Lab 0.1.7
git clone --branch v0.1.7 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab
pip install -r requirements.txt
pip install -r habitat_baselines/rl/requirements.txt
python setup.py develop --all
cd ..

# Agent packages (Python 3.7)
pip install torch==1.9.1+cu111 torchvision==0.10.1+cu111 -f https://download.pytorch.org/whl/torch_stable.html
pip install -r requirements.txt
```

</details>

---

## 4. Verification & Troubleshooting

Verify that headless GPU rendering and PyTorch operate properly:

```bash
# 1. Verify PyTorch CUDA availability
python -c "import torch; assert torch.cuda.is_available(), 'CUDA not available'; print('PyTorch CUDA OK:', torch.cuda.get_device_name(0))"

# 2. Verify Headless Habitat-Sim rendering
python -c "import habitat_sim; print('Habitat-Sim version:', habitat_sim.__version__)"

# 3. Test scene rendering & dynamic humans
# Headless verification (saves a verification frame to scripts/test/demo_frame.png):
python scripts/demo.py --scan 1LXtFkjw3qL --headless

# Interactive keyboard control (requires an active GUI display session):
python scripts/demo.py --scan 1LXtFkjw3qL
```
