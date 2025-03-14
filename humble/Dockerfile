# -*- coding: UTF-8 -*-
# Dockerfile to create a base image required to run tesseract
# Author: Siddharth Srivatsa <srivaatsasiddharth@gmail.com>

# Build command
# docker build --build-arg ROSDISTRO=galactic --build-arg ROS_BASE_TAG=galactic -t tesseract:galactic .

ARG ROSDISTRO=humble
ARG ROS_BASE_TAG=humble

FROM ros:$ROS_BASE_TAG-ros-core
LABEL maintainer="Siddharth Srivatsa srivatsasiddharth@gmail.com"

ARG ROSDISTRO
ARG DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    openssh-client \
    python3-pip \
    python3-vcstool \
    python3-rosdep \
    python3-colcon-common-extensions \
    libassimp-dev \
    libompl-dev \
    build-essential \
    git \
    wget \
    libbullet-dev \
    libbullet-extras-dev \
    libxmlrpcpp-dev && \
    apt-get clean && \
    rosdep init && rosdep update && \
    apt-get update && rm -rf /var/lib/apt/lists/*

# Add ssh credentials to the build
ARG SSH_PRIVATE_KEY

# TODO: Pass private key from a github secret
RUN mkdir ~/.ssh/ && \
    echo "${SSH_PRIVATE_KEY}" > ~/.ssh/id_ed25519 && \
    chmod 400 ~/.ssh/id_ed25519 && \
    cat ~/.ssh/id_ed25519 && \
    # Ensure your domain is accepted
    touch ~/.ssh/known_hosts && \
    eval $(ssh-agent) && \
    ssh-keyscan github.com >> ~/.ssh/known_hosts

WORKDIR /tesseract_ws/src
COPY dependencies.repos /tesseract_ws/
# Setup Tesseract dependencies
RUN vcs import < /tesseract_ws/dependencies.repos && \
    # Setup Argus dependencies
    cd .. && apt-get update && \
    apt-get install -y software-properties-common qt3d5-dev libccd-dev libboost-filesystem-dev libboost-system-dev libboost-serialization-dev ros-humble-octomap ros-humble-tf2-eigen coinor-libipopt-dev && \
    add-apt-repository ppa:ros-industrial/ppa -y && \
    python3 -m pip install open3d transforms3d && \
    rosdep install --from-paths src  --ignore-src -r -y \
    --skip-keys "catkin tesseract_plugins libvtk rviz plugin_loader rosidl_default_generators ament_cmake ament_lint_common pluginlib octomap ament_cmake_gtest ament_lint_common ros_testing tf2_eigen" && \
    . /opt/ros/$ROSDISTRO/setup.sh && \
    colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release -DTESSERACT_BUILD_FCL=OFF -DBUILD_RENDERING=OFF -DBUILD_STUDIO=OFF  --event-handlers terminal_title- console_cohesion+ && \
    rm -rf build log src

RUN cd ~ && git clone --depth 1 --branch v0.17.0 --recursive https://github.com/intel-isl/Open3D.git && /bin/bash ~/Open3D/util/install_deps_ubuntu.sh assume-yes && cd ~/Open3D && mkdir build && cd build && \
    # Specify install directory to be same as install dir of your workspace
    cmake -DBUILD_SHARED_LIBS=ON -DGLIBCXX_USE_CXX11_ABI=ON -DCMAKE_BUILD_TYPE=Release .. && \
    make install && rm -rf ~/Open3D

ADD entrypoint.sh /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh" ]
