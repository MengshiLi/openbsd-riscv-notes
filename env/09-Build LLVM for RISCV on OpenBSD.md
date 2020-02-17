### Prerequisites
- OpenBSD 6.6
- git: `pkg_add git`
- cmake: `pkg_add cmake`
- ninja: `pkg_add ninja`
- python3: `pkt_add python`

### Clone the LLVM Project repo from github
- `git clone https://github.com/llvm/llvm-project.git`
- switch to a stable version
- `git checkout origin/release/10.x`

### Build the LLVM Cross-compiler Toolchain
- starting from the root of the cloned llvm-project repo
```
mkdir build
cd build
cmake \
  -DLLVM_ENABLE_FFI:Bool=False \
  -DLLVM_ENABLE_TERMINFO:Bool=False \
  -DLLVM_ENABLE_RTTI:Bool=False \
  -DLLVM_BUILD_LLVM_DYLIB:Bool=True \
  -DLLVM_LINK_LLVM_DYLIB:Bool=True \
  -DLLVM_INSTALL_TOOLCHAIN_ONLY:Bool=True \
  -DLLVM_TARGETS_TO_BUILD="X86" \
  -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="RISCV" \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/usr/local/riscv \
  -DCMAKE_CXX_FLAGS="-fno-ret-protector -mno-retpoline" \
  -G Ninja \
  ../llvm

ninja -j8
ninja install
```

### Some comments
- install prefix is `/usr/local/riscv`, which is not part of the default library search paths, therefore need to `export LD_LIBRARY_PATH=/usr/local/riscv/lib`.
- it dlopen()s things that are not in a normal library path like `libLLVM-9svn.so'`
- could put that in some sort of setup script, e.g. `~/.profile`, that you run before doing riscv stuff
- But had we installed tools into prefix `/usr` then it may have worked without `LD_LIBRARY_PATH`
- but that could clobber the system compiler
- In this way, when we need to rev the cross compiler, we can all just dump /usr/local/riscv and un-tar a new version, no fuss.
