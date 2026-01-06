# Darknet CUDA Podman compose

This repository provides,
* Podman-based container environment for building and running Darknet with CUDA support
* A simple Jupyter Notebook for validation.
* A Training code for Seaship Dataset

## Notes

The following fields in `compose.yaml` must be adjusted for your environment:
* `devices`
* `ports`
* `volumes`

## Specification
* JupyterLab Server
  * JupyterLab server container for Darknet and DarkHelp testing.
  * Access: http://127.0.0.1:8888
* Env
  * NVIDIA CUDA 12.8
  * cuDNN
  * Ubuntu 24.04
* Darknet, DarkHelp
  * CUDA enabled
  * Multi-architecture (sm_75, sm_80, sm_86, sm_89, sm_90, sm_120)

## Files
### Dockerfile
* Builds a CUDA-enabled Darknet container image

### compose.yaml
* Runs the Darknet CUDA container using Podman Compose with GPU support.
  ```
   podman compose up -d
  ```

### darknet_workspace/darknet_cuda_check.ipynb
* Jupyter Notebook for verifying the Darknet CUDA build and runtime environment.

## Issues and Resolutions
### Can't Packaging (CPack error)
* Do not package Darknet and DarkHelp
  * Run `make` and `make install`. It's OK
* Built and installed directly from source

### nvcc warning: Cannot find valid GPU for '-arch=native'
* Why
  * GPU is not available during image build
  * Podman Compose builds without GPU access
  * `-arch=native` falls back to a default architecture
* How to solve
  * Use explicit multi-architecture flags
    ```
    -DCMAKE_CUDA_FLAGS="--generate-code=arch=compute_75,code=sm_75 \
                        --generate-code=arch=compute_80,code=sm_80 \
                        --generate-code=arch=compute_86,code=sm_86 \
                        --generate-code=arch=compute_89,code=sm_89 \
                        --generate-code=arch=compute_90,code=sm_90 \
                        --generate-code=arch=compute_120,code=sm_120" 
    ```
    * ⚠️ Warning may still appear, **but specified flags are still applied**
* Verification
  * See the darknet_cuda_check.ipynb notebook

### libcuda.so not found
* Why
  * NVIDIA driver API is not available during image build
* How to solve
  * Use the CUDA stub library provided by NVIDIA
  * Temporarily create a symbolic link to `libcuda.so` during build
    ```
    ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/lib/libcuda.so.1
    ```
  * Remove the symbolic link after build completion
    ```
    rm -f /usr/lib/libcuda.so.1
    ```
* Notes
  * Stub is used for **build-time linking only**
  * Runtime uses the actual driver library provided by the host
  * See the darknet_cuda_check.ipynb notebook for checking Darknet and DarkHelp working
