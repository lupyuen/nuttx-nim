![Nim App runs OK on Apache NuttX Real-Time Operating System](https://lupyuen.github.io/images/nim-title.png)

# Experiments with Nim on Apache NuttX Real-Time Operating System

Read the article...

-   ["Nim on a Real-Time Operating System: Apache NuttX RTOS + Ox64 BL808 SBC"](https://lupyuen.github.io/articles/nim)

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

![Nim + NuttX on QEMU Emulator](https://lupyuen.github.io/images/nim-qemu.png)

# Fix NuttX for Nim

Read the article...

-   ["Nim on a Real-Time Operating System: Apache NuttX RTOS + Ox64 BL808 SBC"](https://lupyuen.github.io/articles/nim)

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

Read the article...

-   ["Nim on a Real-Time Operating System: Apache NuttX RTOS + Ox64 BL808 SBC"](https://lupyuen.github.io/articles/nim)

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

![Nim App runs OK on Apache NuttX Real-Time Operating System and Ox64 BL808 RISC-V SBC](https://lupyuen.github.io/images/nim-ox64.png)

# Blink an LED with Nim

Read the article...

-   ["Nim on a Real-Time Operating System: Apache NuttX RTOS + Ox64 BL808 SBC"](https://lupyuen.github.io/articles/nim)

This is how we Blink an LED with Nim on NuttX: [hello_nim_async.nim](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/nim/examples/hello_nim/hello_nim_async.nim)

```nim
import std/strformat  ## String Formatting

## Import NuttX Functions and Macros from C.
## Based on https://github.com/nim-lang/Nim/blob/devel/lib/std/syncio.nim
proc c_open(filename: cstring, mode: cint): cint {.
  importc: "open", header: "<fcntl.h>",
  nodecl.}
proc c_close(fd: cint): cint {.
  importc: "close", header: "<fcntl.h>",
  nodecl, discardable.}
proc c_ioctl(fd: cint, request: cint): cint {.
  importc: "ioctl", header: "<sys/ioctl.h>",
  nodecl, varargs.}
proc c_usleep(usec: cuint): cint {.
  importc: "usleep", header: "<unistd.h>",
  nodecl, discardable.}
var O_WRONLY {.
  importc: "O_WRONLY", header: "<fcntl.h>".}: cint
var ULEDIOC_SETALL {.
  importc: "ULEDIOC_SETALL", header: "<nuttx/leds/userled.h>".}: cint

## Blink the LED
proc blink_led() =

  ## Open the LED Driver
  echo "Opening /dev/userleds"
  let fd = c_open("/dev/userleds", O_WRONLY)
  if fd < 0:
    echo "Failed to open /dev/userleds"
    return

  ## On Return: Close the LED Driver
  defer: c_close(fd)

  ## Turn on LED
  echo "Set LED 0 to 1"
  var ret = c_ioctl(fd, ULEDIOC_SETALL, 1)
  if ret < 0:
    echo "ioctl(ULEDIOC_SETALL) failed"
    return

  ## Wait a second (literally)
  echo "Waiting..."
  c_usleep(1000_000)

  ## Turn off LED
  echo "Set LED 0 to 0"
  ret = c_ioctl(fd, ULEDIOC_SETALL, 0)
  if ret < 0:
    echo "ioctl(ULEDIOC_SETALL) failed"
    return

  ## Wait again
  echo "Waiting..."
  c_usleep(1000_000)

## Main Function in Nim
proc hello_nim() {.exportc, cdecl.} =

  ## On Return: Force the Garbage Collection
  defer: GC_runOrc()

  ## Print something
  echo "Hello Nim!"

  ## Blink the LED 20 times
  for loop in 0..19:
    blink_led()
```

Which calls our barebones NuttX LED Driver for Ox64 BL808...

- ["LED Driver for Ox64 BL808"](https://github.com/lupyuen/nuttx-ox64#led-driver-for-ox64-bl808)

And Nim blinks our LED on Ox64 BL808 SBC!

- [Watch the Demo on YouTube](https://youtube.com/shorts/KCkiXFxBgxQ)

- [See the Log](https://gist.github.com/lupyuen/553c2da4ad5d119468d223e162573e96)

![Nim blinks our LED on Ox64 BL808 SBC](https://lupyuen.github.io/images/nim-blink2.jpg)

_How did we figure out the Nim Code?_

The code above is equivalent to this in C: [hello_main.c](https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/nim/examples/hello/hello_main.c#L25-L85)

```c
#include <nuttx/config.h>
#include <sys/ioctl.h>
#include <stdio.h>
#include <fcntl.h>
#include <nuttx/leds/userled.h>

int main(int argc, FAR char *argv[])
{
  printf("Hello, World!!\n");

  // Open the LED driver
  printf("Opening /dev/userleds\n");
  int fd = open("/dev/userleds", O_WRONLY);
  if (fd < 0)
    {
      int errcode = errno;
      printf("ERROR: Failed to open /dev/userleds: %d\n",
             errcode);
      return EXIT_FAILURE;
    }

  // Turn on LED
  puts("Set LED 0 to 1");
  int ret = ioctl(fd, ULEDIOC_SETALL, 1);
  if (ret < 0)
    {
      int errcode = errno;
      printf("ERROR: ioctl(ULEDIOC_SUPPORTED) failed: %d\n",
              errcode);
      return EXIT_FAILURE;
    }

  // Sleep a while
  puts("Waiting...");
  usleep(500 * 1000L);

  // Turn off LED
  puts("Set LED 0 to 0");
  ret = ioctl(fd, ULEDIOC_SETALL, 0);
  if (ret < 0)
    {
      int errcode = errno;
      printf("ERROR: ioctl(ULEDIOC_SUPPORTED) failed: %d\n",
              errcode);
      return EXIT_FAILURE;
    }

  // Close the LED Driver
  close(fd);

  return 0;
}
```

![Blink an LED with Nim](https://lupyuen.github.io/images/nim-code.png)

# Inside a Nim App for NuttX

Read the article...

-   ["Nim on a Real-Time Operating System: Apache NuttX RTOS + Ox64 BL808 SBC"](https://lupyuen.github.io/articles/nim)

TODO: What happens inside a NuttX App for NuttX?

https://github.com/lupyuen2/wip-pinephone-nuttx-apps/blob/c714a317e531aa8ab2de7b9a8e4c4b0f89f66626/config.nims

```text
$ export TOPDIR=/workspaces/bookworm/nuttx
$ cd /workspaces/bookworm/apps/examples/hello_nim
$ nim c --header hello_nim_async.nim

read_config: /workspaces/bookworm/nuttx/.config
line=CONFIG_DEBUG_SYMBOLS=y
line=CONFIG_DEBUG_FULLOPT=y
line=CONFIG_ARCH="risc-v"
@["keyval=", "ARCH", "\"risc-v\""]
keyval[1]="risc-v"
line=CONFIG_RAM_SIZE=33554432
* arch:    riscv64
* opt:     oSize
* debug:   true
* ramSize: 33554432
* isSim:   false
Hint: used config file '/home/vscode/.choosenim/toolchains/nim-#devel/config/nim.cfg' [Conf]
Hint: used config file '/home/vscode/.choosenim/toolchains/nim-#devel/config/config.nims' [Conf]
Hint: used config file '/workspaces/bookworm/apps/config.nims' [Conf]
....................................................................................................................................
Hint: mm: orc; opt: size; options: -d:danger
92931 lines; 1.214s; 137.633MiB peakmem; proj: /workspaces/bookworm/apps/examples/hello_nim/hello_nim_async.nim; out: /workspaces/bookworm/apps/.nimcache/hello_nim_async.json [SuccessX]
```

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

Enable the settings...

- "Device Drivers > LED Support > LED Driver"

- "Device Drivers > LED Support > Generic Lower Half LED Driver"

- "Application Configuration > Examples > Hello World Example (Nim)"

- "Application Configuration > Examples > LED Driver Example"

If the build fails with missing `getsockopt`...

```text
ioselectors_epoll.nim:407: undefined reference to `getsockopt'
```

Then we enable NuttX Networking: Select...

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

If the build fails with missing `arm64_netinitialize`...

```text
arm64_initialize.c:219: undefined reference to `arm64_netinitialize'
```

Then [add the function `arm64_netinitialize`](https://github.com/lupyuen2/wip-pinephone-nuttx/commit/77b8295eb761954370481ed113254b7e930f65c0)

If the build fails with missing `riscv_netinitialize`...

```text
riscv_initialize.c:124: undefined reference to `riscv_netinitialize'
```

Then [add the function `riscv_netinitialize'](https://github.com/lupyuen2/wip-pinephone-nuttx/commit/594761cb413a3413a2d9deae15d2f5c31fcc1b65)

If the build fails with `const` errors...

```text
../../.nimcache/@m..@s..@s..@s..@s..@shome@svscode@s.choosenim@stoolchains@snim-@hdevel@slib@spure@sasyncdispatch.nim.c:362:285: error: initialization discards 'const' qualifier from pointer target type [-Werror=discarded-qualifiers]
  362 | IGNOF(tyObject_SelectorImpl__JWlu9aJYbxUSEZxA8xSjuMg), .depth = 0, .display = TM__gfe9a52b5gJr8GSrnr883iA_3, .traceImpl = (void*)eqtrace___pureZasyncdispatch_u2061, .flags = 0};
      |                                                                               ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Then [disable the `-Werror` option](https://github.com/lupyuen2/wip-pinephone-nuttx/commit/e82be7d1f9f662ef369a3c4adc8290a4035f9857)

# System Call for clock_nanosleep

TODO: usleep in our NuttX App calls clock_nanosleep in NuttX Kernel...

```text
00000000000007e8 <usleep>:
usleep():
/workspaces/bookworm/nuttx/libs/libc/unistd/lib_usleep.c:100
{
  struct timespec rqtp;
  time_t sec;
  int ret = 0;

  if (usec)
     7e8:	cd15                	beqz	a0,824 <.L3>	7e8: R_RISCV_RVC_BRANCH	.L3

00000000000007ea <.LVL1>:
/workspaces/bookworm/nuttx/libs/libc/unistd/lib_usleep.c:104
    {
      /* Let clock_nanosleep() do all of the work. */

      sec          = usec / 1000000;
     7ea:	000f47b7          	lui	a5,0xf4
     7ee:	2407879b          	addw	a5,a5,576 # f4240 <.LASF110+0xe2ec1>
     7f2:	02f5573b          	divuw	a4,a0,a5
/workspaces/bookworm/nuttx/libs/libc/unistd/lib_usleep.c:95
{
     7f6:	1101                	add	sp,sp,-32
/workspaces/bookworm/nuttx/libs/libc/unistd/lib_usleep.c:108
      rqtp.tv_sec  = sec;
      rqtp.tv_nsec = (usec - (sec * 1000000)) * 1000;

      ret = clock_nanosleep(CLOCK_REALTIME, 0, &rqtp, NULL);
     7f8:	860a                	mv	a2,sp
     7fa:	4681                	li	a3,0
     7fc:	4581                	li	a1,0
```

clock_nanosleep makes ecall to Kernel clock_nanosleep...

```text
0000000000001dee <clock_nanosleep>:
clock_nanosleep():
/workspaces/bookworm/nuttx/syscall/proxies/PROXY_clock_nanosleep.c:8
#include <nuttx/config.h>
#include <time.h>
#include <syscall.h>

int clock_nanosleep(clockid_t parm1, int parm2, FAR const struct timespec * parm3, FAR struct timespec * parm4)
{
    1dee:	88aa                	mv	a7,a0

0000000000001df0 <.LVL1>:
    1df0:	882e                	mv	a6,a1

0000000000001df2 <.LVL2>:
    1df2:	87b2                	mv	a5,a2

0000000000001df4 <.LVL3>:
    1df4:	8736                	mv	a4,a3

0000000000001df6 <.LBB4>:
sys_call4():
/workspaces/bookworm/nuttx/include/arch/syscall.h:281
  register long r0 asm("a0") = (long)(nbr);
    1df6:	03100513          	li	a0,49

0000000000001dfa <.LVL5>:
/workspaces/bookworm/nuttx/include/arch/syscall.h:282
  register long r1 asm("a1") = (long)(parm1);
    1dfa:	85c6                	mv	a1,a7

0000000000001dfc <.LVL6>:
/workspaces/bookworm/nuttx/include/arch/syscall.h:283
  register long r2 asm("a2") = (long)(parm2);
    1dfc:	8642                	mv	a2,a6

0000000000001dfe <.LVL7>:
/workspaces/bookworm/nuttx/include/arch/syscall.h:284
  register long r3 asm("a3") = (long)(parm3);
    1dfe:	86be                	mv	a3,a5

0000000000001e00 <.LVL8>:
/workspaces/bookworm/nuttx/include/arch/syscall.h:287
  asm volatile
    1e00:	00000073          	ecall
/workspaces/bookworm/nuttx/include/arch/syscall.h:294
  asm volatile("nop" : "=r"(r0));
    1e04:	0001                	nop

0000000000001e06 <.LBE4>:
clock_nanosleep():
/workspaces/bookworm/nuttx/syscall/proxies/PROXY_clock_nanosleep.c:10
  return (int)sys_call4((unsigned int)SYS_clock_nanosleep, (uintptr_t)parm1, (uintptr_t)parm2, (uintptr_t)parm3, (uintptr_t)parm4);
}
    1e06:	2501                	sext.w	a0,a0
    1e08:	8082                	ret
```

System Call Number for clock_nanosleep is 49...

```text
 <2><b5b0>: Abbrev Number: 1 (DW_TAG_enumerator)
    <b5b1>   DW_AT_name        : (strp) (offset: 0x8ca9): SYS_clock_nanosleep
    <b5b5>   DW_AT_const_value : (data1) 49
```

# Documentation

- ["Nim on a Real-Time Operating System: Apache NuttX RTOS + Ox64 BL808 SBC"](https://lupyuen.github.io/articles/nim)

- [NuttX support for Nim](https://github.com/apache/nuttx-apps/pull/1597)

- [Nim support for NuttX](https://github.com/nim-lang/Nim/pull/21372/files)

- [For Nuttx, change ioselectors to use "select"](https://github.com/nim-lang/Nim/pull/21384)

- [Which implementation of NuttX select/poll/EPOLL is recommended in terms of performance and efficiency](https://github.com/apache/nuttx/issues/8604)

- [Nim on Arduino](https://disconnected.systems/blog/nim-on-adruino/)

- [Nim for Embedded Systems](https://github.com/nim-lang/Nim/blob/devel/doc/nimc.md#nim-for-embedded-systems)

- [Nim Compiler User Guide](https://nim-lang.org/docs/nimc.html)

- [Nim Wrapper for LVGL](https://github.com/mantielero/lvgl.nim)
