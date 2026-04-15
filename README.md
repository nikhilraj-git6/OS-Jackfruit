OS Jackfruit — Lightweight Container Runtime
A minimal, Linux-based container runtime built using low-level system programming concepts like namespaces, kernel modules, and IPC.

Team Information
NIKHIL RAJ G- PES2UG24CS319
P DHANUSH- PES2UG24CS333


Setup Instructions:
Follow these steps to set up the project:

# Clone the repository
git clone https://github.com/shivangjhalani/OS-Jackfruit.git

# Navigate to project
cd OS-Jackfruit/boilerplate

# Build the project
make clean
make

# Load kernel module
sudo insmod monitor.ko

# Verify device creation
ls /dev/container_monitor

# (If needed) Fix /proc mount
sudo mount -t proc proc /proc



Run Instructions:
1. Start Supervisor
sudo ./engine supervisor ../rootfs-base
2. Start Containers
sudo ./engine start alpha ../rootfs-alpha /bin/sh
sudo ./engine start beta ../rootfs-beta /bin/sh
3. List Running Containers
sudo ./engine ps
4. View Logs
sudo ./engine logs alpha
5. Memory Stress Test
cp memory_hog ../rootfs-alpha/
chmod +x ../rootfs-alpha/memory_hog

sudo ./engine start alpha ../rootfs-alpha /memory_hog
* Check Kernel Logs
sudo dmesg | grep monitor
* Stop Containers
sudo ./engine stop alpha
sudo ./engine stop beta
* Cleanup Verification
ps aux | grep defunct
* Unload Kernel Module
sudo rmmod monitor



Screenshots (Execution Proof)
ScreenshotS:	

Kernel Module Loaded	/dev/container_monitor created successfully
<img width="1409" height="634" alt="image" src="https://github.com/user-attachments/assets/bf99122e-d923-4360-a6c3-8fa393a30d18" />

Containers Running	Supervisor + multiple containers active
<img width="1022" height="322" alt="image" src="https://github.com/user-attachments/assets/a9383bc0-74f3-42fe-9479-28a330e461f2" />

engine ps Output	Shows container IDs and PIDs
<img width="1024" height="290" alt="image" src="https://github.com/user-attachments/assets/7e582970-a710-410b-9809-8ad94d906432" />

Logs Output	Container execution logs
<img width="976" height="379" alt="image" src="https://github.com/user-attachments/assets/e9897595-2eb3-4d17-9bbb-e7e5a26f4959" />

Soft Limit Trigger	Warning in kernel logs
<img width="1749" height="772" alt="image" src="https://github.com/user-attachments/assets/6e8fc1c1-38aa-45f3-b1c0-231550eaea58" />

Hard Limit Trigger	Container killed due to limit
<img width="1024" height="736" alt="WhatsApp Image 2026-04-15 at 20 37 46" src="https://github.com/user-attachments/assets/66900b4b-e5e7-44f7-89c2-a4dd8cd53e58" />

Stop Command	Container termination confirmation
<img width="1027" height="296" alt="image" src="https://github.com/user-attachments/assets/49de75d3-04fb-40e6-9648-91847b987479" />

No Zombie Processes	No <defunct> processes present
<img width="968" height="360" alt="image" src="https://github.com/user-attachments/assets/ba7b3b61-7180-4b53-b2f4-83384e3810d1" />





Engineering Analysis:
* Namespace Isolation
PID Namespace (CLONE_NEWPID) → Independent process IDs
UTS Namespace (CLONE_NEWUTS) → Custom hostname per container
Mount Namespace (CLONE_NEWNS) → Isolated filesystem

* Containers behave like independent systems.




Inter-Process Communication (IPC):
Method: UNIX Domain Sockets
Path: /tmp/mini_runtime.sock
Flow: CLI ↔ Supervisor

Advantages:

Low latency
No network overhead
Secure local communication



Memory Monitoring (Kernel Module):
Uses get_mm_rss() for memory tracking

Enforcement:

Soft Limit → Warning logged
Hard Limit → Process killed (SIGKILL)

* Enables real-time resource control.




Process Scheduling Analysis:
Tested using cpu_hog
Priority controlled via nice values

Observations:

Lower nice → Higher priority → Faster
Higher nice → Lower priority → Slower



Design Decisions:
Component	Choice	Reason
IPC	UNIX Domain Socket	Efficient & simple
Process Creation	clone()	Lightweight
Filesystem Isolation	chroot()	Secure
Memory Tracking	Kernel Module	Low-level accuracy
Container Storage	Linked List	Dynamic management
Process Cleanup	SIGCHLD	Prevent zombies



Scheduling Results:
Process	Nice Value	Behavior
cpu_hog (default)	0	Normal execution
cpu_hog (low priority)	10	Slower
cpu_hog (high priority)	-5	Faster



Analysis:
Linux scheduler favors lower nice values
Demonstrates fairness and CPU sharing
Confirms scheduling impact on performance



Conclusion:

This project successfully demonstrates:

Linux containerization using namespaces
IPC via UNIX domain sockets
Kernel-level memory monitoring
Process scheduling behavior
Proper lifecycle management (no zombies)

* Combines user-space and kernel-space to simulate a real container runtime.

Outcomes
Linux namespaces (PID, UTS, Mount)
Kernel module development
System programming with clone()
Memory & process management
IPC and synchronization
