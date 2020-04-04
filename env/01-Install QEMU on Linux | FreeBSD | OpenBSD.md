Suggest install from source, so that we can get latest bits for risc-v support.

### Install on Linux:
- need gcc to build:
```
sudo apt update
sudo apt install build-essential
```

- prerequisites:
  - `sudo apt-get install glib2.0`
  - `sudo apt-get install libpixman-1-dev`

- note 1: specify target to install on demand
```
git clone https://git.qemu.org/git/qemu.git
cd qemu
git submodule init
git submodule update --recursive
./configure --target-list=riscv64-softmmu
make -j8
sudo make install
```

### Install on FreeBSD:
```
sudo pkg install glib pixman

export PREFIX="/usr/local/riscv"

git clone https://github.com/qemu/qemu
cd qemu
mkdir build && cd build
../configure --target-list=riscv64-softmmu --prefix=$PREFIX
sudo gmake -j8
sudo gmake install

export PATH="$PATH:$PREFIX/bin"
```

### Install on OpenBSD:
```
export PREFIX="/usr/local/riscv"

git clone https://github.com/qemu/qemu
cd qemu
git submodule init
git submodule update --recursive
mkdir build && cd build
../configure --target-list=riscv64-softmmu --prefix=$PREFIX --enable-debug
doas gmake -j8
doas gmake install
export PATH="$PATH:$PREFIX/bin"
```
