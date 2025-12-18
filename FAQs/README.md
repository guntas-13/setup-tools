# 🧠 Systems, OS, Networking & Architecture

---

## 1. What is an "image" in computing?

**Q:** What does the term _image_ actually mean?

**A:**
An _image_ is a **binary representation of something that normally lives on hardware or memory**.

Depending on context, it could be:

- **Disk image** → byte-for-byte copy of a disk (`.img`)
- **Filesystem image** → a formatted filesystem (`.iso`, `.squashfs`)
- **Kernel image** → compressed executable kernel (`bzImage`, `vmlinuz`)
- **Initramfs image** → early root filesystem (cpio archive)
- **VM image** → virtual disk (`qcow2`, `vmdk`)

> "Image" means _"ready to be loaded, mounted, or executed."_

---

## 2. What does it mean to _mount_ an image?

**Q:** What does mounting an image do?

**A:**
Mounting means **attaching a filesystem to the directory tree** so the OS can access its contents.

Example:

```bash
mount -o loop ubuntu.iso /mnt
```

Here:

- `ubuntu.iso` is treated as a **block device**
- Kernel reads its filesystem
- Files appear under `/mnt`

Mounting does **not execute** anything - it only exposes data.

---

## 3. What is a kernel image like `bzImage` and how does it boot?

**Q:** What exactly is `bzImage`?

**A:**
`bzImage` is:

- A **compressed Linux kernel**
- Plus **small boot/setup code**
- Plus a **decompressor**

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

- BIOS loads **first 512 bytes** (MBR)
- Checks signature `0xAA55`
- Executes boot code

### UEFI (modern)

- Reads **FAT32 EFI System Partition**
- Loads `.efi` executable:

  ```
  /EFI/BOOT/BOOTX64.EFI
  ```

---

## 5. What exactly is my home Wi-Fi "router"?

**Q:** Is my home Wi-Fi device really a router?

**A:**
Yes - but not _only_ a router.

It is **4 devices in one**:

| Component               | OSI Layer |
| ----------------------- | --------- |
| Modem (DSL/Fiber/Cable) | L1–L2     |
| Router (IP + NAT)       | L3        |
| Ethernet switch         | L2        |
| Wi-Fi Access Point      | L2        |

That’s why it can:

- Assign IPs
- Route packets
- Switch LAN traffic
- Provide Wi-Fi

---

## 6. Is my home router part of the Internet?

**Q:** Is my router visible to the global Internet?

**A:**

- **WAN side:** Yes, it’s an **IP node in the ISP network**
- **LAN side:** No, it creates a **private network**

Your ISP sees your router as:

- A **Customer Premises Equipment (CPE)**
- Assigned an IP (often private or CGNAT)

---

## 7. Why does my laptop have IP `192.168.x.x` but Google shows another IP?

**Q:** Why two IPs?

**A:**
Because of **NAT (Network Address Translation)**.

- `192.168.0.105` → **private LAN IP**
- `117.235.41.78` → **public IP of ISP/NAT gateway**

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

- Network bits: 24
- Host bits: 8
- Total addresses: `2⁸ = 256`
- Usable devices: **254**

So your LAN can have **~254 devices** simultaneously.

---

## 9. I have 3 Wi-Fi routers at home - are they all "routers"?

**Q:** Does my ISP configure 3 routers for my house?

**A:**
Usually **NO**.

Common setups:

### Case 1: One router + 2 access points

- Only **one device does routing + DHCP**
- Others act as **Wi-Fi APs**
- Same IP pool everywhere

### Case 2: Router-behind-router (bad design)

- Each creates its own subnet
- Causes **double NAT**
- IP changes when switching Wi-Fi

Most ISPs prefer **Case 1**.

---

## 10. Do all Wi-Fi devices give me different IPs?

**Q:** When I switch Wi-Fi devices, does my IP change?

**A:**

- **Local IP:** changes _per device_
- **Subnet:** same (if APs are bridged)
- **Public IP:** same (unless WAN changes)

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

- Home routers get **private WAN IPs**
- ISP edge routers share **public IPs**
- NAT table tracks millions of connections

Your visible IP is often **not your router**, but:

> an **ISP edge NAT gateway**

---

## 12. Why does a website say "Your ISP can see your traffic"?

**Q:** How does it know I’m "unprotected"?

**A:**
Because:

- Your traffic is **unencrypted at ISP level** (unless HTTPS/VPN)
- ISP sees:

  - IPs you connect to
  - DNS queries
  - Traffic volume

HTTPS hides _content_, not _metadata_.

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

- Many devices
- One public IP
- Stateful firewalling

---

## 14. Is NAT limited to 65,536 connections?

**Q:** Are we limited by port numbers?

**A:**
Yes - **per public IP per protocol**.

- TCP ports: ~65k
- UDP ports: ~65k

But ISPs scale by:

- Multiple public IPs
- Port reuse
- Short-lived connections

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

- Loads
- Maps
- Protects
- Switches context

---

## 16. What does "64-bit architecture" actually mean?

**Q:** What is the "64" in x86_64 or ARM64?

**A:**
Primarily:

- **Register width**
- **Address width**
- **Pointer size**
- **Native arithmetic width**

It does **not** limit SIMD.

---

## 17. Then how can CPUs do 512-bit operations?

**Q:** If CPU is 64-bit, how does AVX-512 work?

**A:**
Because:

- SIMD uses **separate vector registers**
- ALUs are **physically wide internally**
- Instructions operate on **packed data**

Example:

```
512-bit vector = 8 × 64-bit values
```

The "64" refers to:

> the **scalar architecture**, not SIMD width

---

## 18. Final Mental Model

- **Firmware** starts execution
- **Bootloader** loads kernel
- **Kernel** maps memory, schedules processes
- **CPU** executes instructions directly
- **Router** routes packets (L3)
- **Switch/AP** forward frames (L2)
- **NAT** multiplexes connections
- **IP** is logical, **MAC** is link-local
- **64-bit** = scalar register & address width

---

# 🦀 Interfacing Between Languages: Rust & C

**(1) cross-language linking (Rust ↔ C)**
**(2) object files & the linker**
**(3) shared libraries (`.so`)**

# PART 1 - How one language (Rust) calls another (C), and vice-versa

## Core idea (very important)

> **At link time, languages do not exist.**
> Only **object files**, **symbols**, **ABIs**, and **calling conventions** exist.

If two languages:

1. Produce **object files** in the same format (ELF `.o`)
2. Agree on **ABI** (how arguments are passed, how stack is used, name mangling)
3. Expose compatible **symbols**

👉 then they can be linked together.

---

## What is the ABI doing here?

An **ABI (Application Binary Interface)** defines:

- How function arguments are passed (registers vs stack)
- How return values are passed
- Stack alignment rules
- Who cleans up the stack
- Name mangling rules

### C ABI (on x86_64 Linux, System V ABI)

- First args in `rdi, rsi, rdx, rcx, r8, r9`
- Return in `rax`
- Symbol names are **not mangled**

Rust normally uses **Rust ABI**, which is _not_ stable across versions.

So Rust must explicitly say:

```rust
extern "C"
```

This tells Rust:

> “Generate code that follows the **C ABI**.”

---

# PART 2 - Example: Calling Rust from C

## Step 1: Rust code (library)

```rust
// lib.rs
#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### Why each keyword matters

- `extern "C"` → use C calling convention
- `#[no_mangle]` → prevent Rust from renaming the symbol
- `pub` → make the symbol visible to the linker

---

## Step 2: Compile Rust to object or shared library

### Object file

```bash
rustc --crate-type=staticlib lib.rs
```

Produces:

```
liblib.a   (static library)
```

or:

```bash
rustc --crate-type=cdylib lib.rs
```

Produces:

```
liblib.so
```

---

## Step 3: C code calling Rust

```c
// main.c
#include <stdio.h>

extern int add(int a, int b);

int main() {
    printf("%d\n", add(2, 3));
    return 0;
}
```

---

## Step 4: Link C with Rust output

### Static linking

```bash
gcc main.c -L. -llib -o main
```

### Dynamic linking

```bash
gcc main.c -L. -llib -Wl,-rpath=. -o main
```

---

## What the linker is actually doing

The linker:

1. Sees `add` referenced in `main.o`
2. Searches libraries (`liblib.a` or `liblib.so`)
3. Finds symbol `add`
4. Resolves the address
5. Writes relocation entries into final ELF

No knowledge of Rust or C is needed.

---

# PART 3 - Reverse: Calling C from Rust

## C code

```c
// mul.c
int mul(int a, int b) {
    return a * b;
}
```

Compile to object:

```bash
gcc -c mul.c -o mul.o
```

---

## Rust code

```rust
extern "C" {
    fn mul(a: i32, b: i32) -> i32;
}

fn main() {
    unsafe {
        println!("{}", mul(3, 4));
    }
}
```

---

## Link them together

```bash
rustc main.rs mul.o
```

Rust hands everything to the **system linker (`ld`)**.

---

# PART 4 - What exactly is an object file?

An ELF `.o` file contains:

- `.text` → machine code
- `.data` → initialized globals
- `.bss` → uninitialized globals
- `.symtab` → symbol table
- `.rel.*` → relocation info

Example:

```bash
readelf -s mul.o
```

You’ll see:

```
mul GLOBAL FUNC
```

This is what the linker uses.

---

# PART 5 - Shared Libraries (`.so`) explained deeply

## What is a shared library?

A **shared library**:

- Contains position-independent machine code
- Is loaded **at runtime**
- Can be shared across multiple processes
- Reduces memory usage
- Can be updated independently

---

## How `.so` works at runtime

1. Program starts
2. Kernel loads ELF
3. ELF interpreter (`ld-linux.so`) runs
4. Dynamic linker:

   - Loads needed `.so` files
   - Resolves symbols
   - Applies relocations

5. Jumps to `main`

Check dependencies:

```bash
ldd ./main
```

---

## Creating a shared library in C

```c
// math.c
int square(int x) {
    return x * x;
}
```

Compile:

```bash
gcc -fPIC -shared math.c -o libmath.so
```

---

## Using it

```c
extern int square(int);

int main() {
    return square(5);
}
```

```bash
gcc main.c -L. -lmath -Wl,-rpath=. -o main
```

---

## `-fPIC` - why it matters

PIC = Position Independent Code
Required so:

- Library can be loaded at any memory address
- Avoid text relocations

---

## Static vs Shared (very important)

| Feature | Static `.a`      | Shared `.so`            |
| ------- | ---------------- | ----------------------- |
| Linked  | At compile time  | At runtime              |
| Size    | Bigger binaries  | Smaller binaries        |
| Updates | Recompile needed | Can update `.so`        |
| Memory  | Per process      | Shared across processes |

---

# PART 6 - How all this fits together mentally

```
C source   Rust source
   |           |
 gcc -c     rustc
   |           |
 main.o     lib.o
      \     /
        ld
         |
       ELF binary
```

The linker only sees:

- Symbols
- Relocations
- Addresses

---

# FINAL PART - Q&A (Focused, Insightful)

---

### Q1. How can two different languages work together?

**Because they agree on ABI and object file format, not language semantics.**

---

### Q2. What actually connects Rust and C code?

**The system linker (`ld`) using symbol tables and relocation entries.**

---

### Q3. Why is `extern "C"` needed in Rust?

**To force Rust to use the stable C ABI instead of Rust’s unstable ABI.**

---

### Q4. Why does name mangling break linking?

**Because the linker matches symbols by name - mismatched names mean unresolved symbols.**

---

### Q5. Why do shared libraries need `-fPIC`?

**Because they must run correctly regardless of where the kernel maps them in memory.**

---

### Q6. When is a `.so` actually loaded?

**At program startup by the dynamic loader, before `main()` runs.**

---

### Q7. Is a `.so` part of the kernel?

**No - it’s user-space code mapped into memory by the kernel and resolved by the dynamic loader.**

---

### Q8. Can Rust produce `.so` files?

**Yes - using `cdylib` or `staticlib`. Rust integrates cleanly with C toolchains.**

---

### Q9. Why does linking fail even if compilation succeeds?

**Because compilation checks syntax, but linking checks symbol resolution across translation units.**

---

### Q10. What is the _real_ boundary between languages?

**The ABI + linker - not the compiler frontends.**

---

# 🐳 Containerization & Docker

---

## Q1. What is containerization in simple terms?

**Containerization** is a way to package an application **with its runtime, libraries, and dependencies** so it runs the same everywhere, while **sharing the host OS kernel**.

- VM → ships a full OS
- Container → ships only _user-space_, reuses host kernel

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

- A **read-only filesystem snapshot**
- Built in **layers**
- Contains:

  - OS user-space (e.g., Ubuntu files)
  - Libraries
  - Tools
  - App binaries

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

- A **running instance of an image**
- Image layers + one writable layer
- Has:

  - Its own filesystem view
  - Its own PID namespace
  - Its own network namespace

Think:

> **Image = blueprint** > **Container = running process**

---

## Q5. If my host is macOS, how does a Linux container run?

This is critical:

### On macOS (and Windows):

Docker **cannot use the host kernel** because:

- macOS kernel ≠ Linux kernel

So Docker Desktop:

- Runs a **lightweight Linux VM**
- That VM provides a **Linux kernel**
- All containers run inside that VM

```
macOS
 └── Linux VM (hidden)
     └── Docker containers
```

---

## Q6. Then how does `ubuntu:22.04` work on macOS?

- `ubuntu:22.04` provides **Ubuntu user-space**
- The **Linux kernel** comes from the hidden VM
- Kernel version ≠ Ubuntu version

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

- Docker uses **QEMU emulation**
- CPU instructions are translated dynamically

⚠️ Slower than native, but perfect for:

- Cross-compilation
- Testing

---

## Q8. What is `--platform=linux/amd64` doing exactly?

It tells Docker:

- Pull the **amd64 version** of the image
- Run it using **emulation** if needed

This is why you can build:

- Linux x86 binaries on Mac ARM

---

## Q9. How does compiling inside a container help?

Because:

- Compiler matches **target OS + architecture**
- Output binaries are **native ELF**
- No pollution of host system

Example:

```bash
gcc hello.c -o hello
file hello
# ELF 64-bit LSB executable, x86-64
```

---

## Q10. Where does my container filesystem live on my machine?

By default:

- Inside Docker’s internal VM storage
- Not directly visible on host

To access host files → **bind mount**

---

## Q11. What does this volume mount do?

```bash
-v $HOME/docker-work/ubuntu:/work
```

Meaning:

- Host directory:
  `$HOME/docker-work/ubuntu`
- Container path:
  `/work`

Inside container:

```bash
/work  <==>  ~/docker-work/ubuntu on host
```

---

## Q12. Why is `/work` empty in my container?

Because:

- `$HOME/docker-work/ubuntu` is empty on your Mac

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

- **Namespaces** → isolation (PID, net, mount, user)
- **cgroups** → resource limits (CPU, memory)
- **OverlayFS** → layered filesystems

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
