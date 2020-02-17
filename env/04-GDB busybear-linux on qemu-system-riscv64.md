#### 1. Environment is setup on Ubuntu:
- `uname -a` output: 
  Linux ubuntu-1804 5.0.0-1029-gcp #30~18.04.1-Ubuntu SMP Mon Jan 13 05:40:56 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
- `sudo apt update` 
- `sudo apt install build-essential`

  
#### 2. Install QEMU on Linux to support emulating risc-v:
- potential prerequisites:
  - `sudo apt-get install glib2.0`
  - `sudo apt-get install libpixman-1-dev`

- note: specify target-list to install riscv only
```
git clone https://git.qemu.org/git/qemu.git
cd qemu
git submodule init
git submodule update --recursive
./configure --target-list=riscv64-softmmu,riscv32-softmmu
make -j8
sudo make install
```


#### 3. Install RISC-V cross-compiler tool chain:
- `git clone --recursive https://github.com/riscv/riscv-gnu-toolchain`
- prerequisites for ubuntu: `sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev`
- create isolated folder as the install path: `/usr/local/riscv`
- `./configure --prefix=/usr/local/riscv`
- `make -j4 linux`
- add it to PATH `export PATH=$PATH:/usr/local/riscv/bin`.


#### 4. OR substitutely, just install GDB instead of the whole tool chain:
- download the riscv GDB repo: `git clone https://github.com/riscv/riscv-binutils-gdb.git`
- `cd riscv-binutils-gdb`
- `git checkout fsf-gdb-8.3-with-sim`
- `./configure --target=riscv64-unknown-linux-gnu`
- `make`
- `sudo make install`
- optionally copy gdb to `/usr/local/riscv/bin`
- full name is: `riscv64-unknown-linux-gnu-gdb`
- verified above steps also succeed on macos

  
#### 5. build busybear linux with debugging stub installed:
- prerequisites:
  - riscv-gnu tool chain (as in step 4)
  - qemu (as in step 2)
- get busybear-linux: 
  - `git clone --recursive https://github.com/michaeljclark/busybear-linux.git`
  - `cd busybear-linux`
  - if submodule recursive download fails, download riscv-pk `git clone https://github.com/riscv/riscv-pk.git` and move it to `busybear-linux/src` manually. 

- add a config patch to `busybear-linux/config/`, these lines install the debug stub:
```
cat <<EOF >.config-fragment
CONFIG_DEBUG_INFO=y
CONFIG_DEBUG_KERNEL=y
CONFIG_GDB_SCRIPTS=y
EOF
```

- edit `busybear-linux/scripts/build.sh`: 
  - find line 56: `cp conf/linux.config build/linux-${LINUX_KERNEL_VERSION}/.config`
  - add one line below: `cp conf/.config-fragment build/linux-${LINUX_KERNEL_VERSION}/.config-fragment`
  - find line 73: `cd build/linux-${LINUX_KERNEL_VERSION}`;
  - add one line below to patch the config file: `./scripts/kconfig/merge_config.sh .config .config-fragment`
  
- run the script, `./script/build.sh`, which will automatically build the following targets sequentially: 
  - busybox
  - dropbear
  - linux
  - bbl

- if build bbl fails due to `stubs-lp64.h`, check this patch and apply it to Makefile.in.[patch](https://github.com/riscv/riscv-pk/pull/114/commits/00f0dd04cbdb670f7e81d7fe5c686cb49e7cd182)

- if normal user has no privilege, try `sudo su` to enter a new root-privileged shell first.

#### 6. Run busybear linux in QEMU
```
sudo qemu-system-riscv64 -s -S\
  -nographic -machine virt \
  -kernel build/riscv-pk/bbl \
  -append "root=/dev/vda ro console=ttyS0" \
  -drive file=busybear.bin,format=raw,id=hd0 \
  -device virtio-blk-device,drive=hd0
  
//// skip the following two lines so far  
  -netdev type=tap,script=scripts/ifup.sh,downscript=scripts/ifdown.sh,id=net0 \
  -device virtio-net-device,netdev=net0
```
- qemu `-gdb {device}`), remote debug stub
  - allows you to specify QEMU to wait for a connection in the specified device.
  - It can accept serial, socket, udp, tcp, stdio, etc. 
  - E.g. `-gdb tcp::9000` to listen on port 9000, then from _GDB_ you can connect to it with `target remote localhost:9000`.
  - The `-s` switch is a shorthand for `-gdb tcp::1234`.
- qemu `-S` (stop machine at start up): 
  - This gives time for the debugger to connect and allows to start debugging from the very beginning, even the early platform firmware.
  - To start execution, you must send QEMU the "continue" command, either via the debugger or the monitor console.
- ?? no symbols in gdb.
- if gdb is too old, report register missing error.
- egdb. 

#### 7. Start debugging with GDB
- launch: 
  - `cd build/linux-5.0/`
  - `riscv64-unknown-linux-gnu-gdb vmlinux`, or after enter gdb, `file /path/to/vmlinuz`
  - 
- inside gdb:
  - `set architecture riscv:rv64`
  - `target remote localhost:1234`
  - `break start_kernel`
  - `continue`


#### 8. have fun
- [ref:](https://stackoverflow.com/questions/11408041/how-to-debug-the-linux-kernel-with-gdb-and-qemu/33203642#33203642)
