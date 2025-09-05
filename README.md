# Platform-Setup-Guides

This repository is to provide clear, step-by-step instructions to configure development machines from a clean OS install to ready-to-use state for machine learning, robotics, computer vision and genrative AI development.

## Support Configuration

This table outlines the currently available setup guides.

| Hardware Platform | Operating System / SDK |
| :---------------- | :--------------------- |
| Ubuntu Desktop (x86_64) | Ubuntu 22.04 LTS |
| NVIDIA Jetson Orin Nano | NVIDIA JetPack 6.2.1 |

## Directory Structure

The repository is organized by `hardware-platform` and then by `os-version`. Each directory contains numberous markdown files that should be followed in order.

Example:

```txt
platform-setup-guides/
├── ubuntu_desktop/
│   └── ubuntu_22.04/
│       ├── 01_nvidia_driver_cuda_cudnn_and_tensorrt.md
│       └── ...
├── jetson_orin_nano/
│   └── jetpack_5.1/
│       ├── ...
├── scripts/
│   └── ... (automation scripts for various steps)
├── LICENSE
└── README.md
```

## How to Contribute

Contributions are welcome! If you have a setup guide for a new platform or a suggestion for an existing one, please follow these steps:

1. Fork the repository.

2. Create a new branch for your changes (`git checkout -b feature/add-jetson-xavier-guide`).

3. Add your guide(s) following the existing directory structure.

4. Submit a pull request with a clear description of your changes.

## License

This project is license under the MIT License. See the `LICENSE` file for details.
