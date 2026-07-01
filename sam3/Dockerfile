FROM nvidia/cuda:13.0.3-cudnn-devel-ubuntu24.04

# Blackwell = sm_120
ENV DEBIAN_FRONTEND=noninteractive \
  TORCH_CUDA_ARCH_LIST="12.0" \
  MAX_JOBS=4

# install basic package
RUN apt-get update && apt-get install -y --no-install-recommends \
  ca-certificates libeigen3-dev libjpeg-dev libglib2.0-dev libgomp1 libgl1 \
  libbz2-dev libegl1-mesa-dev libffi-dev libgdbm-dev libreadline-dev libsqlite3-dev libssl-dev tk-dev uuid-dev xz-utils zlib1g-dev \
  build-essential cmake ninja-build \
  python3 python3-dev python3-pip python3-venv \
  curl ffmpeg git im tmux unzip wget \
  apt clean && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /workspace/venv
ENV PATH="/workspace/venv/bin:${PATH}"

# install conda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
  chmod +x Miniconda3-latest-Linux-x86_64.sh && \
  ./Miniconda3-latest-Linux-x86_64.sh -b -p /workspace/miniconda3 && \
  rm Miniconda3-latest-Linux-x86_64.sh

# update PATH environment variable
ENV PATH="/workspace/miniconda3/bin:${PATH}"

# initialize conda
RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main && \
  conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

# create and activate conda environment
RUN conda create -n sam3 python=3.12 && echo "source activate sam3" > ~/.bashrc
ENV PATH="/workspace/miniconda3/envs/sam3/bin:${PATH}"

# install pytorch
RUN pip install --no-cache-dir torch==2.10.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130

# change the working directory to the repository
RUN git clone --recursive https://github.com/facebookresearch/sam3.git /workspace/sam3
WORKDIR /workspace/sam3
RUN pip install --no-cache-dir ".[notebooks,train,dev]"

# Optional dependencies for faster inference
RUN pip install --no-cache-dir einops ninja setuptools==69.5.1 && \
  pip install --no-cache-dir --no-build-isolation flash-attn-3 --no-deps --index-url https://download.pytorch.org/whl/cu128 && \
  pip install --no-cache-dir --no-build-isolation git+https://github.com/ronghanghu/cc_torch.git

CMD ["/bin/bash"]
