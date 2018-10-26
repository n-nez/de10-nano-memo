# DE10 Nano memo

*My list of random notes on DE10 Nano SoC related stuff*

---
# Required Software

## FPGA & HLS Development
- [Quartrus](https://www.intel.com/content/www/us/en/software/programmable/quartus-prime/overview.html)
## SoC software development
- [SoC EDS](https://www.altera.com/products/design-software/embedded-software-developers/soc-eds/overview.html)
    *(should be of same version as quartus)*

---
# Tutorials
- [Altera SoC Workshop](https://rocketboards.org/foswiki/Documentation/AlteraSoCWorkshopSeries)
    Series on rocketboards
  - \*:*it is for DE0 Nano but contains some useful information*
- [SoCEDS Getting Started](https://fpgawiki.intel.com/wiki/SoCEDSGettingStarted)
    on Intel FPGA Wiki

---

# Documentation
- [HPS SoC Boot Guide](https://www.intel.com/content/dam/altera-www/global/en_US/pdfs/literature/an/an709.pdf) *(pdf)*
- [SoC FPGA Embedded Development Suite User Guide](https://www.intel.com/content/dam/www/programmable/us/en/pdfs/literature/ug/ug_soc_eds.pdf) *(pdf)*

# Installing software

## Quartus

Install libpng12 to make Quartus happy
```
wget -q -O /tmp/libpng12.deb http://mirrors.kernel.org/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb \
    && sudo dpkg -i /tmp/libpng12.deb \
    && rm /tmp/libpng12.deb
```

Install quartus using this [howto](http://www.bitsnbites.eu/installing-intelaltera-quartus-in-ubuntu-17-10/)
  \*:*skip libpng12 step*

---
# Random terms
- MPU: Micro Processor Unit
- HPS: Hard Processor System
- BSP: [Board Support Package](https://en.wikipedia.org/wiki/Board_support_package)

---
# Generate qsys system using tcl script
```
$ qsys-script --script=soc_system.tcl
$ qsys-generate soc_system.qsys --synthesis=Verilog
```

---
# Compile quartus project from command line
```
$ quartus_sh --flow compile soc_system.qpf
```

---
# Convert sof to rbf
```
$ quartus_cpf -c -o bitstream_compression=on file.sof file.rbf
```

---
# Connect to USB Serial
```
$ screen /dev/ttyUSB0 115200
```

---
# Build BSP
```
$ bsp-create-settings --type spl --bsp-dir bsp_build_dir/ --settings settings.bsp --preloader-settings-dir ./hps_isw_handoff/soc_system_hps_0
$ cd bsp_build_dir
$ make
$ make uboot
```

# Program preloader & uboot into sdcard
```
$ sudo $(which alt-boot-disk-util) -b uboot-socfpga/u-boot.img -a write /dev/sdX
$ sudo $(which alt-boot-disk-util) -p preloader-mkpimage.bin -a write /dev/sdX
```

---
# Driver program example
```cpp
constexpr size_t HPS_TO_FPGA_LW_BASE = 0xFF200000;

// Half part of phys memory
constexpr size_t HW_BUFFERS_PHYS_ADDR_BASE = 0x20000000;

uint8_t* mapPhysicalMemory(size_t base, size_t size) {
    int fd = open("/dev/mem", O_RDWR | O_SYNC, 0);
    if (fd == -1)
        throw std::system_error(errno, std::generic_category());
    int rw = PROT_READ | PROT_WRITE;
    auto* mapped_base = reinterpret_cast<uint8_t*>(mmap(nullptr, size, rw, MAP_SHARED, fd, base));
    if (mapped_base == MAP_FAILED)
        throw std::system_error(errno, std::generic_category());
    return mapped_base;
}

auto* base = mapPhysicalMemory(HPS_TO_FPGA_LW_BASE, 0x2000);
```
