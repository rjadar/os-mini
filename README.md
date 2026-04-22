Mini Container Runtime with Kernel Memory Monitoring
Overview
This project is a lightweight container runtime implemented in C, featuring kernel-level memory monitoring. It leverages custom Linux kernel modules to provide process isolation, communication, and resource enforcement.

Build and Setup
First, compile the project and insert the kernel module into the host system.

Bash
make
sudo insmod monitor.ko
ls /dev/container_monitor
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss1.png: Displays the compilation process and confirms the kernel module is successfully loaded and registered as a character device in /dev.

Usage Guide
1. Starting the Supervisor
The supervisor acts as the central daemon managing container lifecycles.

Bash
sudo ./engine supervisor ./rootfs-base
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss2.png: Shows the supervisor initializing and entering an idle state, listening for incoming commands via UNIX sockets.

2. Starting a Container
Launch a container with defined memory constraints (Soft and Hard limits).

Bash
sudo ./engine start c1 ./rootfs-alpha /memory_hog --soft-mib 64 --hard-mib 72
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss3.png: Shows the engine spawning a new container process, assigning it a unique PID, and isolating it within the specified root filesystem.

3. Monitoring State
You can query the supervisor to list all active environments.

Bash
sudo ./engine ps
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss4.png: A status table showing container names, PIDs, and their current 'RUNNING' state.

4. Viewing Logs
The system uses a bounded buffer to capture container output.

Bash
sudo ./engine logs c1
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss5.png: The standard output of the application running inside the container, retrieved by the engine CLI.

5. Kernel Memory Monitoring
This demonstrates the interaction between the user-space process and the kernel module.

Bash
sudo dmesg | tail -20
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss6.png: Kernel ring buffer logs showing the monitor module detecting a soft limit breach (warning) and hard limit enforcement.

6. Resource Management & Multi-tenancy
The runtime supports multiple concurrent containers and CPU prioritization.

Bash
# Running Multiple Containers
sudo ./engine start c2 ./rootfs-beta /memory_hog --soft-mib 64 --hard-mib 72
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss7.png: The ps command output confirming two independent containers running side-by-side.

Bash
# CPU Scheduling with Nice Values
sudo ./engine start fast ./rootfs-alpha /cpu_hog --nice 0
sudo ./engine start slow ./rootfs-alpha /cpu_hog --nice 15
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss8.png: Illustrates how the runtime applies different Linux 'nice' values to influence CPU scheduling priority between containers.

Clean Teardown
Gracefully stop containers and verify the system state is restored.

Bash
sudo ./engine stop c1
sudo ./engine stop c2
sudo ./engine ps
https://github.com/rjadar/os-mini/blob/d42a58bd7b61eef6d06dbe215d8b2cea03b3139e/ss9.png: The final state showing an empty process list, confirming that resources were released and the supervisor cleaned up the child processes.

Key Features
Process Isolation: Uses namespaces to jail processes within a specific rootfs.

IPC: Robust UNIX socket communication between the CLI and the supervisor.

Kernel Enforcement: A custom module monitors memory usage in real-time, bypassing traditional user-space lag.

Tiered Limits: Implements a two-tier memory system (Soft warnings vs. Hard kills).

Scheduling: Integrated support for CPU prioritization via nice values.

Conclusion
This project bridges the gap between user-space management and kernel-space enforcement, providing a transparent look at how modern container runtimes function at a low level.
