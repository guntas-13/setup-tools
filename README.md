# Ubuntu & Kali Containers on MacOS

---

## Ubuntu AMD64 Container Setup

```docker pull --platform=linux/amd64 ubuntu:22.04```

```docker run -it \
  --name ubuntu-amd64-dev \
  --platform=linux/amd64 \
  -v $HOME/docker-work/ubuntu:/work \
  ubuntu:22.04 \
  bash
```
```docker start -ai ubuntu-amd64-dev```

**Container Name**: `ubuntu-amd64-dev`
**Work Directory**: `~/docker-work/ubuntu`

---

## Ubuntu ARM64 Container Setup

```docker pull --platform=linux/arm64 ubuntu:22.04```
```docker run -it \
  --name ubuntu-arm64-dev \
  --platform=linux/arm64 \
  -v $HOME/docker-work/ubuntu-arm64:/work \
  ubuntu:22.04 \
  bash
```
```docker start -ai ubuntu-arm64-dev```

**Container Name**: `ubuntu-arm64-dev`
**Work Directory**: `~/docker-work/ubuntu-arm64`

---

## Kali AMD64 Container Setup

```docker pull --platform=linux/amd64 kalilinux/kali-rolling```
```docker run -it \
  --name kali-amd64-sec \
  --platform=linux/amd64 \
  -v $HOME/docker-work/kali:/work \
  kalilinux/kali-rolling \
  bash
```
```docker start -ai kali-amd64-sec```

**Container Name**: `kali-amd64-sec`
**Work Directory**: `~/docker-work/kali`

---

## General Commands for Containers

```
# List all containers
docker ps -a
```

```
# Get architecture info
uname -m
```

```
# Get OS info
cat /etc/os-release
```

```
# Get CPU info
lscpu
```

```
apt update && apt upgrade -y
```

```
# Install common packages on Kali
apt install -y \
  kali-linux-core \
  build-essential \
  gcc \
  gdb \
  nasm \
  radare2 \
  binwalk \
  strace \
  ltrace \
  tcpdump \
  wireshark \
  nmap \
  hashcat \
  john \
  python3 \
  python3-pip \
  git \
  vim \
  openssl \
  libssl-dev \
  libgmp-dev \
  libmpc-dev
```

```
# Install common packages on Ubuntu
apt install -y \
  build-essential \
  gcc \
  g++ \
  clang \
  llvm \
  make \
  cmake \
  gdb \
  binutils \
  nasm \
  pkg-config \
  git \
  vim \
  python3 \
  python3-pip \
  linux-tools-common \
  linux-tools-generic
```