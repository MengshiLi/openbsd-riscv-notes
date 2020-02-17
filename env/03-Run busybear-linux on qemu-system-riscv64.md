### [main ref](https://github.com/michaeljclark/busybear-linux)

### [supplementary ref](https://www.cnx-software.com/2018/03/16/how-to-run-linux-on-risc-v-with-qemu-emulator/)

### prerequisites:
  -   riscv-gnu tool chain
  -   qemu

### build busybear-linux
```
git clone --recursive https://github.com/michaeljclark/busybear-linux.git
cd busybear-linux
make
```
- might need to download riscv-pk and move it to `busybear-linux/src` manually. `git clone https://github.com/riscv/riscv-pk.git`

- then will automatically build: 
  - busybox
  - dropbear
  - linux
  - bbl

- if build bbl fails due to `stubs-lp64.h`, check this patch to the Makefile.in.[patch](https://github.com/riscv/riscv-pk/pull/114/commits/00f0dd04cbdb670f7e81d7fe5c686cb49e7cd182)

  
### Or if you want to build them manually:
- build riscv-linux
```
git clone https://github.com/riscv/riscv-linux.git
cd riscv-linux
git checkout riscv-linux-4.14
cp ../busybear-linux/conf/linux.config .config
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu-  olddefconfig
make -j4 ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu-  vmlinux
``` 

- build bbl
```
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
mkdir build && cd build
../configure \
    --enable-logo \
    --host=riscv64-unknown-linux-gnu \
    --with-payload=../../riscv-linux/vmlinux
make
```

Note: the automatically build bbl is located at: `busybear-linux/build/riscv-pk/bbl`

### run it on qemu
```
sudo qemu-system-riscv64 -s -S\
  -nographic -machine virt \
  -kernel build/riscv-pk/bbl \
  -append "root=/dev/vda ro console=ttyS0" \
  -drive file=busybear.bin,format=raw,id=hd0 \
  -device virtio-blk-device,drive=hd0 \
  -netdev type=tap,script=scripts/ifup.sh,downscript=scripts/ifdown.sh,id=net0 \
  -device virtio-net-device,netdev=net0
```
- on openbsd
```
doas qemu-system-riscv64 -s \
-nographic -machine virt \
-kernel bbl \
-append "root=/dev/vda ro console=ttyS0" \
-drive file=busybear.bin,format=raw,id=hd0 \
-device virtio-blk-device,drive=hd0
```
- after around 1 second, booted up!
- root:busybear
- some basic information:
```
root@busybear:/# uname -a
Linux busybear 5.0.0 #1 SMP Mon Sep 2 03:26:15 UTC 2019 riscv64 GNU/Linux
root@busybear:/# cat /proc/interrupts
           CPU0       
  7:         82  SiFive PLIC   7  virtio1
  8:        192  SiFive PLIC   8  virtio0
 10:        515  SiFive PLIC  10  ttyS0
IPI0:         0  Rescheduling interrupts
IPI1:         0  Function call interrupts
IPI2:         0  CPU stop interrupts
root@busybear:/# cat /proc/cpuinfo 
processor	: 0
hart		: 0
isa		: rv64imafdcu
mmu		: sv48
```
- don't forget to `halt` to quit linux and qemu.
