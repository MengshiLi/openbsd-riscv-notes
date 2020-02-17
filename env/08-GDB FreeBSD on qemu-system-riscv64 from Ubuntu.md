### 1. Install GDB on FreeBSD (Failed)
Therefore we GDB on Ubuntu18 instead.

### 2. Start Freebsd kernel in QEMU
- install qemu first, refer [here](https://github.com/MengshiLi/openbsd-riscv-notes/blob/master/env/01-Install%20QEMU%20on%20Linux%20%7C%20FreeBSD%20%7C%20OpenBSD.md)

- run freebsd (w/o network device)
```
sudo qemu-system-riscv64 -s -S\
-machine virt \
-m 2048M \
-kernel /home/mengsli/freebsd-img/bbl \
-nographic \
-drive file=/home/mengsli/freebsd-img/riscv.img,format=raw,id=hd0 \
-device virtio-blk-device,drive=hd0 
```

### 3. Debug from Ubuntu18
- copy `kernel.debug` to Ubuntu18 so that GDB can refer to the symbols.
  - use `gcloud compute scp`

- the kernel.debug is referencing freebsd source code, which is located at `/usr/home/mengsli/freebsd-riscv` on GCP machine-`freebsd12`. Therefore we need to clone same repop on ubuntu, and need to put it at the same absolute path as `freebsd12`.
  - `sudo mkdir /usr/home/mengsli/`
  - `cd /usr/home/mengsli/`
  - `sudo git clone https://github.com/freebsd/freebsd freebsd-riscv`

- assuming riscv-xtools have been installed on ubuntu18, it will provide `riscv64-unknown-linux-gnu-gdb`
- enter the folder of `kernel.debug`
- `riscv64-unknown-linux-gnu-gdb kernel.debug`
- inside gdb:
  - `target remote 10.138.0.3:1234`
  - `break _start`

