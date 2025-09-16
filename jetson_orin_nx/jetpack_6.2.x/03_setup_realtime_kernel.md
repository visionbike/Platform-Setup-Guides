# A Guide to Real-time Kernel Setup on Jetson Orin NX with JetPack 6

For many robotics, industrial control, and high-speed data acquisition tasks, standard operating systems are not enough. While the default Linux kernel on your Jetson Orin NX is optimized for high throughput, it doesn't guarantee when a task will execute. 

This guide will walk you through the advanced process of installing a **Real-Time Kernel** on your **Jetson Orin NX running JetPack 6**. This will transform your device into a low-latency powerhouse capable of meeting strict timing deadlines.

## I. What is a Real-time Kernel?

A **Real-Time Kernel** (`PREEMPT_RT`) is a modified version of the Linux kernel that allows high-priority tasks to interrupt, or "preempt," lower-priority ones almost instantly. This dramatically reduces latency and makes the system's timing behavior predictable and deterministic, which is essential for robotics.

From JetPack with release `35.1`, **Real-Time Kernel** support is provided for the following platforms:

- Jetson Thor.
- Jetson Orin AGX series.
- Jetson Orin NX and Nano series.

You can install **Real-Time Kernel** by using OTA update.

### Prerequisites:

- A Jetson Orin NX running **JetPack 6.2.1**.
- A reliable internet connection.

## II. Real-Time Kernel Setup

### 1. Installing Real-Time (RT) Kernel

The **Real-Time (RT) Kernel** can be installed with Debian package management-based OTA on Jetsone Device.

Open the `apt` source configuration file.

```sh
sudo nano /etc/apt/sources.list.d/nvidia-l4t-apt-source.list
```

Add the RT Kernel repository.

```sh
deb https://repo.download.nvidia.com/jetson/rt-kernel <release> main
```

You replace `<release>` with the release number you want to update, i.e., `r36.4` for updating to release `36.4`.

Save and close the source configuration file. Then update the system:

```sh
sudo apt update -y
```

Install RT kernel packages.

```sh
sudo apt install -y nvidia-l4t-rt-kernel nvidia-l4t-rt-kernel-headers nvidia-l4t-rt-kernel-oot-modules nvidia-l4t-display-rt-kernel
```

Reboot the Jetson device.

```sh
sudo reboot
```

After installing the RT Kernel on Jetson device,you can switch the install RT Kernel image and the original generic kernel image without re-installing or removing kernel packages. You can do this by editing `/boot/extlinux/extlinux.conf` and change `DEFAULT` property to `real-time`.

```sh
TIMEOUT 30
DEFAULT real-time
```

### 2. Tuning for Low-latency Performance

After installing the RT kernel, you can apply further optimizations to minimize latency. These are typically implemented on the target Jetson device. 

#### Adding real-time parameters for Isolating CPU Cores

These parameters are used to configure a Linux system for predictable, low-latency performance required by time-sensitive applications like robotics, and industrial automation.

Check the CPU core in the Jetson device

```sh
nproc
```

In Jetson Orin NX, the number of CPU cores is `8`. This value varies in different devices.

Open `/boot/extlinux/extlinux.conf` file.

```sh
sudo nano `/boot/extlinux/extlinux.conf`
```

Locate the `real-time` label in the file.

```txt
LABEL real-time
    MENU LABEL real-time kernel
    LINUX /boot/Image.real-time
    INITRD /boot/initrd
    APPEND ${cbootargs} ...
```

Based on the number of CPU cores, add these parameters in the `APPEND` line.

1. **rcu_nocb_poll:** Enables polling mode for RCU callbacks on isolated CPUs.
2. **rcu_nocbs=<cpu_ids>:** Specifies the range of CPUs to use for RCU callbacks (adjust to match your core isolation).
3. **nohz=on:** Enables tickless mode for the kernel.
4. **nohz_full=<cpu_ids>:** Enables full tickless mode for the specified CPUs, isolating them from system timer interrupts.
5. **kthread_cpus=<cpu_ids>:** Confines kernel threads to the specified CPUs, leaving others free.
6. **irqaffinity=<cpu_ids>:** Pins all device interrupts to the specified CPUs.
7. **isolcpus=managed_irq,domain,<cpu_ids>:** Isolates a range of CPUs from kernel scheduling and interrupts, with domain isolation.

**Note:** The CPU numbers and ranges must be adjusted based on the total number of cores on your specific Jetson model.

E.g.,

```sh
APPEND ${cbootargs} ... rcu_nocb_poll rcu_nocbs=4-7 nohz=on nohz_full=4-7 kthread_cpus=0,1,2,3 irqaffinity=0,1,2,3 isolcpus=managed_irq,domain,4-7
```

Save changes and reboot the system.

```sh
sudo reboot
```

#### Disabling Real-Time Throttling

**Real-Time (RT) throttling** is designed to limit the execution time of high-priority tasks to prevent them from consuming all CPU resources. However, for critical real-time applications, this limitation is counterproductive, as it can delay or interrupt the task's required execution. 

On a Jetson, especially when using the real-time kernel, high-priority, high-usage processes can hog the CPU, leading to instability. Disabling throttling allows these critical processes to run to completion without being preempted by the throttling mechanism, thus preventing potential kernel crashes.

To set the RT throttling persistently, adding it to a configuration file in `/etc/sysctl.d/`.

Create `/etc/sysctl.d/99-realtime.conf`.

```sh
sudo nano /etc/sysctl.d/99-realtime.conf
```

Add the setting to the file:

```txt
# Disable real-time task CPU time throttling
kernel.sched_rt_runtime_us = -1

# Disable real-time CPU runtime throttling
kernel.timer_migration = 0
```

Apply changes.

```sh
sudo sysctl --system
```

By following these steps, the real-time throttling will be disabled automatically on every system boot.

#### Setting up Power Mode and Consistent Clock Speeds

Please see the guide in Section `I. Setting Jetson Performance` in [02_setup_nvidia_jetson.md](./02_setup_nvidia_jetson.md).


## III. Testing Real-Time Performance

To quantify the improvement, you can use `cyclictest` to measure the difference between when a task is scheduled to run and when it actually runs (latency).

Install `rt-tests` package.

```sh
sudo apt install -y rt-tests
```

Install `stress-ng` package, which is a tool to generate a wide range of synthetic workloads, to test and measure a computer system's performance, stability, and reliability.

```sh
sudo apt install -y stress-ng
```

The Orin NX typically has 8 CPU cores (0-7). Assuming your kernel parameters isolate cores 4-7, you should stress cores 0-3. Open a terminal and run heavy workloads in non-isolated CPU cores by using `taskset` wuth 4 CPU-intensive worokers in 2 minutes.

```sh
sudo taskset -c 0-3 stress-ng --cpu 4 --io 4 --vm 4 --vm-bytes 1G -t 2m
```

Open another terminal and run `cyclictest` on isolated cores in 50000 loops.

```sh
sudo cyclictest -t4 -a4-7 -p80 -i200 -d100 -m -l500000
```

When finish the test, you will see latency values (in microseconds) for each isolated core (e.i., `4-7`).

```txt
T: 0 (23532) P:80 I:200 C: 500000 Min:      1 Act:    2 Avg:    2 Max:       8
T: 1 (23533) P:80 I:300 C: 333352 Min:      2 Act:    2 Avg:    2 Max:       6
T: 2 (23534) P:80 I:400 C: 250014 Min:      2 Act:    2 Avg:    2 Max:       7
T: 3 (23535) P:80 I:500 C: 200012 Min:      2 Act:    2 Avg:    2 Max:       7
```
The results show that the `Min` and `Max` latencies in each isolated core are under 10 microseconds, indicating thatindicates that the real-time kernel is working well.

## Conclusion

You have successfully completed the complex but rewarding process of building and deploying a real-time kernel on your Jetson Orin NX. This capability is the foundation to unlock the full potential of Jetson device for serious robotics and automation. 