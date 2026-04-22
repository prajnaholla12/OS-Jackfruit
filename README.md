# Multi-Container Runtime in C

## Team Information
Name: Prajna Holla  
      Raksha Arun
---

## Project Overview

This project implements a lightweight multi-container runtime in C with a long-running supervisor and a kernel-space memory monitor. The system demonstrates core operating system concepts including process isolation, inter-process communication, scheduling, and kernel-user interaction.

The project consists of two integrated components:
- A user-space runtime and supervisor (engine.c)
- A kernel module for memory monitoring (monitor.c)

The runtime manages multiple containers concurrently, captures logs, handles lifecycle events, and enforces memory limits through kernel integration.

---

## Architecture Overview

### User-Space Runtime (engine.c)
- Implements the supervisor process
- Handles CLI commands
- Maintains container metadata
- Launches containers using clone()
- Manages logging using a bounded-buffer pipeline

### Kernel Module (monitor.c)
- Tracks container processes using their PIDs
- Enforces soft and hard memory limits
- Communicates with user-space via ioctl

### IPC Design

Two IPC mechanisms are used:
- Control Plane: UNIX domain socket between CLI and supervisor
- Logging Pipeline: Pipes from container stdout/stderr to supervisor

---

## Build and Run Instructions

### Build
make

### Load Kernel Module
sudo insmod monitor.ko

### Start Supervisor
sudo ./engine supervisor ../rootfs-base

### Create Container Root Filesystems
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta

### Start Container
sudo ./engine start alpha ../rootfs-alpha /bin/sh

### List Containers
sudo ./engine ps

### View Logs
cat logs/alpha.log

### Stop Container
sudo ./engine stop alpha

### Check Kernel Logs
sudo dmesg | tail -20

### Unload Kernel Module
sudo rmmod monitor

---

## Demo and Screenshots

### 1. Supervisor Initialization
The supervisor starts and remains active, waiting for container commands.  
This demonstrates a long-running process managing container lifecycle.

### 2. CLI and IPC Communication
The CLI sends commands to the supervisor through a UNIX domain socket.  
The supervisor receives and processes the command.

### 3. Container Creation
A container is created using clone() with PID, UTS, and mount namespaces.  
The supervisor prints the container ID and PID.

### 4. Multiple Containers Running
Multiple containers run simultaneously under a single supervisor process.  
This is verified using ps aux.

### 5. Metadata Tracking
The ps command displays container metadata including ID, PID, and state.

Example:
alpha | PID 43303 | running

### 6. Container Stop
The stop command terminates a running container and updates its state.

### 7. Logging System
Container output is captured using pipes and stored in log files.  
The logging system uses a bounded buffer to ensure safe concurrent access.

Example:
cat logs/alpha.log

### 8. Kernel Monitoring (Soft and Hard Limits)
The kernel module tracks container processes and enforces memory limits.

Example output:
[container_monitor] Timer running
[container_monitor] SOFT LIMIT alpha
[container_monitor] HARD LIMIT alpha

### 9. Scheduling Experiment
Containers are run with different nice values to observe scheduling behavior.

Example:
./engine start c1 ../rootfs-alpha ./cpu_hog --nice 10
./engine start c2 ../rootfs-beta ./cpu_hog --nice -5

---

## Engineering Analysis

### Isolation Mechanisms
Containers use PID, UTS, and mount namespaces to isolate processes and filesystem views.  
Each container operates within its own root filesystem using chroot().

### Supervisor and Process Lifecycle
The supervisor acts as the parent process managing all containers.  
It tracks metadata, reaps child processes, and handles signals to avoid zombies.

### IPC, Threads, and Synchronization
Two IPC mechanisms are used:
- UNIX sockets for control communication
- Pipes for logging

A bounded buffer with mutexes and condition variables ensures safe producer-consumer synchronization.

### Memory Management and Enforcement
RSS (Resident Set Size) is used to measure memory usage.  
Soft limits trigger warnings, while hard limits terminate processes.  
Enforcement is done in kernel space for reliability.

### Scheduling Behavior
Different nice values demonstrate how Linux scheduling prioritizes processes.  
Lower nice values receive more CPU time compared to higher nice values.

---

## Design Decisions and Tradeoffs

| Component | Design Choice | Tradeoff |
|----------|-------------|---------|
| IPC | UNIX domain socket | Simpler implementation, limited scalability |
| Isolation | chroot | Easier than pivot_root, less secure |
| Logging | bounded buffer | Increased complexity, ensures correctness |
| Kernel Monitoring | Timer-based checks | Periodic, not real-time |

---

## Conclusion

This project successfully implements a supervised multi-container runtime with kernel-level memory monitoring. It demonstrates key operating system concepts including process isolation, inter-process communication, synchronization, scheduling, and kernel-user interaction.

The implementation provides a practical understanding of how container runtimes and operating system mechanisms work together.
