# Linux kernel
## Optimized for Xilinx Zynq Ultrascale+ ZCU104 devboard

### Intro
This branch is based on Linux v5.0-rc4. In addition, it contains:
* Zynq Ultrascale+ Cortex-A53 CoreSight components DTS
* Zynq Ultrascale+ `defconfig` borrowed from Xilinx github.
* Custom `.config` file (based on the Xilinx `defconfig`) with
  CoreSight components, perf subsystem and NFS compiled in.
  Some unnecessary components (audio, wifi, bt) are removed
  to achieve lower footprint. If you find anything missing,
  feel free to adjust the config.
* `README.md` file you are just reading.

### Adjusting config
It is assumed that you already have AARCH64 GNU toolchain.
```
CROSS_COMPILE=aarch64-linux-gnu- ARCH=arm64 make menuconfig
```

Perform necessary changes and save it in `.config`. Afterwards, you can commit
the updated configuration.

### Building
It is assumed that you already have AARCH64 GNU toolchain.
```
$ CROSS_COMPILE=aarch64-linux-gnu- ARCH=arm64 make
```
* Kernel image is generated in `arch/arm64/boot/Image`
* DTB file is generated in `arch/arm64/boot/dts/xilinx/zynqmp-zcu104-revA.dtb`

### Further reading
* Take a look into the coresight/debian-rootfs> project to learn how to
preapre Debian root filesystem to use with your kernel.
* The coresight/uboot> project will tell you how to boot your kernel.

