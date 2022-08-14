# tesseract_docker_image
This repository holds dockerfiles to build an image of the [Tesseract planning framework](https://github.com/tesseract-robotics/tesseract) from SWRI.

This container is not optimized for space and is fairly large in space. Users must be aware


## Dependencies
This repository uses GitHub actions to build a docker container and push it to the GitHub container registry

## Usage
In order to pull and use this container locally on your machine run
```bash
docker pull ghcr.io/srsidd/tesseract:galactic
```
The Tesseract workspace can be found under `/tesseract_ws/install/` in the container. The docker image does not hold of the source or build files to reduce the size, and only holds the install folder.

## License
This package contains code licensed under Apache-2.0, BSD-2-Clause and BSD-3-Clause.

The details of each license can be found in LICENSE.Apache-2.0, LICENSE.BSD-2-Clause and LICENSE.BSD-3-Clause
