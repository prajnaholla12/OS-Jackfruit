# Multi-Container Runtime in C

## Team Information
Name: Prajna Holla , Raksha Arun

---

# 1. Project Overview

This project implements a lightweight multi-container runtime in C with a long-running supervisor and a kernel-space memory monitor.

The system demonstrates key operating system concepts including:
- Process isolation using namespaces
- Inter-process communication (IPC)
- Process lifecycle management
- Logging and synchronization
- Kernel-level resource monitoring
- Linux scheduling behavior

The project consists of two main components:
1. User-space runtime (engine.c)
2. Kernel module (monitor.c)

---

# 2. Architecture Overview

## 2.1 Supervisor Design

The system uses a long-running supervisor process that:
- Starts once and remains active
- Manages all containers
- Handles CLI commands
- Tracks container metadata

---

## 2.2 IPC Mechanisms

Two communication paths are used:

### Control Path
- CLI → Supervisor
- Implemented using UNIX domain sockets

### Logging Path
- Container → Supervisor
- Implemented using pipes

---

# 3. Build and Run Instructions

## Build
make

## Load Kernel Module
sudo insmod monitor.ko

## Start Supervisor
sudo ./engine supervisor ../rootfs-base

## Create Root Filesystems
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta

## Start Container
sudo ./engine start alpha ../rootfs-alpha /bin/sh

## List Containers
sudo ./engine ps

## View Logs
cat logs/alpha.log

## Stop Container
sudo ./engine stop alpha

## Kernel Logs
sudo dmesg | tail -20

## Unload Module
sudo rmmod monitor

---

# 4. Project Demo 

---

## 4.1 Supervisor Initialization

The supervisor process is started and remains active, waiting for commands.

Command:
sudo ./engine supervisor ../rootfs-base

Explanation:
The supervisor is a long-running process that manages all containers and handles all incoming CLI requests.

<img width="940" height="122" alt="image" src="https://github.com/user-attachments/assets/a283ef43-1251-489c-b934-5770e7358528" />



---

## 4.2 CLI to Supervisor Communication

A CLI command is issued to start a container.

Command:
sudo ./engine start alpha ../rootfs-alpha /bin/sh

Explanation:
The CLI sends a request to the supervisor through a UNIX domain socket. The supervisor receives the request and processes it.

<img width="940" height="115" alt="image" src="https://github.com/user-attachments/assets/f1b5e23f-848b-4a93-ad5e-eb0f4391934f" />



---

## 4.3 Container Creation

The supervisor creates a container using clone().

Explanation:
The container is created with:
- PID namespace
- UTS namespace
- Mount namespace

This ensures process and filesystem isolation.

Expected Output:
Started container alpha with PID ...

<img width="940" height="134" alt="image" src="https://github.com/user-attachments/assets/68272a52-d33d-4f24-b401-510a2f7f17b7" />




---

## 4.4 Multiple Containers Running

Multiple containers can run at the same time.

Command:
ps aux | grep engine

Explanation:
This shows the supervisor process along with multiple container processes running concurrently.

<img width="940" height="323" alt="image" src="https://github.com/user-attachments/assets/2a25d15d-d2ca-418f-9500-7c1c5cb76a7e" />


---

## 4.5 Metadata Tracking

The supervisor tracks container information.

Command:
sudo ./engine ps

Explanation:
The ps command displays container metadata such as:
- Container ID
- PID
- State

Example Output:
alpha | PID 43303 | running

<img width="940" height="263" alt="image" src="https://github.com/user-attachments/assets/32eea031-9916-4369-8be6-664abde11b10" />


---

## 4.6 Stop Container

A running container is stopped.

Command:
sudo ./engine stop alpha

Explanation:
The supervisor sends a termination signal to the container and updates its state.

<img width="940" height="102" alt="image" src="https://github.com/user-attachments/assets/d50cabab-7df7-4b89-8193-c1007b912ab7" />
<img width="940" height="316" alt="image" src="https://github.com/user-attachments/assets/06729696-fadd-4c08-b565-2544c25393ea" />



---

## 4.7 Logging System

Container output is captured and stored.

Command:
cat logs/alpha.log

Explanation:
The logging system uses:
- Pipes to capture output
- A bounded buffer for synchronization
- A consumer thread to write logs

<img width="940" height="568" alt="image" src="https://github.com/user-attachments/assets/cf7754ef-4b4a-47f9-b0d0-3e6e19b4d34c" />


---

## 4.8 Kernel Monitoring (Soft and Hard Limits)

The kernel module monitors container memory usage.

Command:
sudo dmesg | tail -20

Explanation:
- Soft limit → warning message
- Hard limit → process is killed

Example Output:
[container_monitor] SOFT LIMIT alpha  
[container_monitor] HARD LIMIT alpha  

<img width="940" height="543" alt="image" src="https://github.com/user-attachments/assets/a8258135-58d3-49e5-8e5c-0a2dc2074269" />


---

## 4.9 Scheduling Experiment

Containers are run with different priorities.

Command:
sudo ./engine start c1 ../rootfs-alpha ./cpu_hog --nice 10  
sudo ./engine start c2 ../rootfs-beta ./cpu_hog --nice -5  

Explanation:
Lower nice value means higher priority.  
This demonstrates Linux scheduling behavior.

<img width="940" height="200" alt="image" src="https://github.com/user-attachments/assets/d1e4a9d7-69ea-4290-b4af-adc8174d5766" />


---

# 5. Engineering Analysis

## Isolation Mechanisms
Namespaces isolate processes, hostname, and filesystem. Each container runs in its own environment.

## Supervisor Design
The supervisor manages all containers, tracks metadata, and prevents zombie processes.

## IPC and Synchronization
- UNIX sockets for control
- Pipes for logging
- Bounded buffer ensures safe concurrency

## Memory Monitoring
RSS is used to measure memory usage. Kernel module enforces limits.

## Scheduling Behavior
Nice values affect CPU allocation and process priority.

---

# 6. Design Decisions

| Component | Choice | Reason |
|----------|-------|-------|
| IPC | UNIX socket | Simple and effective |
| Isolation | chroot | Easy to implement |
| Logging | bounded buffer | Prevents data loss |
| Monitoring | kernel module | Accurate enforcement |

---

# 7. Conclusion

This project demonstrates a complete container runtime system with user-space and kernel-space integration. It showcases key OS concepts such as process isolation, IPC, scheduling, and memory management.
