name: Build GKI Kernel with KernelSU Next

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-22.04

    steps:
    - name: Checkout Kernel Source
      uses: actions/checkout@v4
      with:
        repository: google/kernel/common
        ref: android12-5.10-2024-08

    - name: Set up build environment (Clang & Tools)
      run: |
        sudo apt update
        sudo apt install -y build-essential libncurses5 bison flex libssl-dev python3 python3-pip
        pip install abootimg

    - name: Integrate KernelSU Next
      run: |
        cd common
        curl -LSs "https://raw.githubusercontent.com" | bash -
        echo "CONFIG_KSU=y" >> arch/arm64/configs/gki_defconfig
        echo "CONFIG_KSU_NEXT=y" >> arch/arm64/configs/gki_defconfig

    - name: Build the Kernel (GKI 5.10)
      run: |
        export ARCH=arm64
        export SUBARCH=arm64
        LTO=thin BUILD_CONFIG=common/build.config.gki.aarch64 build/build.sh

    - name: Upload Kernel Artifact
      uses: actions/upload-artifact@v4
      with:
        name: GKI-KernelSU-Boot-Image
        path: out/android12-5.10/dist/Image
