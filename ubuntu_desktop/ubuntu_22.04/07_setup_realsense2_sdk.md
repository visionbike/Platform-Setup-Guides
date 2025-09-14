# A Guide to Intel RealSense SDK 2.0 Setup on Ubuntu 22.04

**Intel RealSense** cameras are a family of Intel’s depth and tracking-capable vision modules designed to give machines the **3D perception** of their environment. They combine multiple sensors—typically an IR projector, a pair of stereo IR cameras, an RGB color camera, and on-board IMU—along with computer-vision algorithms provided by the `librealsense` SDK to output synchronized depth maps, point clouds, and RGB streams in real time.

The **Intel RealSense SDK 2.0** (`librealsense2`) is open-source, cross-platform software developement kit that enables developers to interface with Intel RealSense depth cameras. It provides the tools to stream, process, analyze 3D data, e.g., depth, color, infrared, IMU, from RealSense devices.

This tutorial aims to guide you to install the **pre-built** packages in **Ubuntu 22.04**.

## I. Prequisition

Ensure that your kernel machine matches the version supported by the `librealsense2-dkms` package if you are not installing from source. Please refer to the [offical documentation](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md) for more details.

The following steps must be completed before installing the `librealsense2` SDK:

1. Reboot the system into the UEFI:

```sh
systemtcl reboot --firmware-setup
```

2. Disable **Secure Boot** in the UEFI settings.

## II. RealSense SDK 2.0 Installation

Register the server's public key:

```sh
sudo mkdir -p /etc/apt/keyrings
curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | sudo tee /etc/apt/keyrings/librealsense.pgp > /dev/null
```

Ensure the apt HTTPS support installed:

```sh
sudo apt install -y apt-transport-https
```

Add the server to the list of repositories:

```sh
echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | \
sudo tee /etc/apt/sources.list.d/librealsense.list
sudo apt update -y
```

Install the Realsense2 SDKs:

```sh
sudo apt install -y librealsense2-dkms librealsense2-utils
```

The above two lines will deploy `librealsense2` udev rules, build and activate kernel modules, runtime library and executable demos and tools.

Optionally install the developer and debug packages:

```sh
sudo apt install -y librealsense2-dev librealsense2-dbg
```

With dev package installed, you can compile an application with `librealsense` using `g++ -std=c++11 filename.cpp -lrealsense2` or an IDE of your choice.

To verify the installation, reconnect the Intel RealSense depth camera and run:

```sh
realsense-viewer
```

![alt text](images/07_realsense_viewer.png)

Verify that the kernel is updated:

```sh
modinfo uvcvideo | grep "version:"
```

The `realsense` string should be included:

```txt
version:        1.1.1-realsense-1.3.28
srcversion:     62F07434F4A5D0D6EBBBD1D
```

## III. (Optional) Upgrading the Packages

Refresh local packages and upgrade all installed packages, including `librealsense2`:

```sh
sudo apt update -y && sudo apt upgrade -y
```

Upgrade `librealsense2`'s libraries:

```sh
sudo apt --only-upgrade install -y librealsense2-dkms librealsense2-utils
```

## III. (Optional) RealSense2 SDK Uninstallation

Removing the **Debian** package is allowed only when no other installed packages directly refer to it. For example, removing `librealsense2-udev-rules` requires `librealsense2` to be removed first.

Removing all RealSense2 SDK-related packages:

```sh
pkg -l | grep "realsense" | cut -d " " -f 3 | xargs sudo dpkg --purge
```

# IV. Examples

With the SDK installed and verified, you can now start programming. 

Create an samples folder for RealSense SDK:

```sh
mkdir -p ~/projects/realsense2_samples
```

The directory structure should be as bellows:

```txt
realsense2_samples
├── rs_test_cpp
│   ├── CMakeLists.txt
│   └── rs_test.cpp
├── rs_test_python
│   ├── requirements.txt
│   └── rs_test.py
```
### 1. C++ Example

The SDK comes with a rich set of C++ code examples. They are the best place to learn how to capture frames, align streams, and work with the data.

The following code snippet is simple program starts the camera, captures one frame of depth data, and prints its dimensions.

- `CMakeLists.txt` file:

```txt
cmake_minimum_required(VERSION 3.20)
project(rs_test_cpp)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find the librealsense2 package provided by the SDK installation
find_package(realsense2 REQUIRED)

# Add our executable
add_executable(rs_test rs_test.cpp)

# Link the executable against the librealsense2 library
# realsense2::realsense2 is the modern target to link against
target_link_libraries(rs_test PRIVATE realsense2::realsense2)
```

- `rs_test.cpp` file:

```cpp
#include <iostream>
#include <librealsense2/rs.hpp> // Include the RealSense Cross-Platform API

int main() {
    try {
        rs2::context ctx;
        std::cout << "librealsense2 - " << RS2_API_VERSION_STR << std::endl;
        std::cout << "You have " << ctx.query_devices().size() << " RealSense devices connected" << std::endl;
        // Create a Pipeline - this manages the device and streaming
        rs2::pipeline pipe;

        // Start streaming from the camera
        pipe.start();

        // Wait for the next set of frames from the camera
        rs2::frameset frames = pipe.wait_for_frames();

        // Get a depth frame
        rs2::depth_frame depth = frames.get_depth_frame();

        // Get the depth frame's dimensions
        float width = depth.get_width();
        float height = depth.get_height();

        std::cout << "Depth frame received!" << std::endl;
        std::cout << "Dimensions: " << width << " x " << height << std::endl;

        // Query the distance from the camera to the object in the center of the image
        float dist_to_center = depth.get_distance(int(width / 2), int(height / 2));

        // Print the distance
        std::cout << "The camera is facing an object " << dist_to_center << " meters away \n";

        return EXIT_SUCCESS;
    } catch (const rs2::error & e) {
        std::cerr << "RealSense error calling " << e.get_failed_function() << "(" << e.get_failed_args() << "):\n    " << e.what() << std::endl;
        return EXIT_FAILURE;
    } catch (const std::exception& e) {
        std::cerr << e.what() << std::endl;
        return EXIT_FAILURE;
    }
}
```

Create a `build` directory to keep things clean, configure and then compile the project:

```sh
# create a build directory
mkdir build && cd build

# configure the project
cmake ..

# compile the code
make

# run the executable
./rs_test
```

The output should be:

```txt
librealsense2 - 2.56.5
You have 1 RealSense devices connected
Depth frame received!
Dimensions: 848 x 480
The camera is facing an object 0.54 meters away 
```

#### Troubleshooting

In the RealSense SDK version `2.56.x`, you may meet the error as bellows:

```txt
CMake Error at CMakeLists.txt:5 (find_package):
  Found package configuration file:

    /usr/lib/x86_64-linux-gnu/cmake/realsense2/realsense2Config.cmake

  but it set realsense2_FOUND to FALSE so package "realsense2" is considered
  to be NOT FOUND.  Reason given by package:

  The following imported targets are referenced, but are missing: fastcdr
  fastrtps
```

To solve this problem, you can add:

```txt
find_package(fastrtps REQUIRED)
find_package(fastcdr REQUIRED)
```

in `/usr/lib/x86_64-linux-gnu/cmake/realsense2/realsense2Config.cmake` before this line: 

```txt
include("${CMAKE_CURRENT_LIST_DIR}/realsense2Targets.cmake")
```

### 2. Python Example

You first need to install the `pyrealsense2` wrapper, which allows you to control the camera and process its data using a simple anf powerful Python API.

```sh
pip install pyrealsense2 numpy
```

You can store packages installed in `requirements.txt` then install using the file, which is useful when setup a Python environment (native or virtual environment).

```
pip install -r requirements.txt
```

Execute the `rs_test.py` file:

```sh
python rs_test.py
```

The output should be:

```txt
Starting pipeline...
Depth frame received!
  - Shape: (480, 640)
  - Data type: uint16
Stopping pipeline...
```

## Conclusion

You have successfully installed the Intel RealSense SDK (librealsense2), bridging the gap between your powerful 3D camera hardware and your development environment. You are now ready to build applications that perceive the world in three dimensions, opening up incredible possibilities in robotics, augmented reality, 3D scanning, and more.