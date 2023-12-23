# Experiments with Nim on Apache NuttX Real-Time Operating System

TODO

https://github.com/lupyuen2/wip-pinephone-nuttx-apps/pull/3/files

https://github.com/lupyuen2/wip-pinephone-nuttx/pull/47/files

https://github.com/apache/nuttx-apps/pull/1597

# Build NuttX with Debian Container in VSCode

Nim Compiler won't install on some machines (like a 10-year-old Mac). So we create a Debian Bookworm Container in VSCode that will compile Nim and NuttX...

1.  Install [Rancher Desktop](https://rancherdesktop.io/)

1.  In Rancher Desktop, click "Settings"...

    Set "Container Engine" to "dockerd (moby)"

    Under "Kubernetes", uncheck "Enable Kubernetes"

    (To reduce CPU Utilisation)

1.  Restart VSCode to use the new PATH

    Install the VSCode Docker Extension

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

[Install Nim](https://nim-lang.org/install_unix.html)...

```bash
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
```

Add to PATH...

```bash
export PATH=/home/vscode/.nimble/bin:$PATH
```

Select Latest Dev Version of Nim...

```bash
## Will take a while!
choosenim devel --latest
```

Create a.nim...

```text
echo "Hello World"
```

Test Nim...

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
tools/configure.sh tools/configure.sh rv-virt:nsh64
make menuconfig
```

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
make

qemu-system-riscv64 \
  -semihosting \
  -M virt,aclint=on \
  -cpu rv64 \
  -smp 8 \
  -bios none \
  -kernel nuttx \
  -nographic
```
