### 1. Environment is setup on Ubuntu:
- `uname -a` output: 
  Linux ubuntu-1804 5.0.0-1029-gcp #30~18.04.1-Ubuntu SMP Mon Jan 13 05:40:56 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
- `sudo apt update` 
- `sudo apt install build-essential`

  
### 2. Install QEMU on Linux to emulate risc-v machine:
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


### 3. Install RISC-V cross-compiler tool chain:
- `git clone --recursive https://github.com/riscv/riscv-gnu-toolchain`
- prerequisites for ubuntu: 
  - `sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev`

- the following will install gcc, binutilities, gdb, etc.
```
export RISCV="/usr/local/riscv"
./configure --prefix=$RISCV
make -j4 linux
export PATH=$PATH:$RISCV/bin
```

  
### 4. Get the OpenBSD kernel
- two files:
  - `bsd`: kernel without debug information
  - `bsd.gdb`: kernel with debug stubs.
  - `bsd` is obtained by `llvm-strip bsd.gdb`
  
- copy these two files to Ubuntu18 so that GDB can refer to the symbols.
  - `scp root@10.138.0.2:/home/mars/riscv/src/openbsd/sys/arch/riscv64/compile/GENERIC/obj/bsd.gdb .`
  - if failed to scp from freebsd12 to ubuntu18 directly, can use gcloud compute scp instead: on local laptop (as a relay):
  - `gcloud compute scp mars@openbsd65-rv64:/home/mars/riscv/img/bsd.gdb .`
  - `gcloud compute scp ./bsd.gdb mars@ubuntu18:/home/mars/riscv/gdb4openbsd/`

- the bsd.gdb is referencing openbsd source code, prepare the source code and put it the same absolute path as it is on openbsd machine. 
  - `mkdir /home/mars/riscv/src/`
  - `git clone git@github.com:MengshiLi/openbsd.git`


### 5. Build bbl with bsd as payload
```

../configure \
    --host=riscv64-unknown-linux-gnu \
    --with-payload=/home/mars/riscv/openbsdGDB/bsd

make
```
- the binary bbl is located at: `build/riscv-pk/bbl`, copy it to `/home/mars/riscv/openbsdGDB/` for easy manipulation

- if build bbl fails due to `stubs-lp64.h`, check this patch and apply it to Makefile.in.[patch](https://github.com/riscv/riscv-pk/pull/114/commits/00f0dd04cbdb670f7e81d7fe5c686cb49e7cd182)

- if normal user has no privilege, try `sudo su` first.


### 5b. Build bbl on OpenBSD
- install riscv gnu tool chain on OpenBSD: `pkg_add riscv-elf-binutils riscv-elf-gcc riscv-elf-newlib`

- build bbl: first ensure $CC is not set
```
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
mkdir build && cd build
../configure \
    --host=riscv64-unknown-elf \
    --with-payload=../../../img/bsd
gmake
```


### 6. Run openbsd bbl in QEMU
```
qemu-system-riscv64 -s -S\
  -nographic -machine virt \
  -d in_asm -D debug.log \
  -kernel /home/mars/riscv/img/bbl 
```


### 7. Start gdb 
- launch: 
  - `cd /home/mars/riscv/openbsdGDB/`
  - `riscv64-unknown-linux-gnu-gdb bsd.gdb`, or after enter gdb, `file /path/to/bsd.gdb`

- inside (gdb):
```
set architecture riscv:rv64
target remote localhost:1234
set riscv use-compressed-breakpoints no
break *0x80200000 //_start_kern_bootstrap
continue
```

### Reference:
- [How to debug](http://docs.keystone-enclave.org/en/latest/Getting-Started/How-to-Debug.html)
- [sbi to linux](https://github.com/slavaim/riscv-notes/blob/master/bbl/sbi-to-linux.md)


