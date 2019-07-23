FROM balenalib/jetson-nano-ubuntu:bionic-run as builder

ENV DEBIAN_FRONTEND noninteractive

WORKDIR /usr/src/app

RUN \
  apt-get update && \
  apt-get install --no-install-recommends -y \  
    python3 \
    python3-dev \
    python3-setuptools \
    lbzip2 \
    libavcodec57 \
    libavformat57 \
    libavutil55 \
    libcairo2 \
    libgdk-pixbuf2.0-0 \
    libgstreamer-plugins-base1.0-0 \
    libgstreamer1.0-0 \
    libgtk2.0-0 \
    gstreamer1.0-plugins-bad \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    libjpeg8 \
    libpng16-16 \
    libswscale4 \
    libtbb2 \
    libtiff5 \
    libtbb-dev \
    wget \
    unzip \ 
    zlib1g-dev \
    zip \
    libhdf5-100 \
    git \
    cmake \
    pkg-config \
    python3-pillow \
    lbzip2 \
    xorg-dev \
    libavcodec-dev \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    libdc1394-22-dev && \
  python3 /usr/lib/python3/dist-packages/easy_install.py pip && \
  rm -rf /var/lib/apt/lists/* && \
  apt-get clean

# NVIDIA JetPack SDK & drivers (1.7GB)
ADD nvidia /usr/src/app/nvidia

# Install NVIDIA JetPack SDK & drivers & toolchain
RUN \
  dpkg -i nvidia/deb/cuda-repo-l4t-10-0-local-10.0.166_1.0-1_arm64.deb && \  
  apt-key add /var/cuda-repo-10-0-local-10.0.166/*.pub && \
  apt-get update && \
  apt-get install -y cuda-cublas-10-0 cuda-cudart-10-0 cuda-toolkit-10-0 && \
  dpkg --remove cuda-repo-l4t-10-0-local-10.0.166 && \
  dpkg -P cuda-repo-l4t-10-0-local-10.0.166 && \
  rm nvidia/deb/cuda-repo-l4t-10-0-local-10.0.166_1.0-1_arm64.deb && \
  dpkg -i nvidia/deb/*.deb && \
  tar xjf nvidia/nvidia_drivers.tbz2 -C / && \
  tar xjf nvidia/config.tbz2 -C / --exclude=etc/hosts --exclude=etc/hostname && \
  tar xjf nvidia/nvgstapps.tbz2 -C / && \
  cp /usr/lib/aarch64-linux-gnu/tegra-egl/nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json && \
  echo "/usr/lib/aarch64-linux-gnu/tegra" > /etc/ld.so.conf.d/nvidia-tegra.conf && \
  echo "/usr/lib/aarch64-linux-gnu/tegra-egl" > /etc/ld.so.conf.d/nvidia-tegra-egl.conf && \
  ldconfig && \
  rm -rf /usr/local/cuda-10.0/doc && \
  rm -rf /usr/local/cuda-10.0/samples && \
  rm -rf /usr/local/cuda-10.0/NgithCompute-1.0 && \
  rm -rf /usr/src/tensorrt/python/data && \
  rm -rf /usr/src/tensorrt/data && \
  rm -rf nvidia && \
  rm -rf /var/lib/apt/lists/* && \
  apt-get clean

RUN \
  mkdir wheel && cd wheel && \
  wget -q https://github.com/zrzka/python-wheel-aarch64/releases/download/jetson-nano-1.1/grpcio-1.21.1-cp36-cp36m-linux_aarch64.whl && \
  wget -q https://github.com/zrzka/python-wheel-aarch64/releases/download/jetson-nano-1.1/h5py-2.9.0-cp36-cp36m-linux_aarch64.whl && \
  wget -q https://github.com/zrzka/python-wheel-aarch64/releases/download/jetson-nano-1.1/numpy-1.16.4-cp36-cp36m-linux_aarch64.whl && \
  # DON'T CHANGE THE ORDER - otherwise pip will fetch sources and will build them again
  pip3 install --no-cache-dir grpcio-1.21.1-cp36-cp36m-linux_aarch64.whl && \
  pip3 install --no-cache-dir numpy-1.16.4-cp36-cp36m-linux_aarch64.whl && \
  pip3 install --no-cache-dir h5py-2.9.0-cp36-cp36m-linux_aarch64.whl && \
  cd .. && rm -rf wheel && \
  pip3 install --no-cache-dir wheel && \
  pip3 install --no-cache-dir traitlets ipywidgets && \
  pip3 install --no-cache-dir absl-py py-cpuinfo psutil portpicker six mock requests gast astor termcolor tensorflow-estimator && \
  pip3 install --no-cache-dir --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42 tensorflow-gpu

# OpenCV
ARG OPENCV_MAKE_J=2

RUN \
  wget https://github.com/opencv/opencv/archive/4.0.1.zip && \
  unzip 4.0.1.zip && rm 4.0.1.zip && \
  # TODO: Check if libopencv contains same features, so, we can skip compilation phase
  apt-get remove libopencv -y && \
  wget https://github.com/opencv/opencv_contrib/archive/4.0.1.zip -O opencv_modules.4.0.1.zip && \
  unzip opencv_modules.4.0.1.zip && rm opencv_modules.4.0.1.zip && \
  export CUDA_HOME=/usr/local/cuda-10.0/ && \
  export LD_LIBRARY_PATH=${CUDA_HOME}/lib64 && \
  PATH=${CUDA_HOME}/bin:${PATH} && export PATH && \
  mkdir -p opencv-4.0.1/build && cd opencv-4.0.1/build && \
  cmake \
    -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_BUILD_TYPE=RELEASE \
    -D WITH_CUDA=ON \
    -D CUDA_ARCH_BIN="5.3" \
    -D CUDA_ARCH_PTX="" \
    -D WITH_CUBLAS=ON \
    -D ENABLE_FAST_MATH=ON \
    -D ENABLE_NEON=ON \
    -D WITH_LIBV4L=ON \
    -D BUILD_TESTS=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D BUILD_EXAMPLES=OFF \
    -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.0.1/modules \
    -D WITH_QT=OFF \
    -D WITH_OPENGL=OFF \
    -D WITH_GSTREAMER=ON \
    -D WITH_V4L=ON \
    -D CMAKE_INSTALL_PREFIX=/usr/local .. && \
  make -j$OPENCV_MAKE_J && \
  make install && \
  cp /usr/src/app/opencv-4.0.1/build/bin/opencv_version /usr/src/app/ && \
  cd /usr/src/app/ && rm -rf /usr/src/app/opencv-4.0.1 && \
  rm -rf /usr/src/app/opencv_contrib-4.0.1 && \
  rm /usr/local/cuda-10.0/targets/aarch64-linux/lib/*.a

# https://www.waveshare.com/wiki/IMX219-160_Camera
RUN \
  pip3 install \
    boto3 \
    flask \
    flask-socketio \
    requests \
    eventlet \
    pytelegraf

#RUN \
#  wget https://www.waveshare.com/w/upload/e/eb/Camera_overrides.tar.gz && \
#  tar zxvf Camera_overrides.tar.gz -C /var/nvidia/nvcam/settings/ && \
#  rm -f Camera_overrides.tar.gz && \

# Development - to be removed
#RUN \
#  apt-get update && \
#  apt-get install -y xinit xterm ssh && \
#  rm -rf /var/lib/apt/lists/* && \
#  apt-get clean

CMD [ "bash" ]
