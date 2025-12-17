# 🧠 Systems, OS, Networking & Architecture

---

## 1. What is an "image" in computing?

**Q:** What does the term *image* actually mean?

**A:**
An *image* is a **binary representation of something that normally lives on hardware or memory**.

Depending on context, it could be:

* **Disk image** → byte-for-byte copy of a disk (`.img`)
* **Filesystem image** → a formatted filesystem (`.iso`, `.squashfs`)
* **Kernel image** → compressed executable kernel (`bzImage`, `vmlinuz`)
* **Initramfs image** → early root filesystem (cpio archive)
* **VM image** → virtual disk (`qcow2`, `vmdk`)

> "Image" means *"ready to be loaded, mounted, or executed."*

---

## 2. What does it mean to *mount* an image?

**Q:** What does mounting an image do?

**A:**
Mounting means **attaching a filesystem to the directory tree** so the OS can access its contents.

Example:

```bash
mount -o loop ubuntu.iso /mnt
```

Here:

* `ubuntu.iso` is treated as a **block device**
* Kernel reads its filesystem
* Files appear under `/mnt`

Mounting does **not execute** anything — it only exposes data.

---

## 3. What is a kernel image like `bzImage` and how does it boot?

**Q:** What exactly is `bzImage`?

**A:**
`bzImage` is:

* A **compressed Linux kernel**
* Plus **small boot/setup code**
* Plus a **decompressor**

Boot sequence (simplified):

```
Firmware → Bootloader → bzImage
            ↓
        kernel setup
            ↓
       kernel decompresses itself
            ↓
        kernel starts execution
```

The kernel is **self-contained** and knows how to expand itself into RAM.

---

## 4. How does a bootable drive work?

### BIOS vs UEFI

**Q:** How does the system know a drive is bootable?

### BIOS (legacy)

* BIOS loads **first 512 bytes** (MBR)
* Checks signature `0xAA55`
* Executes boot code

### UEFI (modern)

* Reads **FAT32 EFI System Partition**
* Loads `.efi` executable:

  ```
  /EFI/BOOT/BOOTX64.EFI
  ```

---

## 5. What exactly is my home Wi-Fi "router"?

**Q:** Is my home Wi-Fi device really a router?

**A:**
Yes — but not *only* a router.

It is **4 devices in one**:

| Component               | OSI Layer |
| ----------------------- | --------- |
| Modem (DSL/Fiber/Cable) | L1–L2     |
| Router (IP + NAT)       | L3        |
| Ethernet switch         | L2        |
| Wi-Fi Access Point      | L2        |

That’s why it can:

* Assign IPs
* Route packets
* Switch LAN traffic
* Provide Wi-Fi

---

## 6. Is my home router part of the Internet?

**Q:** Is my router visible to the global Internet?

**A:**

* **WAN side:** Yes, it’s an **IP node in the ISP network**
* **LAN side:** No, it creates a **private network**

Your ISP sees your router as:

* A **Customer Premises Equipment (CPE)**
* Assigned an IP (often private or CGNAT)

---

## 7. Why does my laptop have IP `192.168.x.x` but Google shows another IP?

**Q:** Why two IPs?

**A:**
Because of **NAT (Network Address Translation)**.

* `192.168.0.105` → **private LAN IP**
* `117.235.41.78` → **public IP of ISP/NAT gateway**

Your router (or ISP CGNAT) maps:

```
(private IP, private port) → (public IP, public port)
```

---

## 8. How many devices can my home network support?

**Q:** How many devices can my subnet have?

**A:**
Given:

```
192.168.0.0/24
```

* Network bits: 24
* Host bits: 8
* Total addresses: `2⁸ = 256`
* Usable devices: **254**

So your LAN can have **~254 devices** simultaneously.

---

## 9. I have 3 Wi-Fi routers at home — are they all "routers"?

**Q:** Does my ISP configure 3 routers for my house?

**A:**
Usually **NO**.

Common setups:

### Case 1: One router + 2 access points

* Only **one device does routing + DHCP**
* Others act as **Wi-Fi APs**
* Same IP pool everywhere

### Case 2: Router-behind-router (bad design)

* Each creates its own subnet
* Causes **double NAT**
* IP changes when switching Wi-Fi

Most ISPs prefer **Case 1**.

---

## 10. Do all Wi-Fi devices give me different IPs?

**Q:** When I switch Wi-Fi devices, does my IP change?

**A:**

* **Local IP:** changes *per device*
* **Subnet:** same (if APs are bridged)
* **Public IP:** same (unless WAN changes)

So:

```
Laptop → 192.168.0.105
Phone  → 192.168.0.106
```

Same gateway, same NAT.

---

## 11. How does the ISP assign IPs to home routers?

**Q:** If WAN IPs aren’t global, how does ISP manage millions of users?

**A:**
Using **CGNAT (Carrier-Grade NAT)**.

* Home routers get **private WAN IPs**
* ISP edge routers share **public IPs**
* NAT table tracks millions of connections

Your visible IP is often **not your router**, but:

> an **ISP edge NAT gateway**

---

## 12. Why does a website say "Your ISP can see your traffic"?

**Q:** How does it know I’m "unprotected"?

**A:**
Because:

* Your traffic is **unencrypted at ISP level** (unless HTTPS/VPN)
* ISP sees:

  * IPs you connect to
  * DNS queries
  * Traffic volume

HTTPS hides *content*, not *metadata*.

VPN encrypts everything → ISP sees only encrypted tunnel.

---

## 13. What is NAT in detail?

**Q:** What exactly does NAT do?

**A:**
NAT maps:

```
(inside IP, inside port) → (outside IP, outside port)
```

Example:

```
192.168.0.10:54321 → 117.235.41.78:40001
```

Router maintains a **connection table**.

This enables:

* Many devices
* One public IP
* Stateful firewalling

---

## 14. Is NAT limited to 65,536 connections?

**Q:** Are we limited by port numbers?

**A:**
Yes — **per public IP per protocol**.

* TCP ports: ~65k
* UDP ports: ~65k

But ISPs scale by:

* Multiple public IPs
* Port reuse
* Short-lived connections

This is why IPv6 exists.

---

## 15. How does Linux execute machine code from ELF?

**Q:** How does kernel "run" machine instructions?

**A:**
Flow:

```
execve()
 ↓
Kernel reads ELF headers
 ↓
Maps segments into virtual memory
 ↓
Sets instruction pointer (RIP/PC)
 ↓
Switches to user mode
 ↓
CPU fetches instructions directly
```

Kernel **never interprets instructions**.
CPU executes them **directly from memory**.

The kernel only:

* Loads
* Maps
* Protects
* Switches context

---

## 16. What does "64-bit architecture" actually mean?

**Q:** What is the "64" in x86_64 or ARM64?

**A:**
Primarily:

* **Register width**
* **Address width**
* **Pointer size**
* **Native arithmetic width**

It does **not** limit SIMD.

---

## 17. Then how can CPUs do 512-bit operations?

**Q:** If CPU is 64-bit, how does AVX-512 work?

**A:**
Because:

* SIMD uses **separate vector registers**
* ALUs are **physically wide internally**
* Instructions operate on **packed data**

Example:

```
512-bit vector = 8 × 64-bit values
```

The "64" refers to:

> the **scalar architecture**, not SIMD width

---

## 18. Final Mental Model

* **Firmware** starts execution
* **Bootloader** loads kernel
* **Kernel** maps memory, schedules processes
* **CPU** executes instructions directly
* **Router** routes packets (L3)
* **Switch/AP** forward frames (L2)
* **NAT** multiplexes connections
* **IP** is logical, **MAC** is link-local
* **64-bit** = scalar register & address width

---


# 🐳 Containerization & Docker

---

## Q1. What is containerization in simple terms?

**Containerization** is a way to package an application **with its runtime, libraries, and dependencies** so it runs the same everywhere, while **sharing the host OS kernel**.

* VM → ships a full OS
* Container → ships only *user-space*, reuses host kernel

---

## Q2. Is a Docker container a virtual machine?

**No.**

| VM                      | Container               |
| ----------------------- | ----------------------- |
| Has its own kernel      | Shares host kernel      |
| Heavy (GBs)             | Lightweight (MBs)       |
| Hardware virtualization | OS-level virtualization |

Containers feel like mini-OSes, but they’re just **isolated processes**.

---

## Q3. What exactly is a Docker image?

A **Docker image** is:

* A **read-only filesystem snapshot**
* Built in **layers**
* Contains:

  * OS user-space (e.g., Ubuntu files)
  * Libraries
  * Tools
  * App binaries

Example:

```text
ubuntu:22.04 image
├── /bin
├── /lib
├── /usr
└── /etc
```

👉 No kernel inside the image.

---

## Q4. What is a Docker container then?

A **container** is:

* A **running instance of an image**
* Image layers + one writable layer
* Has:

  * Its own filesystem view
  * Its own PID namespace
  * Its own network namespace

Think:

> **Image = blueprint**
> **Container = running process**

---

## Q5. If my host is macOS, how does a Linux container run?

This is critical:

### On macOS (and Windows):

Docker **cannot use the host kernel** because:

* macOS kernel ≠ Linux kernel

So Docker Desktop:

* Runs a **lightweight Linux VM**
* That VM provides a **Linux kernel**
* All containers run inside that VM

```
macOS
 └── Linux VM (hidden)
     └── Docker containers
```

---

## Q6. Then how does `ubuntu:22.04` work on macOS?

* `ubuntu:22.04` provides **Ubuntu user-space**
* The **Linux kernel** comes from the hidden VM
* Kernel version ≠ Ubuntu version

You can check:

```bash
uname -a
```

---

## Q7. How can Docker run containers for different CPU architectures?

Docker supports **multi-architecture images**.

Example:

```bash
docker run --platform=linux/amd64 ubuntu
docker run --platform=linux/arm64 ubuntu
```

Behind the scenes:

* Docker uses **QEMU emulation**
* CPU instructions are translated dynamically

⚠️ Slower than native, but perfect for:

* Cross-compilation
* Testing

---

## Q8. What is `--platform=linux/amd64` doing exactly?

It tells Docker:

* Pull the **amd64 version** of the image
* Run it using **emulation** if needed

This is why you can build:

* Linux x86 binaries on Mac ARM

---

## Q9. How does compiling inside a container help?

Because:

* Compiler matches **target OS + architecture**
* Output binaries are **native ELF**
* No pollution of host system

Example:

```bash
gcc hello.c -o hello
file hello
# ELF 64-bit LSB executable, x86-64
```

---

## Q10. Where does my container filesystem live on my machine?

By default:

* Inside Docker’s internal VM storage
* Not directly visible on host

To access host files → **bind mount**

---

## Q11. What does this volume mount do?

```bash
-v $HOME/docker-work/ubuntu:/work
```

Meaning:

* Host directory:
  `$HOME/docker-work/ubuntu`
* Container path:
  `/work`

Inside container:

```bash
/work  <==>  ~/docker-work/ubuntu on host
```

---

## Q12. Why is `/work` empty in my container?

Because:

* `$HOME/docker-work/ubuntu` is empty on your Mac

Docker does **not** auto-populate it.

Fix:

```bash
mkdir -p ~/docker-work/ubuntu
touch ~/docker-work/ubuntu/test.txt
```

Then inside container:

```bash
ls /work
# test.txt
```

---

## Q13. Can a container access all my host files?

**Only if you mount them explicitly.**

Example:

```bash
-v $HOME:/host-home
```

Inside container:

```bash
/host-home/Documents
/host-home/Downloads
```

⚠️ Containers are isolated by default (security feature).

---

## Q14. What kernel features make containers possible?

Linux kernel provides:

* **Namespaces** → isolation (PID, net, mount, user)
* **cgroups** → resource limits (CPU, memory)
* **OverlayFS** → layered filesystems

Docker is mostly a **user-friendly wrapper** over these.

---

## Q15. Basic Docker commands everyone should know

```bash
# Run a container
docker run -it ubuntu bash

# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop <name>

# Start existing container
docker start -i <name>

# Remove container
docker rm <name>

# Remove image
docker rmi ubuntu:22.04
```

---
