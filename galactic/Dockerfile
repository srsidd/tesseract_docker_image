# -*- coding: UTF-8 -*-
# Dockerfile to create a base image required to run tesseract
# Author: Siddharth Srivatsa <srivaatsasiddharth@gmail.com>

# Build command
# docker build --build-arg ROSDISTRO=galactic --build-arg ROS_BASE_TAG=galactic -t tesseract:galactic .

ARG ROSDISTRO=galactic
ARG ROS_BASE_TAG=galactic

FROM ros:$ROS_BASE_TAG-ros-core
LABEL maintainer="Siddharth Srivatsa srivaatsasiddharth@gmail.com"

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
    apt-get install -y software-properties-common && \
    add-apt-repository ppa:ros-industrial/ppa -y && \
    apt-get install -y --no-install-recommends taskflow ros-galactic-rviz-default-plugins ros-galactic-interactive-markers ros-galactic-octomap ros-galactic-octomap-msgs ros-galactic-pcl-conversions liborocos-kdl-dev ros-galactic-tf2 ros-galactic-tf2-ros ros-galactic-tf2-eigen ros-galactic-urdf ros-galactic-cv-bridge ros-galactic-rviz-common && \
    rosdep install --from-paths src  --ignore-src -r -y \
    --skip-keys "catkin tesseract_plugins rviz plugin_loader rosidl_default_generators ament_cmake ament_lint_common pluginlib octomap ament_cmake_gtest ament_lint_common ros_testing tf2_eigen" && \
    python3 -m pip install -U install open3d && . /opt/ros/$ROSDISTRO/setup.sh && \
    colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --event-handlers terminal_title- console_cohesion+ && \
    rm -rf build log src

ADD entrypoint.sh /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh" ]
