# A Guide to NVIDIA Driver, CUDA Toolkit, CUDNN and TensorRT Setup on Ubuntu 22.04

Setting up a powerful GPU development environment is essential for deep learning, AI research, and high-performance computing tasks. In this guide, you will walk through the step-by-step process to correctly install **NVIDIA CUDA**, **cuDNN**, and **TensorRT** libraries on **Ubuntu 22.04**.

This guide will cover both the driver installation and the manual setup of **CUDA** and **cuDNN** using `.run` and `.tar.xz` files, ensuring maximum flexibility and control — a method especially useful when fine-tuning configurations for machine learning frameworks like **TensorFlow** and **PyTorch**.

## I. NVIDIA Driver Installation

### 1. Checking if Available NVIDIA GPU is Detected

Before installing the driver, you should ensure that the system detects the NVIDA GPU.

Open a terminal and run command:

```sh
lspci | grep -i nvidia
```

The command lists all PCI devices and NVIDIA hardware. If the GPU is correctly detected, the output should be:

```txt
1:00.0 VGA compatible controller: NVIDIA Corporation Device 24e0 (rev a1)
01:00.1 Audio device: NVIDIA Corporation GA104 High Definition Audio Controller (rev a1)
```

### 2. Checking if NVIDIA Driver is Installed

You need check whether any NVIDIA drivers are already installed on the system.

```sh
dpkg -l | grep nvidia
```

The command shows all NVIDIA-related packages currently installed. If they are installed, you may still face conflicts with the default **Nouveau** driver.

The installed drivers should be uninstalled.

```sh
sudo apt remove --purge -y '.*nvidia.*'
sudo apt autoremove -y
sudo apt autoclean -y
```

Reboot the system.

```sh
sudo reboot
```

### 3. Installing Linux Kernel Header

Make sure headers and image files related to the current Linux kernel were installed.

```sh
sudo apt install -y linux-headers-$(uname -r)
```

Reboot the system.

```sh
sudo reboot
```

### 4. Blacklisting the Nouveau Driver

**Nouveau** is an open-source driver for NVIDIA cards, but it’s incompatible with the official NVIDIA driver. To avoid conflicts, you need to disable it.

Create a configuration file to blacklist the **Nouveau** driver:

```sh
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
```

Add these lines to the file:

```txt
blacklist nouveau
options nouveau modeset=0
```

Save and close the file by using these shorcuts `CTRL + S` and `CTRL + X`. Update the `inittramfs` to apply changes:

```sh
sudo update-initramfs -u
```

Reboot the system:

```sh
sudo reboot
```

### 5. Disabling Secure Boot

If Secure Boot is enabled, it can block the loading of unsigned drivers, including the NVIDIA proprietary drivers. To resolve this:

1. Reboot your system and enter the BIOS/UEFI settings (usually by pressing `F2`, `Delete`, or `Esc` during boot).

2. Locate the “Secure Boot” option in the BIOS and disable it.

3. Save and exit the BIOS.

### 6. Installing NVIDIA Driver

Before installing the NVIDIA driver, certain development tools should be installed first:

```sh
sudo apt update -y
sudo apt install -y build-essential gcc make dkms pkg-config libglvnd-dev curl wget git
```

Download the CMake's latest version [here](https://cmake.org/download/) and install it.

```sh
wget https://github.com/Kitware/CMake/releases/download/v4.1.0/cmake-4.1.0-linux-x86_64.sh

chmod +x cmake-4.1.0-linux-x86_64.sh

sudo ./cmake-4.1.0-linux-x86_64.sh --skip-licence --prefix=/usr/local --exclude-subdir

# check version
cmake --version
```

This will install the necessary compilers and build tools for driver installation.

Next, download the correct driver from official [NVIDIA website](https://www.nvidia.com/en-us/drivers/). Since for some specific support (e.g., NVIDIA Issac Sim), make sure install the NVIDIA driver from `*.run` file, **NOT** from **PPA** drivers with Ubuntu distribution. 

For Ubuntu 22.04.5 and current ussage, the latest driver `NVIDIA-Linux-x86_64-580.105.08.run` was downloaded.

The NVIDIA driver installation should be installed without `Xorg` graphic. Logout and go to TTY's mode (`CTRL + ALT + F3`).

Install the NVIDIA driver using the `*.run` file:

```sh
sudo chmod +x NVIDIA-Linux-x86_64-580.105.08.run
sudo ./NVIDIA-Linux-x86_64-580.105.08.run
```

Ignore errors related to missing 32-bit libraries, and build any missing library if it required a confirmation.

**IMPORTANT:** If you meet the problem that fails the NVIDIA driver installation, please read the problem in `/var/log/nvidia-installer.log` and resolve it before installing again.

Finally, reboot the system and access the graphic login screen.

```sh
sudo reboot
```

### 7. Driver Incompatability Issues (Black Screen Issues)

### Soluiton 1:

After finishing NVIDIA driver, if you meet the **black screen** issue, go to TTY's mode again and then delete `/etc/X11/xorg.conf`.

### Solution 2:

If the above solution can not be solved, you could access **recovery mode** then remove the NVIDIA driver installation and `/etc/modprobe.d/blacklist-nouveau.conf` file.

### 8. Verifying Installation

When logging in, run `nvidia-smi` to verify the installation:

```sh
nvidia-smi
```

The output should be like:

```txt
Thu Nov 27 04:45:22 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.105.08             Driver Version: 580.105.08     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3070 ...    Off |   00000000:01:00.0  On |                  N/A |
| N/A   53C    P5              9W /  115W |     245MiB /   8188MiB |      5%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            5775      G   /usr/lib/xorg/Xorg                      132MiB |
+-----------------------------------------------------------------------------------------+
```

The driver version also can be found by executing the command:

```sh
cat /proc/driver/nvidia/version
```

The output is shown like:

```txt
NVRM version: NVIDIA UNIX x86_64 Kernel Module  580.105.08  Wed Oct 29 23:15:11 UTC 2025
GCC version:  gcc version 12.3.0 (Ubuntu 12.3.0-1ubuntu1~22.04.2) 
```

### 9. Preventing NVIDIA Driver from Automatically Upgrading

Ubuntu’s **unattended-upgrades** service automatically installs security updates and critical patches in the background. By default, it may include NVIDIA packages unless explicitly excluded.
To prevent it from upgrading NVIDIA packages, you can edit the **unattended-upgrades** configuration.

```sh
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

To add NVIDIA packages to the blacklist, you find the section `Unattended-Upgrade::Package-Blacklist { ... };` and add `.*nvidia-.*` to block all NVIDIA-related packages

```txt
Unattended-Upgrade::Package-Blacklist {
    ".*nvidia-.*";  // Block all NVIDIA packages
    "linux-image-*";  // Optional: Block kernel upgrades (if needed)
    // ...
};
```

Save the file and then run a dry-run of `unattended-upgrades` to verify:

```sh
sudo unattended-upgrades --dry-run --debug
```

Ensure no NVIDIA packages are listed as "To upgrade."

### 10. (Optional) Uninstalling NVIDIA Driver

Similar to the installation step, switch to a text-only console (**no GUI**) by go to TTY's mode (`CTRL + ALT + F3`). 

Then run the NVIDIA `.run` installer with `--uninstall` option.

E.g.,

```sh
sudo bash NVIDIA-Linux-x86_64-580.105.08.run --uninstall
```

You need to use the *same* `.run` installer (or download it again).

## II. NVIDIA CUDA Toolkit Installation

### 1. Installing NVIDIA CUDA Toolkit

If the NVIDIA driver was installed using `*.run` file, e.g., `NVIDIA-Linux-x86_64-580.105.08.run`, the CUDA Toolkit installation method should be *chosen carefully* using **local** `.run` **installer for CUDA**. 

For current ussage, the **CUDA Toolkit 13.0.2** is selected and downloaded.

```sh
wget https://developer.download.nvidia.com/compute/cuda/13.0.2/local_installers/cuda_13.0.2_580.95.05_linux.run
```

Similar installing NVIDIA driver, logout and go to TTY's mode (`CTRL + ALT + F3`).

```sh
sudo chmod +x cuda_13.0.2_580.95.05_linux.run
sudo ./cuda_13.0.2_580.95.05_linux.run
```

When you run it, it asks *whether you want to install the driver*. You must choose **UNSELECT** when it asks to install the driver, because you already *manually installed the driver yourself*. If you install driver again, it might *overwrite or cause problems*.

### 2. Post-installing CUDA Toolkit

Before CUDA Toolkit and driver can be used, the environment setup is needed. Open `~/.bashrc`.

```sh
nano ~/.bashrc
```

Add these lines to the end of the file.

```sh
# >>> CUDA Toolkit environment setup >>>
export PATH=/usr/local/cuda-13.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_PATH=/usr/local/cuda-13.0
# <<< CUDA Toolkit environment setup <<<
```

Save and close the file by using these shorcuts `CTRL + S` and `CTRL + X`. Activate the changes in `~/.bashrc` file.

```sh
source ~/.bashrc
```

To verify the setup, run `nvcc -V` to check.

```sh
nvcc -V
```

- The output should be:

```txt
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Wed_Aug_20_01:58:59_PM_PDT_2025
Cuda compilation tools, release 13.0, V13.0.88
Build cuda_13.0.r13.0/compiler.36424714_0
```

NVIDIA also provides a user-space daemon on Linux to support persistence of driver state across CUDA job runs. The **Persistence Mode** keeps a target GPU initialized even when no clients are connected to it. The **NVIDIA Persistence Daemon** can be started as the root user by running:

```sh
sudo -i

nvidia-smi -pm 1

exit
```

### 3. Running Examples

CUDA samples are now located in [https://github.com/nvidia/cuda-samples](https://github.com/nvidia/cuda-samples), which includes instructions for obtaining, building, and running the samples.

Download the `cuda-samples` for `CUDA 13.0` using `git` command.

```sh
git clone --branch v13.0 --single-branch https://github.com/nvidia/cuda-samples.git cuda_samples
```

Go to `Samples/1_Utilities/deviceQuery` and compile.

```sh
cd Samples/1_Utilities/deviceQuery && mkdir build && cd build

cmake ..

make -j $(nproc)
```

After compilation, run `deviceQuery`.

```sh
./deviceQuery
```

The output should be like:

```txt
./deviceQuery Starting...

CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "NVIDIA GeForce RTX 3070 Ti Laptop GPU"
CUDA Driver Version / Runtime Version          13.0 / 13.0
CUDA Capability Major/Minor version number:    8.9
Total amount of global memory:                 7851 MBytes (8232435712 bytes)
(046) Multiprocessors, (128) CUDA Cores/MP:    5888 CUDA Cores
GPU Max Clock rate:                            1410 MHz (1.41 GHz)
Memory Clock rate:                             7001 Mhz
Memory Bus Width:                              256-bit
L2 Cache Size:                                 4194304 bytes
Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
Total amount of constant memory:               65536 bytes
Total amount of shared memory per block:       49152 bytes
Total shared memory per multiprocessor:        102400 bytes
Total number of registers available per block: 65536
Warp size:                                     32
Maximum number of threads per multiprocessor:  1536
Maximum number of threads per block:           1024
Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
Maximum memory pitch:                          2147483647 bytes
Texture alignment:                             512 bytes
Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
Run time limit on kernels:                     Yes
Integrated GPU sharing Host Memory:            No
Support host page-locked memory mapping:       Yes
Alignment requirement for Surfaces:            Yes
Device has ECC support:                        Disabled
Device supports Unified Addressing (UVA):      Yes
Device supports Managed Memory:                Yes
Device supports Compute Preemption:            Yes
Supports Cooperative Kernel Launch:            Yes
Supports MultiDevice Co-op Kernel Launch:      Yes
Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
Compute Mode:
    < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 13.0, CUDA Runtime Version = 13.0, NumDevs = 1
Result = PASS
```

Some CUDA samples use third-party libraries which may not be installed be default on the system.

```sh
sudo apt install -y g++ libfreeimage3 freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libglu1-mesa-dev libfreeimage-dev libglfw3-dev
```

For GPU rendering, you should add `__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia`.

E.g.,

```sh
cd Samples/5_Domain_Specific/simpleGL && mkdir build && cd build

cmake .. 

make -j $(nproc)
```

Run the program.

```sh
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia ./simpleGL
```

- The output should be:

![alt text](images/01_simplegl_result.png)

### 4. (Optional) Uninstalling CUDA Toolkit

If you use `*.run` file method to install CUDA Toolkit, the bellows will be the clean and safe way to uninstall CUDA.

Find the CUDA installation directory in `/usr/local/`, e.g., `/usr/local/cuda-13.0`.

Run the uninstall script inside CUDA folder.

```sh
sudo /usr/local/cuda-13.0/bin/cuda-uninstaller
```

It will ask for confirmation and then remove all installed files. Then, open `~/.bashrc` and remove manually CUDA-related environment variables. Save and then source changes.

## III. cuDNN Backend Installation

### 1. Installing cuDNN Backend

In case of installing CUDA manually by `.run` file, the *local installation* method is a clean and safe way.

Go to [NVIDIA cuDNN Downloads](https://developer.nvidia.com/cudnn-downloads). You need an NVIDIA Developer account (free).

Download the **cuDNN Library for Linux** (`.tar.xz`) file.

- Choose: 
    - cuDNN Version: **cuDNN 9.16.0**
    - Operating System: **Linux**
    - Architecture: **x86_64**
    - Distribution: **Tarball** 
    - CUDA Version: **13**.

E.g., 
```sh
wget https://developer.download.nvidia.com/compute/cudnn/redist/cudnn/linux-x86_64/cudnn-linux-x86_64-9.16.0.29_cuda13-archive.tar.xz
```

Extract the downloaded archive.

```sh
tar -xvf cudnn-linux-x86_64-9.16.0.29_cuda13-archive.tar.xz
```

You will get folders like `include/`, `lib/`, etc. Then copy cuDNN files into CUDA directory.

E.g.,
```sh
sudo cp ./include/cudnn*.h /usr/local/cuda-13.0/include
sudo cp -P ./lib/libcudnn* /usr/local/cuda-13.0/lib64
```

Set permissions the copied files.

```sh
sudo chmod a+r /usr/local/cuda-13.0/include/cudnn*.h /usr/local/cuda-13.0/lib64/libcudnn*
```

Verify cuDNN version:

```sh
cat /usr/local/cuda-13.0/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

- The output should be:

```txt
#define CUDNN_MAJOR 9
#define CUDNN_MINOR 16
#define CUDNN_PATCHLEVEL 0
--
#define CUDNN_VERSION (CUDNN_MAJOR * 10000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

/* cannot use constexpr here since this is a C-only file */
```

### 2. Running Examples

To make sure that the installation worked, you can run some samples provided by NVIDIA. In cuDNN version 9.x, cudnn samples are not provided. You can download the samples from [this](https://developer.download.nvidia.com/compute/cudnn/redist/cudnn_samples/source/).

```sh
wget https://developer.download.nvidia.com/compute/cudnn/redist/cudnn_samples/source/cudnn_samples-source-9.16.0.29-archive.tar.xz
```

Extract to `cudnn_samples` folder.

```sh
mkdir -p ~/projects/cudnn_samples

tar -xvf cudnn_samples-source-9.16.0.29-archive.tar.xz -C ~/projects/cudnn_samples --strip-components 3
```

To compile `mnistCUDNN` sample, run the command:

```sh
cd cudnn_samples/mnistCUDNN

make -j $(nproc)
```

Run the program.

```sh
./mnistCUDNN
```

You should get a bunch of output with the last line saying **Test passed!**.

To compile `conv_sample` sample, run the command:

```sh
cd cudnn_samples/conv_sample

make -j $(nproc)
```

Run the program.

```sh
bash run_conv_sample.sh
```

You would get last line saying **Test PASSED**.

## IV. TensorRT Installation

For CUDA manual installation, the **Tarball** method (`.tar.gz`) is a safe way to install TensorRT.

### 1. Installing TensorRT

Go to [https://developer.nvidia.com/tensorrt/download](https://developer.nvidia.com/tensorrt/download), then select **TensorRT 10**.

- Choose the latest **TensorRT 10.14.1 GA for x86_64 Architecture**, and select **TAR** package for **CUDA** 13.0.

- Download the **TensorRT 10.14.1 GA** (`*.tar.gz`) file.

E.g.,

```sh
wget -c https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/10.14.1/tars/TensorRT-10.14.1.48.Linux.x86_64-gnu.cuda-13.0.tar.gz
```

Create a directory in `/opt` for TensorRT. It's good practice to include the version in the name.

```sh
sudo mkdir -p /opt/tensorrt-10.14.1
```

Give all permissions to the `/opt/tensorrt-10.14.1` directory in Linux.

```sh
sudo chmod -R 777 /opt/tensorrt-10.14.1
```

Extract the tarball directly into your new directory. Using `--strip-components=1` is a useful trick to rmeove the top-level folder from the archive, resulting in a cleaner path.

```sh
tar -hxzvf TensorRT-10.14.1.48.Linux.x86_64-gnu.cuda-13.0.tar.gz -C /opt/tensorrt-10.14.1 --strip-components=1
```

After extracting, TensorRT files will be located in `bin/`, `include/`, `lib/`, etc. For the system and compiler to find the TensorRT libraries and executables, you must add its location to your environemnt variables (`.bashrc` file).

```sh
# >>> TensorRT environment setup >>>
export PATH=/opt/tensorrt-10.14.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/opt/tensorrt-10.14.1/lib${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export TRT_LIBPATH=/opt/tensorrt-10.14.1/lib
# <<< TensorRT environment setup <<<
```

Apply the changes by "sourcing" the `.bashrc` file.

```sh
source ~/.bashrc
```
### 2. Running TensorRT Examples

You can quickly check that everything is working by running the trtexec command, which is TensorRT's command-line tool.

```sh
trtexec --help
```
For running examples, you can copu entire `samples` directory from the TensorRT installation in `/opt` to you home directory, where you have write permissions.
E.g.,

```sh
git clone --branch v10.14 --single-branch https://github.com/NVIDIA/TensorRT.git tensorrt_samples
cd tensorrt_samples
git submodule update --init --recursive
```

Many samples require the TensorRT sample data package. From `v10.14`, the sample data is excluded from the tarball. You can download data from [TensorRT GitHub Releases](https://github.com/NVIDIA/TensorRT/releases).

```sh
unzip tensorrt_sample_data_20251106.zip
mkdir -p /opt/tensorrt-10.14.1/data
cp -r tensorrt_sample_data_20251106/* /opt/tensorrt-10.14.1/data
```

Add the following environment variable in `.bashrc` then source.

```sh
export TRT_DATADIR=/opt/tensorrt-10.14.1/data
```

For C++ samples, build all samples:

```sh
cd tensorrt_samples
mkdir -p build && cd build
cmake .. -DTRT_LIB_DIR=$TRT_LIBPATH -DTRT_OUT_DIR=./out
make -j $(nproc)
```

Run the `sample_onnx_mnist` sample.

```sh
cd ../build/samples/sampleOnnxMNIST/out && ./sample_onnx_mnist --datadir=$TRT_DATADIR/mnist --int8
```

The output should be as follows:

```txt
&&&& RUNNING TensorRT.sample_onnx_mnist [TensorRT v101401] [b48] # ./sample_onnx_mnist --datadir=/opt/tensorrt-10.14.1/data/mnist --int8
[11/27/2025-05:30:15] [I] Building and running a GPU inference engine for Onnx MNIST
[11/27/2025-05:30:16] [I] [TRT] [MemUsageChange] Init CUDA: CPU +26, GPU +0, now: CPU 31, GPU 445 (MiB)
[11/27/2025-05:30:16] [I] [TRT] ----------------------------------------------------------------
[11/27/2025-05:30:16] [I] [TRT] Input filename:   /opt/tensorrt-10.14.1/data/mnist/mnist.onnx
[11/27/2025-05:30:16] [I] [TRT] ONNX IR version:  0.0.3
[11/27/2025-05:30:16] [I] [TRT] Opset version:    8
[11/27/2025-05:30:16] [I] [TRT] Producer name:    CNTK
[11/27/2025-05:30:16] [I] [TRT] Producer version: 2.5.1
[11/27/2025-05:30:16] [I] [TRT] Domain:           ai.cntk
[11/27/2025-05:30:16] [I] [TRT] Model version:    1
[11/27/2025-05:30:16] [I] [TRT] Doc string:       
[11/27/2025-05:30:16] [I] [TRT] ----------------------------------------------------------------
[11/27/2025-05:30:16] [I] [TRT] [MemUsageChange] Init builder kernel library: CPU +204, GPU +4, now: CPU 433, GPU 449 (MiB)
[11/27/2025-05:30:16] [W] [TRT] Calibrator is not being used. Users must provide dynamic range for all tensors that are not Int32 or Bool.
[11/27/2025-05:30:16] [I] [TRT] Local timing cache in use. Profiling results in this builder pass will not be stored.
[11/27/2025-05:30:20] [I] [TRT] Detected 1 inputs and 1 output network tensors.
[11/27/2025-05:30:20] [I] [TRT] Total Host Persistent Memory: 26432 bytes
[11/27/2025-05:30:20] [I] [TRT] Total Device Persistent Memory: 0 bytes
[11/27/2025-05:30:20] [I] [TRT] Max Scratch Memory: 0 bytes
[11/27/2025-05:30:20] [I] [TRT] [BlockAssignment] Started assigning block shifts. This will take 6 steps to complete.
[11/27/2025-05:30:20] [I] [TRT] [BlockAssignment] Algorithm ShiftNTopDown took 0.009647ms to assign 3 blocks to 6 nodes requiring 32256 bytes.
[11/27/2025-05:30:20] [I] [TRT] Total Activation Memory: 31744 bytes
[11/27/2025-05:30:20] [I] [TRT] Total Weights Memory: 34308 bytes
[11/27/2025-05:30:20] [I] [TRT] Engine generation completed in 4.31297 seconds.
[11/27/2025-05:30:20] [I] [TRT] [MemUsageStats] Peak memory usage of TRT CPU/GPU memory allocators: CPU 0 MiB, GPU 5 MiB
[11/27/2025-05:30:20] [I] [TRT] Loaded engine size: 0 MiB
[11/27/2025-05:30:20] [I] [TRT] [MemUsageChange] TensorRT-managed allocation in IExecutionContext creation: CPU +0, GPU +0, now: CPU 0, GPU 0 (MiB)
[11/27/2025-05:30:20] [I] Input:
[11/27/2025-05:30:20] [I] @@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@%.-@@@@@@@@@@@
@@@@@@@@@@@*-    %@@@@@@@@@@
@@@@@@@@@@= .-.  *@@@@@@@@@@
@@@@@@@@@= +@@@  *@@@@@@@@@@
@@@@@@@@* =@@@@  %@@@@@@@@@@
@@@@@@@@..@@@@%  @@@@@@@@@@@
@@@@@@@# *@@@@-  @@@@@@@@@@@
@@@@@@@: @@@@%   @@@@@@@@@@@
@@@@@@@: @@@@-   @@@@@@@@@@@
@@@@@@@: =+*= +: *@@@@@@@@@@
@@@@@@@*.    +@: *@@@@@@@@@@
@@@@@@@@%#**#@@: *@@@@@@@@@@
@@@@@@@@@@@@@@@: -@@@@@@@@@@
@@@@@@@@@@@@@@@+ :@@@@@@@@@@
@@@@@@@@@@@@@@@*  @@@@@@@@@@
@@@@@@@@@@@@@@@@  %@@@@@@@@@
@@@@@@@@@@@@@@@@  #@@@@@@@@@
@@@@@@@@@@@@@@@@: +@@@@@@@@@
@@@@@@@@@@@@@@@@- +@@@@@@@@@
@@@@@@@@@@@@@@@@*:%@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@

[11/27/2025-05:30:20] [I] Output:
[11/27/2025-05:30:20] [I]  Prob 0  0.0000 Class 0: 
[11/27/2025-05:30:20] [I]  Prob 1  0.0000 Class 1: 
[11/27/2025-05:30:20] [I]  Prob 2  0.0000 Class 2: 
[11/27/2025-05:30:20] [I]  Prob 3  0.0000 Class 3: 
[11/27/2025-05:30:20] [I]  Prob 4  0.0004 Class 4: 
[11/27/2025-05:30:20] [I]  Prob 5  0.0000 Class 5: 
[11/27/2025-05:30:20] [I]  Prob 6  0.0000 Class 6: 
[11/27/2025-05:30:20] [I]  Prob 7  0.0000 Class 7: 
[11/27/2025-05:30:20] [I]  Prob 8  0.0000 Class 8: 
[11/27/2025-05:30:20] [I]  Prob 9  0.9996 Class 9: **********
[11/27/2025-05:30:20] [I] 
&&&& PASSED TensorRT.sample_onnx_mnist [TensorRT v101401] [b48] # ./sample_onnx_mnist --datadir=/opt/tensorrt-10.14.1/data/mnist --int8
```

For Python samples, the process involve the following steps:

1. Setup the python environment using `requirements.txt` file in `samples/python/`. 

- You could use the `virtual` or `conda` environment to setup. In this guide, `mamba` from [Miniforge](https://github.com/conda-forge/miniforge) was used.

```sh
mamba create -n tensorrt_samples python=3.11
mamba activate tensorrt_samples
```

- Remember install `tensorrt-*-cp3x-none-linux_x86_64.whl` in `/opt/tensorrt-10.14.1/python/` directory first. 

```sh
pip install /opt/tensorrt-10.14.1/python/tensorrt-*-cp311-none-linux_x86_64.whl
```

- Install the `samples/python/requirements.txt` file.

```sh
pip install -r requirements.txt
```

2. Go to the specific sample folder, e.g., `introductory_parser_samples`, then install the `requirements.txt`.

```sh
cd introductory_parser_samples/ && pip install -r requirements.txt
```

3. Run the program.

```sh
python onnx_resnet50.py -d $TRT_DATADIR
```

- The output should be shown as bellows:

```sh
Correctly recognized /opt/tensorrt-10.14.1/data/tabby_tiger_cat.jpg as tiger cat
```

## Conclusion

Your machine is now a powerful platform for AI development and accelerated computing. The next step is to install a GPU-accelerated library like PyTorch or TensorFlow. From there, you can start training models, running inference, and exploring the performance gains from your new setup.
