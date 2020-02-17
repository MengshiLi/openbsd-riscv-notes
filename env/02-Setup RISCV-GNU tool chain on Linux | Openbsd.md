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
./configure --prefix=/usr/local/riscv
sudo make linux -j8
export PATH=$PATH:/usr/local/riscv/bin
export RISCV=/usr/local/riscv
```

## Install on Openbsd
- `pkg_add riscv-elf-binutils riscv-elf-gcc riscv-elf-newlib`
