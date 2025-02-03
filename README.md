# Keystone MobileNet Implementations

This repository contains two main directories demonstrating **MobileNet** in different environments:

1. **keystone-mobilenet-no-enclave**  
2. **keystone-mobilenet-enclave**

---

## 1. keystone-mobilenet-no-enclave

This folder contains a standard MobileNet implementation **without** using any Keystone enclaves.

- **Supported Depth Multipliers**: 0.7, 0.5, 0.25  
- **Description**:  
  - A baseline MobileNet model suitable for scenarios where hardware-backed security isolation (enclaves) is **not** required.  
  - Serves as a reference point for comparing performance and functionality against the enclave-based versions.
- Add this folder to your keystone/sdk/examples directory and add the following line to keystone/sdk/examples/CMakeLists
    ```bash
    add_subdirectory(mobilenet)
    ```

---

## 2. keystone-mobilenet-enclave

This folder contains Keystone-based MobileNet implementations split into enclaves for enhanced security. There are two primary variants:

### a. `mobilenet-split-three-0.7-hardcode-input`
- **Depth Multiplier**: 0.7  
- **Number of Enclaves**: 3  
  - (input enclave, intermediate enclave, output enclave)  
- **Input Handling**: The input image is **hardcoded** into the code.  
- **Use Case**: Demonstrates how to protect each stage of inference (from input to output) in its own enclave for maximum isolation.  
  - Hardcoding the input is primarily for demonstration or controlled experiments.
- Add this folder to your keystone/sdk/examples directory and add the following line to keystone/sdk/examples/CMakeLists
    ```bash
    add_subdirectory(mobilenet-split-three)
    ```

### b. `mobilenet-split-three`
- **Depth Multipliers**: 0.7, 0.5, 0.25  
- **Number of Enclaves**: 2  
  - (input enclave, output enclave)  
- **Intermediate Layers**: Run **outside** of an enclave.  
- **Input Handling**: Reads the input image from a file (not hardcoded).  
- **Use Case**: Offers a balance between security and flexibility, allowing certain computations (the intermediate layers) to occur outside the secure enclave.
- Add this folder to your keystone/sdk/examples directory and add the following line to keystone/sdk/examples/CMakeLists
    ```bash
    add_subdirectory(mobilenet-split-three)
    ```
---

## About Keystone Enclaves

[Keystone](https://keystone-enclave.org/) provides open-source, RISC-V-based hardware enclaves that allow you to securely run critical parts of your application. Splitting a model like MobileNet into multiple enclaves can help protect sensitive data and computations from untrusted components of the system.

---

## Depth Multipliers in MobileNet

MobileNet’s **depth multiplier** (often denoted as α) adjusts the width of the network—scaling the number of channels in each layer. Lower multipliers (e.g., 0.25 or 0.5) reduce the model size and computation cost (at the expense of some accuracy), while a higher multiplier (e.g., 0.7) increases accuracy with a larger model size.

---

## How to Build and Run - Keystone Setup

1. **Clone this repository** and navigate to the desired folder:

    ```bash
    git clone --recursive https://github.com/keystone-enclave/keystone.git
    cd keystone
    git checkout 4e96652
    ```
    Without the checkout, latest version of keystone is cloned so make sure to use checkout to clone docker configuration keystone

2. **Install dependencies**

    ```bash
    sudo apt update
    sudo apt install ninja-build
    sudo apt install autoconf automake autotools-dev bc \
    bison build-essential curl expat libexpat1-dev flex gawk gcc git \
    gperf libgmp-dev libmpc-dev libmpfr-dev libtool texinfo tmux \
    patchutils zlib1g-dev wget bzip2 patch vim-common lbzip2 python3 \
    pkg-config libglib2.0-dev libpixman-1-dev libssl-dev screen \
    device-tree-compiler expect makeself unzip cpio rsync cmake p7zip-full
    ```

3. **Setup All Environment Variables**:

    ```bash
    ./fast-setup.sh
    source source.sh
    mkdir build
    cd build
    cmake ..
    make
    make image
    ```
    ### Repeat Setup on Fresh Terminal
    If you close the terminal and start a new one, make sure to set up the environment variables again using
    ```bash
    source source.sh
    ```

4. **Build and Run the Package**
    In your build directory:
    ```bash
    make hello-package
    cp examples/hello/hello.ke ./overlay/root
    make image
    ./scripts/run-qemu.sh
    ```
    Result should show lines: Welcome to Buildroot
    ```bash
    buildroot login:
    ```
    Login as `root` with the password `sifive`. You can exit QEMU by `ctrl-a + x` or using `poweroff` command.

5. Test by Running Hello World
    ```bash
    insmod keystone-driver.ko
    ./hello.ke
    ```