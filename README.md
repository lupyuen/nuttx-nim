![Nim App runs OK on Apache NuttX Real-Time Operating System](https://lupyuen.github.io/images/nim-title.png)

# Experiments with Nim on Apache NuttX Real-Time Operating System

Today Apache NuttX RTOS runs on SBCs that have plenty of RAM: Ox64 with 64 MB RAM!

Now that we have plentiful RAM: Maybe we should build NuttX Apps with a Memory-Safe, Garbage-Collected language... Like Nim!

This Nim App: [hello_nim_async.nim](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/nim/examples/hello_nim/hello_nim_async.nim)

```nim
import std/asyncdispatch
import std/strformat

proc hello_nim() {.exportc, cdecl.} =
  echo "Hello Nim!"
  GC_runOrc()
```

Runs OK on NuttX for QEMU RISC-V 64-bit!

```text
+ qemu-system-riscv64 -semihosting -M virt,aclint=on -cpu rv64 -smp 8 -bios none -kernel nuttx -nographic

NuttShell (NSH) NuttX-12.0.3
nsh> uname -a
NuttX  12.0.3 45150e164c5 Dec 23 2023 07:24:20 risc-v rv-virt

nsh> hello_nim
Hello Nim!
```

This is how we build NuttX with the Nim App inside...

```bash
## Install choosenim, add to PATH, select latest Dev Version of Nim Compiler
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
export PATH=/home/vscode/.nimble/bin:$PATH
choosenim devel --latest

## Download WIP NuttX and Apps
git clone --branch nim https://github.com/lupyuen2/wip-pinephone-nuttx nuttx
git clone --branch nim https://github.com/lupyuen2/wip-pinephone-nuttx-apps apps

## Configure NuttX for QEMU RISC-V (64-bit)
cd nuttx
tools/configure.sh rv-virt:nsh64

## Build NuttX
make

## Start NuttX with QEMU RISC-V (64-bit)
qemu-system-riscv64 \
  -semihosting \
  -M virt,aclint=on \
  -cpu rv64 \
  -smp 8 \
  -bios none \
  -kernel nuttx \
  -nographic
```

We made some minor tweaks in NuttX...

# Fix NuttX for Nim

_How did we fix NuttX to compile Nim Apps correctly?_

We moved .nimcache 2 levels up: [apps/examples/hello_nim/Makefile](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/nim/examples/hello_nim/Makefile)

```text
## Move .nimcache 2 levels up
CFLAGS += -I $(NIMPATH)/lib -I ../../.nimcache
CSRCS += $(wildcard ../../.nimcache/*.c)

## Previously:
## CFLAGS += -I $(NIMPATH)/lib -I ./.nimcache
## CSRCS += $(wildcard .nimcache/*.c)
```

And we switched the Nim Target Architecture from RISC-V 32-bit to 64-bit: [apps/config.nims](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/nim/config.nims)

```nim
## Assume we are compiling with `riscv-none-elf-gcc` instead of `riscv64-unknown-elf-gcc`
switch "riscv32.nuttx.gcc.exe", "riscv-none-elf-gcc" ## TODO: Check for riscv64-unknown-elf-gcc
switch "riscv64.nuttx.gcc.exe", "riscv-none-elf-gcc" ## TODO: Check for riscv64-unknown-elf-gcc
## Previously: switch "riscv32.nuttx.gcc.exe", "riscv64-unknown-elf-gcc"
...
      case arch
      ...
      of "risc-v":
        ## TODO: Check for riscv32 or riscv3
        ## CONFIG_ARCH_RV32=y or CONFIG_ARCH_RV64=y
        result.arch = "riscv64"
        ## Previously: result.arch = "riscv32"
```

See the modified files...

- [Changes to NuttX Apps](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/pull/3/files)

- [Changes to NuttX Kernel](https://github.com/lupyuen2/wip-pinephone-nuttx/pull/47/files)

# Nim on Apache NuttX RTOS and Ox64 BL808 RISC-V SBC

Nim also runs OK on Apache NuttX RTOS and Ox64 BL808 RISC-V SBC!

This Nim App: [hello_nim_async.nim](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/nim/examples/hello_nim/hello_nim_async.nim)

```nim
import std/asyncdispatch
import std/strformat

proc hello_nim() {.exportc, cdecl.} =
  echo "Hello Nim!"
  GC_runOrc()
```

Produces this output...

```text
Starting kernel ...
ABC
NuttShell (NSH) NuttX-12.0.3
nsh> uname -a
NuttX  12.0.3 d27d0fd4be1-dirty Dec 24 2023 12:32:23 risc-v ox64

nsh> hello_nim
Hello Nim!
```

[(Source)](https://gist.github.com/lupyuen/adef0acd97669cd3570a0614e32166fc)

To build NuttX + Nim for Ox64 BL808 SBC...

```bash
## Install choosenim, add to PATH, select latest Dev Version of Nim Compiler
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
export PATH=/home/vscode/.nimble/bin:$PATH
choosenim devel --latest

## Download WIP NuttX and Apps
git clone --branch nim https://github.com/lupyuen2/wip-pinephone-nuttx nuttx
git clone --branch nim https://github.com/lupyuen2/wip-pinephone-nuttx-apps apps

## Configure NuttX for Ox64 BL808 RISC-V SBC
cd nuttx
tools/configure.sh ox64:nsh

## Build NuttX Kernel
make

## Build Apps Filesystem
make -j 8 export
pushd ../apps
./tools/mkimport.sh -z -x ../nuttx/nuttx-export-*.tar.gz
make -j 8 import
popd

## Export the Binary Image to `nuttx.bin`
riscv-none-elf-objcopy \
  -O binary \
  nuttx \
  nuttx.bin

## Prepare a Padding with 64 KB of zeroes
head -c 65536 /dev/zero >/tmp/nuttx.pad

## Append Padding and Initial RAM Disk to NuttX Kernel
cat nuttx.bin /tmp/nuttx.pad initrd \
  >Image

## Copy NuttX Image to Ox64 Linux microSD
cp Image "/Volumes/NO NAME/"
diskutil unmountDisk /dev/disk2

## TODO: Boot Ox64 with the microSD
```

See the modified files...

- [Changes to NuttX Apps](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/pull/3/files)

- [Changes to NuttX Kernel](https://github.com/lupyuen2/wip-pinephone-nuttx/pull/47/files)

TODO: Blink an LED with Nim

# Build NuttX with Debian Container in VSCode

Nim Compiler won't install on some machines (like a 10-year-old Mac). So we create a Debian Bookworm Container in VSCode that will compile Nim and NuttX...

1.  Install [Rancher Desktop](https://rancherdesktop.io/)

1.  In Rancher Desktop, click "Settings"...

    Set "Container Engine" to "dockerd (moby)"

    Under "Kubernetes", uncheck "Enable Kubernetes"

    (To reduce CPU Utilisation)

1.  Restart VSCode to use the new PATH

    Install the [VSCode Dev Containers Extension](https://code.visualstudio.com/docs/devcontainers/containers)

1.  In VSCode, click the "Remote Explorer" icon in the Left Bar

1.  Under "Dev Container", click "+" (New Dev Container)

1.  Select "New Dev Container"

1.  Select "Debian"

1.  Select "Additional Options" > "Bookworm"

    (With other versions of Debian, "apt install" will install outdated packages)

Inside the Debian Bookworm Container:

Install NuttX Prerequisites...

```bash
## From https://lupyuen.github.io/articles/nuttx#install-prerequisites
sudo apt update && sudo apt upgrade
sudo apt install \
  bison flex gettext texinfo libncurses5-dev libncursesw5-dev \
  gperf automake libtool pkg-config build-essential gperf genromfs \
  libgmp-dev libmpc-dev libmpfr-dev libisl-dev binutils-dev libelf-dev \
  libexpat-dev gcc-multilib g++-multilib picocom u-boot-tools util-linux \
  kconfig-frontends

## Extra Tools for RISCV QEMU
sudo apt install xxd
sudo apt install qemu-system-riscv64
```

Install RISC-V Toolchain...

```bash
## Download xPack GNU RISC-V Embedded GCC Toolchain for 64-bit RISC-V
wget https://github.com/xpack-dev-tools/riscv-none-elf-gcc-xpack/releases/download/v13.2.0-2/xpack-riscv-none-elf-gcc-13.2.0-2-linux-x64.tar.gz
tar xf xpack-riscv-none-elf-gcc-13.2.0-2-linux-x64.tar.gz

## Add to PATH
export PATH=$PWD/xpack-riscv-none-elf-gcc-13.2.0-2/bin:$PATH

## Test gcc:
## gcc version 13.2.0 (xPack GNU RISC-V Embedded GCC x86_64) 
riscv-none-elf-gcc -v
```

[(Why we use xPack Toolchain)](https://lupyuen.github.io/articles/riscv#appendix-xpack-gnu-risc-v-embedded-gcc-toolchain-for-64-bit-risc-v)

Assuming that we need [Nim Compiler](https://nim-lang.org/install_unix.html)...

1.  Install [Nim Compiler](https://nim-lang.org/install_unix.html)...

    ```bash
    curl https://nim-lang.org/choosenim/init.sh -sSf | sh
    ```

1.  Add to PATH...

    ```bash
    export PATH=/home/vscode/.nimble/bin:$PATH
    ```

1.  Select Latest Dev Version of Nim...

    ```bash
    ## Will take a while!
    choosenim devel --latest
    ```

1.  Create a file named `a.nim`...

    ```text
    echo "Hello World"
    ```

1.  Test Nim...

    ```bash
    $ nim c a.nim
    Hint: used config file '/home/vscode/.choosenim/toolchains/nim-#devel/config/nim.cfg' [Conf]
    Hint: used config file '/home/vscode/.choosenim/toolchains/nim-#devel/config/config.nims' [Conf]
    .....................................................................
    Hint:  [Link]
    Hint: mm: orc; threads: on; opt: none (DEBUG BUILD, `-d:release` generates faster code)
    27941 lines; 0.342s; 30.445MiB peakmem; proj: /workspaces/debian/a.nim; out: /workspaces/debian/a [SuccessX]

    $ ls -l a
    -rwxr-xr-x 1 vscode vscode 96480 Dec 22 12:19 a

    $ ./a
    Hello World
    ```

Git Clone the `nuttx` and `apps` folders. Then configure NuttX...

```bash
## TODO: git clone ... nuttx
## TODO: git clone ... apps

## Configure NuttX for QEMU RISC-V (64-bit)
cd nuttx
tools/configure.sh rv-virt:nsh64
make menuconfig
```

Enable "Application Configuration > Examples > Hello World Example (Nim)"

If we need NuttX Networking: Select...

```text
Networking support: Enable "Networking support"
Networking Support → SocketCAN Support:
  Enable "SocketCAN Support"
  Enable "sockopt support"
RTOS Features → Tasks and Scheduling:
  Enable "Support parent/child task relationships"
  Enable "Retain child exit status"
```

Save and exit menuconfig, then build and run NuttX...

```bash
## Build NuttX
make

## Start NuttX with QEMU RISC-V (64-bit)
qemu-system-riscv64 \
  -semihosting \
  -M virt,aclint=on \
  -cpu rv64 \
  -smp 8 \
  -bios none \
  -kernel nuttx \
  -nographic
```

# Documentation

- [NuttX support for Nim](https://github.com/apache/nuttx-apps/pull/1597)

- [Nim support for NuttX](https://github.com/nim-lang/Nim/pull/21372/files)

- [For Nuttx, change ioselectors to use "select"](https://github.com/nim-lang/Nim/pull/21384)

- [Which implementation of NuttX select/poll/EPOLL is recommended in terms of performance and efficiency](https://github.com/apache/nuttx/issues/8604)

- [Nim on Arduino](https://disconnected.systems/blog/nim-on-adruino/)

- [Nim for Embedded Systems](https://github.com/nim-lang/Nim/blob/devel/doc/nimc.md#nim-for-embedded-systems)

- [Nim Compiler User Guide](https://nim-lang.org/docs/nimc.html)

- [Nim Wrapper for LVGL](https://github.com/mantielero/lvgl.nim)
