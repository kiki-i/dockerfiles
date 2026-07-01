FROM nvidia/cuda:13.0.3-cudnn-devel-ubuntu24.04

# Add a volume for downloaded models
# VOLUME /workspace/models

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
  curl ffmpeg git im tmux unzip wget && \
  apt clean && rm -rf /var/lib/apt/lists/*

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
RUN conda create -n instantmesh python=3.10 && echo "source activate instantmesh" > ~/.bashrc
ENV PATH="/workspace/miniconda3/envs/instantmesh/bin:${PATH}"

RUN pip install --no-cache-dir torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130 && \
  pip install --no-cache-dir xformers --index-url https://download.pytorch.org/whl/cu130 && \
  pip install --no-cache-dir triton

# change the working directory to the repository
WORKDIR /workspace
RUN git clone --recursive https://github.com/TencentARC/InstantMesh.git
WORKDIR /workspace/InstantMesh

# other dependencies
ADD ./requirements.txt /workspace/InstantMesh/requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /tmp/extensions && \
  git clone https://github.com/NVlabs/nvdiffrast.git /tmp/extensions/nvdiffrast && \
  pip install --no-cache-dir --no-build-isolation /tmp/extensions/nvdiffrast && \
  pip install --no-cache-dir onnxruntime huggingface_hub==0.25.2

CMD ["/bin/bash"]
