#### Before reading: 
This manual shows the build process using manual-built gnu tool chain in part a of each session.
Substitutely, it can also be built with devel toolchain, which is shown in a parallel part b.

#### [main ref](https://wiki.freebsd.org/riscv)

## Install FreeBSD on GCP
- see [6-Install FreeBSD on GCP](https://github.com/MengshiLi/openbsd-riscv-notes/blob/master/env/06-Install%20FreeBSD%20on%20GCP.md), note freeBSD version is 12.0

## Prerequisites after get a fresh freebsd OS.
- `sudo pkg install vim git`

## Install GNU Cross Compiler Tool Chain for RISCV

#### 1. Setup toolchain installation directory
- ` export PREFIX="/usr/local/riscv"  `

#### 2. Install prerequired packages
- `sudo pkg install bison gmp mpfr mpc gawk gsed pkgconf texinfo gmake`

#### 3a. Build Toolchain from source code [Takes longer time, not preferred]
```
git clone https://github.com/freebsd-riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
git submodule update --init --recursive
./configure --prefix=$PREFIX
gmake -j16 freebsd
```

#### 3b. alternatively, install from port directly. [Preferred] 
- [riscv64-xtoolchain-gcc](https://www.freshports.org/devel/riscv64-xtoolchain-gcc)
- `sudo pkg install riscv64-xtoolchain-gcc` 
- after which, we will get: 
```
New packages to be INSTALLED:
	riscv64-xtoolchain-gcc: 0.4_1
	riscv64-gcc: 8.3.0
	riscv64-binutils: 2.33.1,1
```
- These package are installed into `/usr/local/bin`, with prefix `riscv64-unknown-freebsd12.0-`
- the binutils is at: `/usr/local/riscv64-unknown-freebsd12.0`
- They do include the same bunch of x-tools as we built above manually.


## Build FreeBSD Bundle

#### 4. Get _FreeBSD Sources_
- `git clone https://github.com/freebsd/freebsd freebsd-riscv`
- `cd freebsd-riscv`


#### 5a. Build _FreeBSD World_ (w/o Kernel) using hand-built toolchain (Failed).
```
setenv MAKEOBJDIRPREFIX /home/mengsli/rv/obj/
setenv CROSS_BINUTILS_PREFIX $PREFIX/riscv64-unknown-freebsd12.1/bin/
setenv CROSS_COMPILER_PREFIX $PREFIX/bin/riscv64-unknown-freebsd12.1-
setenv XCC ${CROSS_COMPILER_PREFIX}gcc
setenv XCXX ${CROSS_COMPILER_PREFIX}c++
setenv XCPP ${CROSS_COMPILER_PREFIX}cpp
setenv STRIPBIN ${CROSS_COMPILER_PREFIX}strip
setenv X_COMPILER_TYPE gcc

make TARGET_ARCH=riscv64 buildworld -j16
```
- can't make through due to the following error:
```
project/llvm/lib/Analysis/AliasAnalysis.cpp:26:
/usr/include/c++/v1/cstdio:157:9: error: '::gets' has not been declared
using ::gets;
```

#### 5b. Build FreeBSD World (w/o Kernel) using devel/riscv64-xtoolchain-gcc(Succeed)
```
setenv MAKEOBJDIRPREFIX /home/mengsli/obj/
setenv WITHOUT_FORMAT_EXTENSIONS yes
make CROSS_TOOLCHAIN=riscv64-gcc TARGET_ARCH=riscv64 buildworld -j8
```
- approximately takes following time on 8 skylake core: `World built in 1618 seconds, ncpu: 8, make -j8`


#### 6a. Build _FreeBSD kernel_ using hand-built tool:
- `make TARGET_ARCH=riscv64 buildkernel KERNCONF=QEMU`
- the resultant kernel can be found at `/usr/home/mengsli/obj/usr/home/mengsli/freebsd-riscv/riscv.riscv64/sys/QEMU/`
  - kernel
  - kernel.debug
  - kernel.full

#### 6b. Alternatively, Build FreeBSD kernel using devel/riscv64-xtoolchain-gcc: 
- `make TARGET_ARCH=riscv64 buildkernel CROSS_TOOLCHAIN=riscv64-gcc KERNCONF=QEMU`

#### 6c. Alternatively, Get Prebuilt FreeBSD Kernel 
- [from here](https://artifact.ci.freebsd.org/snapshot/head/latest/riscv/riscv64/kernel.xz). 
- Then extract: `xz kernel.xz`


#### 7a. build _BBL_ to boot FreeBSD
```
git clone https://github.com/freebsd-riscv/riscv-pk
cd riscv-pk
mkdir build && cd build
setenv PATH ${PATH}:${PREFIX}/bin
setenv CPP cpp
setenv CFLAGS "-nostdlib"
setenv WITH_ARCH rv64g
../configure --enable-logo --prefix=$PREFIX --host=riscv64-unknown-freebsd12.0 --with-payload=<path_to_freebsd_kernel>
gmake bbl
unsetenv CFLAGS
unsetenv CPP
```

#### 7b. Alternatively, get prebuilt bbl:
- from [here](https://artifact.ci.freebsd.org/snapshot/head/latest/riscv/riscv64/bbl.xz) 

#### 7c, alternatively, build bbl on a linux machine, only need to specify the pay-load.


#### 8. Install FreeBSD world and kernel
```
setenv DESTDIR /home/mengsli/riscv-world
setenv CROSS_BINUTILS_PREFIX /usr/local/riscv64-unknown-freebsd12.0/bin/
make TARGET_ARCH=riscv64 -DNO_ROOT DESTDIR=$DESTDIR installworld
make TARGET_ARCH=riscv64 -DNO_ROOT DESTDIR=$DESTDIR distribution
make TARGET_ARCH=riscv64 INSTALLKERNEL=QEMU -DNO_ROOT DESTDIR=$DESTDIR installkernel
```

#### 9. Build complete rootfs image to run in QEMU
```
cd $DESTDIR
echo 'hostname="qemu"' > etc/rc.conf
echo "/dev/vtbd0        /       ufs     ro      1       1" > etc/fstab
echo "./etc/fstab type=file uname=root gname=wheel mode=0644" >> METALOG
echo "./etc/rc.conf type=file uname=root gname=wheel mode=0644" >> METALOG
makefs -D -f 5000000 -o version=2 -s 20g riscv.img METALOG
```

#### 10. optional, setup the network
- as root:
- [ref0:](http://bsdwiki.reedmedia.net/wiki/networking_qemu_virtual_bsd_systems.html)
- [ref1:](https://www.freshports.org/emulators/qemu) 
  - Needs to set net.link.tap.user_open sysctl in order to use /dev/tap*
  networking as non-root.  Don't forget to adjust device node permissions in
  /etc/devfs.rules.
- [ref2:](https://romain.blogreen.org/blog/2007/09/setting-up-qemu-for-networking-under-freebsd/)
- [ref3:](https://wiki.freebsd.org/qemu)

```
setenv iface vtnet0 # SET iface TO YOUR MAIN INTERFACE, setenv is for csh.
echo "Make configuration persistent through reboots:"
echo net.link.tap.user_open=1 >> /etc/sysctl.conf
echo net.link.tap.up_on_open=1 >> /etc/sysctl.conf
echo chmod 0660 /dev/tap0 >> /etc/rc.local # this should be done with devfs(8)
echo 'cloned_interfaces="tap0 bridge0"' >> /etc/rc.conf
echo 'ifconfig_bridge0="addm '$iface' addm tap0 up"' >> /etc/rc.conf

echo 'Actually configure everything (unless you want to reboot at this point):'
/etc/rc.d/sysctl start
ifconfig bridge0 create
ifconfig tap0 create
ifconfig bridge0 addm $iface addm tap0 up
chmod 0660 /dev/tap0
```

#### 11. Folder Explains:
- freebsd-riscv: freebsd source code which support riscv
- qemu: qemu source code
- obj: output position of build freebsd world and freebsd kernel
- riscv-world: freebsd world with kernel, ready to make a complete rootfs.
- freebsd-img: collected all essential image
