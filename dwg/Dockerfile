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
  libcgal-dev libpcl-dev libboost-all-dev && \
  apt clean && rm -rf /var/lib/apt/lists/*

# change the working directory to the repository
WORKDIR /workspace
RUN git clone --recursive https://github.com/DWGTech/DWG.git
WORKDIR /workspace/DWG


# prepare for build
ENV CPATH="/usr/local/cuda/include/cccl"
ENV C_INCLUDE_PATH="/usr/local/cuda/include/cccl"
ENV CPLUS_INCLUDE_PATH="/usr/local/cuda/include/cccl"

RUN sed -i 's/set( *CMAKE_CUDA_STANDARD *14 *)/set(CMAKE_CUDA_STANDARD 17)/' CMakeLists.txt
RUN sed -i \
  -e 's/set(CMAKE_CUDA_FLAGS "")/set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS}")/' \
  -e 's/set(CMAKE_CUDA_FLAGS_RELEASE "")/set(CMAKE_CUDA_FLAGS_RELEASE "${CMAKE_CUDA_FLAGS_RELEASE}")/' \
  -e 's/set(CMAKE_CUDA_FLAGS_DEBUG "")/set(CMAKE_CUDA_FLAGS_DEBUG "${CMAKE_CUDA_FLAGS_DEBUG}")/' \
  CMakeLists.txt

# build dwg
RUN mkdir build && cd build && \
  cmake .. -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_FLAGS=" -include thrust/sequence.h -include thrust/sort.h" && \
  make -j"$(nproc)"

ENV PATH="/workspace/DWG/build/bin:${PATH}"

CMD ["/bin/bash"]
