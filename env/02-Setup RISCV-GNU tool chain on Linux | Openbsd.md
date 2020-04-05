## Install on Linux (Ubuntu18)
[reference](https://github.com/riscv/riscv-gnu-toolchain)

### get repo:
```
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
```
### prerequisites:
```
sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```

### installation (Linux)
```
cd riscv-gnu-toolchain
./configure --prefix=/usr/local/riscv
sudo make linux -j8
sudo make install
export PATH=$PATH:/usr/local/riscv/bin
export RISCV=/usr/local/riscv
```

## Install on Linux (Ubuntu18) - (Faster)

### Repo
```
git clone https://sourceware.org/git/binutils-gdb.git
```

### Installation
```
git checkout 6b9374f1e07cb250736815ff8db263199416adc6
mkdir build && cd build
../configure --target=riscv64-unknown-elf --prefix=/usr/local/riscv
sudo make -j4
sudo make install -j4
export PATH=$PATH:/usr/local/riscv/bin
export RISCV=/usr/local/riscv
```

## Install on Openbsd
- `pkg_add riscv-elf-binutils riscv-elf-gcc riscv-elf-newlib`
