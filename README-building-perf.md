# Building perf + OpenCSD

### Intro
* `perf` - performance monitor embedded in Linux kernel with user space frontend
* `libopencsd` - library for decoding CoreSight performance data gathered by perf
  to human-readable plaintext


### Dependencies
The list may be incomplete, however build systems are rather verbose in their error messages,
so identifying missing dependencies should be easy. However, dependencies could not be resolved
for x86-64 to aarch64 cross compilation. *It is suggested then, to build natively on aarch64*.

Identified dependencies in Debian:
```
apt-get update && apt-get install                                                               \
   dh-autoreconf bison libdw-dev libelf-dev flex uuid-dev libpopt-dev libnewt-dev libperl-dev   \
   binutils-arm-none-eabi libcrypto++-dev libunwind-dev systemtap-sdt-dev libssl-dev            \
   libiberty-dev elfutils libglib2.0-dev python-dev
```

If root filesystem running on the target board has been prepared using the coresight/debian-rootfs>
project, these dependencies should already be installed automatically on filesystem creation.

### Building
1. Clone OpenCSD and build `libopencsd`:
   ```
   git clone https://github.com/Linaro/OpenCSD.git
   cd decoder/OpenCSD/build/linux
   make
   make install
   ```
1. Get Linux source and build `perf` with the libraries.
   Linux sources used must be the same that were used to build the kernel image running
   on the platform where perf will be used. First, check if they are served along with
   rootfs by the tftp/nfs server. If not, clone it and checkout to the matching branch:
   ```
   git clone git@192.168.1.222:coresight/linux.git
   git checkout <branch used to build your kernel>
   cd linux
   CORESIGHT=1 make -C tools/perf
   ```
1. Optionally, symlink the perf executable to one of your PATH directories:
   ```
   ln -s /<absolute path>/linux/tools/perf/perf /usr/bin
   ```

**Important:** While building perf, `make` lists available features and informs about the missing
ones. This list is **completely misleading** in our case. Example listing:
```
Auto-detecting system features:
...                         dwarf: [ on  ]
...            dwarf_getlocations: [ on  ]
...                         glibc: [ on  ]
...                          gtk2: [ OFF ]
...                      libaudit: [ OFF ]
...                        libbfd: [ OFF ]
...                        libelf: [ on  ]
...                       libnuma: [ OFF ]
...        numa_num_possible_cpus: [ OFF ]
...                       libperl: [ OFF ]
...                     libpython: [ OFF ]
...                      libslang: [ OFF ]
...                     libcrypto: [ OFF ]
...                     libunwind: [ OFF ]
...            libdw-dwarf-unwind: [ on  ]
...                          zlib: [ on  ]
...                          lzma: [ on  ]
...                     get_cpuid: [ OFF ]
...                           bpf: [ on  ]
...                        libaio: [ on  ]

Makefile.config:461: No sys/sdt.h found, no SDT events are defined, please install systemtap-sdt-devel or systemtap-sdt-dev
Makefile.config:507: No libunwind found. Please install libunwind-dev[el] >= 1.1 and/or set LIBUNWIND_DIR
Makefile.config:599: No libcrypto.h found, disables jitted code injection, please install openssl-devel or libssl-dev
Makefile.config:614: slang not found, disables TUI support. Please install slang-devel, libslang-dev or libslang2-dev
Makefile.config:628: GTK2 not found, disables GTK2 support. Please install gtk2-devel or libgtk2.0-dev
Makefile.config:655: Missing perl devel files. Disabling perl scripting support, please install perl-ExtUtils-Embed/libperl-dev
Makefile.config:687: No 'python-config' tool was found: disables Python support - please install python-devel/python-dev
Makefile.config:737: No bfd.h/libbfd found, please install binutils-dev[el]/zlib-static/libiberty-dev to gain symbol demangling
Makefile.config:781: No numa.h found, disables 'perf bench numa mem' benchmark, please install numactl-devel/libnuma-devel/libnuma-dev
Makefile.config:858: No alternatives command found, you need to set JDIR= to point to the root of your Java directory
```
Please note that the above output **does not mention** `libopencsd`.

`libopencsd` is not mentioned directly anywhere. The only clue if it has been
detected or not is a similiar line in the log:

CSD not detected (notice `-stub` version of ETM decoder being linked):
```
CC tests/thread-mg-share.o
CC util/cs-etm-decoder/cs-etm-decoder-stub.o
CC util/intel-pt-decoder/intel-pt-decoder.o
```

CSD detected (notice proper ETM decoder being linked):
```
CC util/auxtrace.o
CC util/cs-etm-decoder/cs-etm-decoder.o
LD util/cs-etm-decoder/libperf-in.o
```

Additionally, please make sure, that the following line says `ON`. Python scripting is required
to disassemble `perf.data`:
```
...                     libpython: [ ON ]
```

