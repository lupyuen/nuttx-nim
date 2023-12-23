# Experiments with Nim on Apache NuttX Real-Time Operating System

TODO

# Create Debian Container in VSCode

Create a Debian Bookworm Container in VSCode:

1.  Click "Remote Explorer" icon in the Left Bar

1.  Under "Dev Container", click "+" (New Dev Container)

1.  Select "New Dev Container"

1.  Select "Debian"

1.  Select "Additional Options" > "Bookworm"

    (With other versions of Debian, "apt install" will install outdated packages)


Nim on Debian Bookworm:

https://github.com/apache/nuttx-apps/pull/1597

https://nim-lang.org/install_unix.html

```bash
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
```

Add to PATH:

```bash
export PATH=/home/vscode/.nimble/bin:$PATH
```

Select Latest Dev Version of Nim:

```bash
## Will take a while!
choosenim devel --latest
```

Create a.nim:

```text
echo "Hello World"
```

Test Nim:

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

Install Toolchain:

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


```bash
## From https://lupyuen.github.io/articles/nuttx#install-prerequisites
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

make menuconfig

```text
Networking support: Enable "Networking support"
Networking Support → SocketCAN Support:
  Enable "SocketCAN Support"
  Enable "sockopt support"
RTOS Features → Tasks and Scheduling:
  Enable "Support parent/child task relationships"
  Enable "Retain child exit status"
```
