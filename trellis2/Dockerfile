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

# install pytorch
RUN pip install --no-cache-dir torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130


# change the working directory to the repository
WORKDIR /workspace
RUN git clone --recursive https://github.com/microsoft/TRELLIS.2.git
WORKDIR /workspace/TRELLIS.2

# basic
RUN pip install --no-cache-dir imageio imageio-ffmpeg tqdm easydict opencv-python-headless ninja \
  trimesh transformers gradio==6.0.1 tensorboard pandas lpips zstandard kornia timm && \
  pip install --no-cache-dir git+https://github.com/EasternJournalist/utils3d.git@9a4eb15e4021b67b12c460c7057d642626897ec8

# flash-attention
RUN pip install --no-cache-dir --no-build-isolation flash-attn==2.7.3

# CuMesh
RUN mkdir -p /tmp/extensions && \
  git clone --recursive https://github.com/JeffreyXiang/CuMesh.git /tmp/extensions/CuMesh && \
  find /tmp/extensions/CuMesh -type f \( -name "*.cu" -o -name "*.cuh" -o -name "*.h" -o -name "*.cpp" \) \
  -exec sed -i 's/cub::Sum()/thrust::plus<float>()/g' {} + && \
  pip install --no-cache-dir /tmp/extensions/CuMesh --no-build-isolation

# FlexGEMM
RUN mkdir -p /tmp/extensions && \
  git clone --recursive https://github.com/JeffreyXiang/FlexGEMM.git /tmp/extensions/FlexGEMM && \
  pip install --no-cache-dir /tmp/extensions/FlexGEMM --no-build-isolation

# nvdiffrast
RUN mkdir -p /tmp/extensions && \
  git clone -b v0.4.0 https://github.com/NVlabs/nvdiffrast.git /tmp/extensions/nvdiffrast && \
  pip install --no-cache-dir /tmp/extensions/nvdiffrast --no-build-isolation

# nvdiffrec
RUN mkdir -p /tmp/extensions && \
  git clone -b renderutils https://github.com/JeffreyXiang/nvdiffrec.git /tmp/extensions/nvdiffrec && \
  pip install --no-cache-dir /tmp/extensions/nvdiffrec --no-build-isolation

# o-voxel
RUN rm -rf o-voxel/third_party/eigen && \
  ln -s /usr/include/eigen3 o-voxel/third_party/eigen && \
  cp -r o-voxel /tmp/extensions/o-voxel && \
  pip install --no-cache-dir /tmp/extensions/o-voxel --no-build-isolation

RUN sed -i \
  -e 's/self\.model\.layer/self.model.model.layer/g' \
  trellis2/modules/image_feature_extractor.py

CMD ["/bin/bash"]
