# Multi-Container Runtime

A lightweight Linux container runtime in C featuring a long-running supervisor daemon, a bounded-buffer logging pipeline, a UNIX-socket CLI, and a kernel-space memory monitor.

---

## 1. Team Information

| Name | SRN |
|------|-----|
| Adarsha.E | PES1UG24AM334 |
| Abdul Mateen Shaikh | PES1UG24AM332 |

---

## 2. Build, Load, and Run Instructions

### Prerequisites

Ubuntu 22.04 or 24.04 in a VM with **Secure Boot OFF**. WSL is not supported.

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

Run the environment preflight check:

```bash
cd src
chmod +x environment-check.sh
sudo ./environment-check.sh
```

### Prepare the Alpine Root Filesystem

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base
```

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

---

### Build

```bash
cd src
make
```

---

### Load Kernel Module

```bash
sudo insmod src/monitor.ko
ls -l /dev/container_monitor
```

---

### Start Supervisor

```bash
sudo ./src/engine supervisor ./rootfs-base
```

---

### CLI Usage

```bash
sudo ./src/engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80
sudo ./src/engine ps
sudo ./src/engine logs alpha
sudo ./src/engine stop alpha
```

---

## 3. Demo with Screenshots

### Screenshot 1 — Multi-Container Supervision
![s1](screenshots/1.png)

---

### Screenshot 2 — Metadata Tracking (`ps`)
![s2](screenshots/2.png)

---

### Screenshot 3 — Bounded-Buffer Logging
![s3](screenshots/3.png)
![s3](screenshots/4.png)

---

### Screenshot 4 — CLI and IPC
![s4](screenshots/5.png)

---

### Screenshot 5 — Soft-Limit Warning
![s5](screenshots/6.png)
![s5](screenshots/7.png)

---

### Screenshot 6 — Hard-Limit Enforcement
![s6](screenshots/8.png)

---

### Screenshot 7 — Scheduling Experiment
![s7](screenshots/9.png)
![s7](screenshots/10.png)

---

### Screenshot 8 — Clean Teardown
![s8](screenshots/11.png)
![s8](screenshots/12.png)

---

## 4. Engineering Analysis

### Isolation
- Uses `CLONE_NEWPID`, `CLONE_NEWUTS`, `CLONE_NEWNS`
- Uses `chroot()` for filesystem isolation

### Supervisor
- Handles lifecycle, logs, IPC
- Uses `waitpid()` to prevent zombies

### IPC
- Pipes → logging
- UNIX socket → control

### Memory Enforcement
- Soft limit → warning
- Hard limit → SIGKILL

### Scheduling
- Demonstrates Linux CFS behavior
- Multi-core hides priority differences

---

## 5. Design Decisions

- `chroot` over `pivot_root` → simplicity
- UNIX socket → clean IPC
- Single consumer logging → avoids race conditions
- Spinlocks in kernel → required for softirq

---

## 6. Conclusion

- Built a working mini container runtime
- Demonstrated:
  - Isolation
  - Scheduling behavior
  - Memory enforcement
- Matches core ideas behind Docker (at a simplified level)
