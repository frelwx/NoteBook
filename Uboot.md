
## 概述

本目录包含 **U-Boot** 的源代码，这是一个适用于基于 PowerPC、ARM、MIPS 及其他多种处理器的嵌入式板卡的引导加载程序（Boot Loader）。U-Boot 可以安装在引导 ROM 中，用于初始化和测试硬件，或下载并运行应用代码。

U-Boot 的开发与 Linux 密切相关：部分源代码来自于 Linux 源码树，部分头文件是通用的，并且特别设计了支持引导 Linux 镜像的功能。

我们在设计中也特别注重使该软件易于配置和扩展。例如，所有监控命令都使用相同的调用接口实现，因此添加新命令非常容易。此外，您可以动态加载和运行一些不常用的代码（例如硬件测试工具），而无需将其永久集成到监控程序中。

---

## 当前状态

一般而言，**configs/** 目录中存在默认配置文件的所有板卡都经过了一定程度的测试，可视为“可用”。事实上，其中许多板卡已用于生产系统。

如果遇到问题，您可以使用以下命令：

```bash
scripts/get_maintainer.pl <路径>
```

以识别负责各个板卡和子系统的人员或公司。或者，您可以查看 Git 日志。

---

## 帮助资源  

如果您对 U-Boot 有疑问、遇到问题或有贡献内容，请发送消息至 **U-Boot 邮件列表**：<u-boot@lists.denx.de>。邮件列表的历史记录也可以查询，在提出常见问题（FAQ）前，请先搜索相关内容。

- 邮件列表归档：https://lists.denx.de/pipermail/u-boot  
- 另一归档链接：https://marc.info/?l=u-boot  

---

## 源代码获取方式

U-Boot 的源代码托管在 Git 仓库中：  
- Git 仓库地址：https://source.denx.de/u-boot/u-boot.git  
- 在线浏览源码：https://source.denx.de/u-boot/u-boot  

页面上的 “Tags” 链接允许您下载任意版本的源码包。此外，官方发布版本也可以通过 DENX 文件服务器下载（支持 HTTPS 或 FTP）：  
- HTTPS 下载：https://ftp.denx.de/pub/u-boot/  
- FTP 下载：ftp://ftp.denx.de/pub/u-boot/  

---

## 项目历史

- 起始于 **8xxrom** 源代码  
- 创建 **PPCBoot** 项目（https://sourceforge.net/projects/ppcboot）  
- 清理代码  
- 使添加自定义板卡变得更容易  
- 支持添加其他 [PowerPC] CPU  
- 扩展功能，特别是：  
  - 为 Linux 引导加载程序提供扩展接口  
  - 支持 S-Record 下载  
  - 支持网络引导  
  - 支持 ATA 磁盘 / SCSI 等引导  
- 创建 **ARMBoot** 项目（https://sourceforge.net/projects/armboot）  
- 添加其他 CPU 家族（从 ARM 开始）  
- 创建 **U-Boot** 项目（https://sourceforge.net/projects/u-boot）  
- 当前项目页面：https://www.denx.de/wiki/U-Boot  

---

## 名称与拼写规范

本项目的正式名称为 **"Das U-Boot"**。在所有书面文本（如文档、源码注释等）中应使用拼写 **"U-Boot"**，例如：

```text
这是 U-Boot 项目的 README 文件。
```

文件名等应基于字符串 **"u-boot"**，例如：

```c
include/asm-ppc/u-boot.h
#include <asm/u-boot.h>
```

变量名、预处理器常量等应基于 **"u_boot"** 或 **"U_BOOT"**，例如：

```c
U_BOOT_VERSION  
u_boot_logo  
IH_OS_U_BOOT  
u_boot_hush_start
```

---

## 软件配置

### 处理器架构和板卡类型的选择

对于所有支持的板卡，都有现成的默认配置文件。只需输入以下命令即可：

```bash
make <board_name>_defconfig
```

示例：对于 TQM823L 模块，运行以下命令：

```bash
cd u-boot  
make TQM823L_defconfig
```

**注意**：如果您正在寻找某板卡的默认配置文件，但发现其缺失，请检查文件 **doc/README.scrapyard**，其中列出了不再支持的板卡。

---

### 沙箱环境

U-Boot 可以本地构建为在 Linux 主机上运行，使用的是名为 **sandbox** 的板卡。这允许在本地平台上进行与特定板卡或架构无关的功能开发。沙箱环境还可用于运行部分 U-Boot 测试。

更多详情请参考：**doc/arch/sandbox/sandbox.rst**。

### 板级初始化设置

在初始化过程中，U-Boot 调用了多个板级特定的函数，用于在驱动程序初始化之前完成板级特定的准备工作，例如引脚设置。要启用这些回调，需要定义以下配置宏。目前，这些宏是与架构相关的，因此请检查 `arch/your_architecture/lib/board.c` 中的实现，通常在 `board_init_f()` 和 `board_init_r()` 中。

- **`CONFIG_BOARD_EARLY_INIT_F`**：调用 `board_early_init_f()`
- **`CONFIG_BOARD_EARLY_INIT_R`**：调用 `board_early_init_r()`
- **`CONFIG_BOARD_LATE_INIT`**：调用 `board_late_init()`

---

### 配置选项说明

#### 基本配置

- **`CONFIG_SYS_LONGHELP`**  
  如果需要包含详细的帮助信息，请定义此选项；如果内存不足，请取消定义。

- **`CFG_SYS_HELP_CMD_WIDTH`**  
  通过定义此选项，可以覆盖 `help` 命令输出中列出命令的默认宽度。

- **`CONFIG_SYS_PROMPT`**  
  U-Boot 在控制台上打印的提示符，提示用户输入指令。

- **`CFG_SYS_BAUDRATE_TABLE`**  
  列出此板支持的合法波特率设置。

- **`CFG_SYS_MEM_RESERVE_SECURE`**  
  目前仅适用于 ARMv8。如果定义了此选项，将从总内存中减去 `CFG_SYS_MEM_RESERVE_SECURE` 指定的大小，这部分内存将不会被操作系统使用，可作为安全内存。变量 `gd->arch.secure_ram` 用于跟踪其位置。如果系统的 RAM 基址非零，或 RAM 被划分为多个区块，则需要重新计算此变量以获取准确地址。

#### 内存配置

- **`CFG_SYS_SDRAM_BASE`**  
  SDRAM 的物理起始地址。必须设置为 `0`。

- **`CFG_SYS_FLASH_BASE`**  
  Flash 内存的物理起始地址。

- **`CONFIG_SYS_MALLOC_LEN`**  
  为 `malloc()` 使用而保留的 DRAM 大小。

- **`CFG_SYS_BOOTMAPSZ`**  
  Linux 内核启动代码映射的最大内存大小。所有必须被 Linux 内核处理的数据（如 `bd_info`、启动参数、FDT blob 等）必须置于此限制以下，除非定义了 `bootm_low` 环境变量且其值非零。在这种情况下，所有数据必须位于 `bootm_low` 和 `bootm_low + CFG_SYS_BOOTMAPSZ` 之间。  
  - 环境变量 `bootm_mapsize` 将覆盖 `CFG_SYS_BOOTMAPSZ` 的值。  
  - 如果未定义 `CFG_SYS_BOOTMAPSZ`，则使用 `bootm_size` 的值。

- **`CONFIG_SYS_BOOT_GET_CMDLINE`**  
  启用后，内核命令行将被分配并保存在 `bootm_low` 和 `bootm_low + BOOTMAPSZ` 之间的空间中。

- **`CONFIG_SYS_BOOT_GET_KBD`**  
  启用后，`bd_info` 的副本将被分配并保存在 `bootm_low` 和 `bootm_low + BOOTMAPSZ` 之间的空间中。

#### Flash 配置

- **`CONFIG_SYS_FLASH_PROTECTION`**  
  如果定义了此选项，将使用硬件 Flash 扇区保护，而非软件保护。

- **`CONFIG_SYS_FLASH_CFI`**  
  如果 Flash 驱动程序在通用 Flash 结构中使用了额外的元素存储 Flash 几何信息，则定义此选项。

- **`CONFIG_FLASH_CFI_DRIVER`**  
  启用构建 `cfi_flash` 驱动程序（位于 `drivers` 目录中）。

- **`CONFIG_FLASH_CFI_MTD`**  
  启用构建 `cfi_mtd` 驱动程序（位于 `drivers` 目录中），该驱动程序将 CFI Flash 导出到 MTD 层。

- **`CONFIG_SYS_FLASH_USE_BUFFER_WRITE`**  
  使用缓冲写入方式写入 Flash。

#### 环境变量相关

- **`CONFIG_ENV_FLAGS_LIST_DEFAULT`**  
- **`CFG_ENV_FLAGS_LIST_STATIC`**  
  启用对环境变量值的验证，限制变量的值为十进制、十六进制或布尔值等格式。如果定义了 `CONFIG_CMD_NET`，变量还可以限制为 IP 地址或 MAC 地址。

  格式说明：  
  - **类型属性**：`[s|d|x|b|i|m]`  
    - `s`：字符串（默认值）  
    - `d`：十进制  
    - `x`：十六进制  
    - `b`：布尔值  
    - `i`：IP 地址  
    - `m`：MAC 地址  
  - **访问属性**：`[a|r|o|c]`  
    - `a`：任意（默认值）  
    - `r`：只读  
    - `o`：只写一次  
    - `c`：更改默认值  

  示例：  
  - **`CONFIG_ENV_FLAGS_LIST_DEFAULT`**：使用字符串定义默认环境变量的 `.flags`。  
  - **`CFG_ENV_FLAGS_LIST_STATIC`**：定义静态验证列表。如果在 `.flags` 环境变量中未找到条目，则使用静态列表中的验证规则。  

  如果定义了 `CONFIG_REGEX`，变量名将作为正则表达式进行评估，从而允许多个变量共享相同的验证规则。

- **`CONFIG_NAND_ENV_DST`**  
  定义 NAND SPL 代码应将环境复制到的 RAM 地址。如果使用冗余环境，它将被复制到 `CONFIG_NAND_ENV_DST + CONFIG_ENV_SIZE`。

---

### 硬件相关低级配置选项

- **`CONFIG_SYS_CACHELINE_SIZE`**  
  CPU 的缓存行大小。

- **`CONFIG_SYS_CCSRBAR_DEFAULT`**  
  在 Freescale PowerPC SOC 上，CCSR 的默认（上电复位）物理地址。

- **`CFG_SYS_INIT_RAM_ADDR`**  
  用于初始数据和堆栈的内存区域起始地址。注意，该地址必须是无需特殊初始化即可写入的内存区域，例如 CPU 的内部内存。

- **`CONFIG_SYS_SCCR`**  
  系统时钟和复位控制寄存器。

- **`CONFIG_SYS_OR_TIMING_SDRAM`**  
  SDRAM 时序设置。

- **`CONFIG_SYS_NAND_BUSWIDTH_16BIT`**  
  定义 NAND 控制器使用 16 位总线。

- **`CONFIG_FSL_DDR_INTERACTIVE`**  
  启用交互式 DDR 调试模式。

- **`CONFIG_FSL_DDR_BIST`**  
  启用 Freescale DDR 控制器的内置内存测试功能。

---

### 其他选项

- **`CONFIG_CRC32_VERIFY`**  
  为 `crc32` 命令添加验证选项，语法如下：  
  ```bash
  => crc32 -v <address> <count> <crc32>
  ```
  其中 `address/count` 表示内存区域，`crc32` 是该区域应具有的正确校验值。

- **`CONFIG_LOOPW`**  
  添加 `loopw` 内存命令。

- **`CONFIG_CMD_MX_CYCLIC`**  
  添加 `mdc` 和 `mwc` 内存命令，用于循环执行 `md` 和 `mw`。

- **`CONFIG_XPL_BUILD`**  
  指明当前编译的目标是用于 SPL、TPL 或 VPL 构建。

- **`CONFIG_ARCH_MAP_SYSMEM`**  
  启用内存访问的映射功能，通过 `map_sysmem()` 和 `unmap_sysmem()` 进行内存映射。  

---

以上是 U-Boot 配置选项的详细说明，涵盖初始化设置、内存配置、Flash 配置以及低级硬件相关选项等内容。

### 建立软件的流程：
======================

### 构建 U-Boot

U-Boot 已在多种本地构建环境和不同的交叉编译环境中进行过测试。当然，我们无法支持所有可能存在的交叉开发工具版本以及所有可能过时的环境版本。如果遇到工具链问题，建议使用 **ELDK**（详情见 [ELDK 文档](https://www.denx.de/wiki/DULG/ELDK)），它被广泛用于构建和测试 U-Boot。

#### 配置交叉编译工具链
如果您不是在本地环境中构建，假设您的路径中已经包含 GNU 交叉编译工具。在这种情况下，您需要在 shell 中设置环境变量 **`CROSS_COMPILE`**。无需修改 Makefile 或其他源文件。例如，针对 4xx CPU 使用 ELDK 时，执行以下命令：

```bash
$ CROSS_COMPILE=ppc_4xx-
$ export CROSS_COMPILE
```

#### 构建步骤

1. **配置 U-Boot**  
   在安装源代码后，必须为特定的板卡类型配置 U-Boot。通过以下命令完成：

   ```bash
   make NAME_defconfig
   ```

   **`NAME_defconfig`** 是现有配置的名称，可以在 `configs/*_defconfig` 中查看支持的配置名称。

   **注意**：某些板卡可能存在特殊的配置名称，请查看板卡供应商的相关信息。例如，TQM823L 系统可以选择有无 LCD 支持的配置：

   - `make TQM823L_defconfig`：配置为普通 TQM823L（无 LCD 支持）。
   - `make TQM823L_LCD_defconfig`：配置为带 LCD 控制台支持的 TQM823L。

2. **编译 U-Boot**  
   运行以下命令生成 U-Boot 镜像：

   ```bash
   make all
   ```

   编译完成后，将生成以下文件：
   - **`u-boot.bin`**：原始二进制镜像。
   - **`u-boot`**：ELF 格式镜像。
   - **`u-boot.srec`**：Motorola S-Record 格式镜像。

3. **自定义编译参数**  
   用户可以通过设置环境变量 **`KCPPFLAGS`**、**`KAFLAGS`** 和 **`KCFLAGS`** 来传递自定义编译器参数。例如，将所有编译器警告视为错误：

   ```bash
   make KCFLAGS=-Werror
   ```

4. **其他注意事项**  
   Makefile 假设使用 GNU make。因此，在某些系统（如 NetBSD）上，可能需要使用 `gmake` 而非原生的 `make`。

---

### 板卡未列出时的移植步骤：
如果您的系统板卡未在支持列表中，则需要将 U-Boot 移植到您的硬件平台。具体步骤如下：

1. **创建板卡代码目录**  
   创建一个新目录保存您的板卡特定代码。添加所需文件。在该目录中至少需要包含 `Makefile` 和 `<board>.c` 文件。

2. **创建配置文件**  
   创建板卡的配置文件 `include/configs/<board>.h`。

3. **移植到新 CPU（如适用）**  
   如果需要将 U-Boot 移植到新的 CPU，则需要创建一个新的目录以保存 CPU 特定代码，并添加所需文件。

4. **运行配置命令**  
   使用新的板卡名称运行以下命令：

   ```bash
   make <board>_defconfig
   ```

5. **编译生成镜像**  
   执行 `make` 命令生成 `u-boot.srec` 文件，并将其安装到目标系统中。

6. **调试与问题解决**  
   通过调试解决可能的问题（这通常比听起来要复杂得多）。

---

### 测试 U-Boot 的修改、新硬件移植等：
==============================================================

如果您修改了 U-Boot 源代码（例如添加了新板卡支持、新设备支持或新 CPU 支持等），需要向其他开发者提供反馈。反馈通常以“补丁”的形式提交，即基于某个版本的 U-Boot 源代码（官方最新版本或 Git 仓库中最新版本）生成的上下文差异文件（context diff）。

#### 提交补丁前的验证
在提交补丁之前，请验证您的修改不会破坏现有代码。至少确保 **所有支持的板卡** 都可以在没有编译器警告的情况下成功编译。为此，可以运行 `buildman` 脚本进行配置和构建：

```bash
tools/buildman/buildman
```

此脚本将为所有支持的系统配置并编译 U-Boot。**注意**：这可能需要较长时间。有关文档信息，请查看 `buildman` 的 README 文件，或运行以下命令查看帮助：

```bash
buildman -H
```

---

### 监控命令概览：
以下是一些常用的监控命令及其说明：

| **命令**       | **功能**                                                                                     |
|----------------|---------------------------------------------------------------------------------------------|
| `go`           | 从地址 `addr` 启动应用程序。                                                                |
| `run`          | 执行环境变量中的命令。                                                                      |
| `bootm`        | 从内存启动应用镜像。                                                                        |
| `bootp`        | 使用 BootP/TFTP 协议通过网络引导镜像。                                                       |
| `bootz`        | 从内存引导 zImage。                                                                         |
| `tftpboot`     | 使用 TFTP 协议通过网络引导镜像，依赖环境变量 `ipaddr` 和 `serverip`（可能还有 `gatewayip`）。 |
| `tftpput`      | 使用 TFTP 协议通过网络上传文件。                                                             |
| `rarpboot`     | 使用 RARP/TFTP 协议通过网络引导镜像。                                                        |
| `diskboot`     | 从 IDE 设备引导镜像。                                                                        |
| `loads`        | 通过串口加载 S-Record 文件。                                                                 |
| `loadb`        | 通过串口加载二进制文件（kermit 模式）。                                                      |
| `loadm`        | 从源地址加载二进制文件到目标地址。                                                           |
| `md`           | 显示内存内容。                                                                              |
| `mm`           | 修改内存内容（自动递增地址）。                                                              |
| `nm`           | 修改内存内容（地址不变）。                                                                  |
| `mw`           | 写入内存（填充）。                                                                          |
| `ms`           | 搜索内存内容。                                                                              |
| `cp`           | 内存拷贝。                                                                                  |
| `cmp`          | 内存比较。                                                                                  |
| `crc32`        | 计算校验和。                                                                                |
| `i2c`          | I2C 子系统命令。                                                                            |
| `sspi`         | SPI 工具命令。                                                                              |
| `base`         | 打印或设置地址偏移量。                                                                      |
| `printenv`     | 打印环境变量。                                                                              |
| `pwm`          | 控制 PWM 通道。                                                                             |
| `setenv`       | 设置环境变量。                                                                              |
| `saveenv`      | 保存环境变量到持久化存储中。                                                                 |
| `erase`        | 擦除 FLASH 存储。                                                                           |
| `flinfo`       | 打印 FLASH 存储信息。                                                                       |
| `nand`         | NAND 存储操作（详情见文档 `doc/README.nand`）。                                              |
| `bdinfo`       | 打印板卡信息结构。                                                                          |
| `iminfo`       | 打印应用镜像头信息。                                                                        |
| `reset`        | 执行 CPU 重置。                                                                             |
| `help` / `?`   | 打印在线帮助信息。                                                                          |


### 监控命令的详细描述：
========================================

该部分尚待编写。  
**当前解决方案**：直接输入命令获取帮助，例如：

```bash
help <command>
```

---

### 关于冗余以太网接口的注意事项：
========================================

某些板卡可能配备冗余以太网接口。U-Boot 支持这样的配置，并可在需要时自动选择一个“工作正常”的接口。MAC 地址的分配规则如下：

- 网络接口以 **`eth0`**、**`eth1`**、**`eth2`** 等命名。
- 对应的 MAC 地址可以存储在环境变量中，例如 **`ethaddr`** (对应 `eth0`)、**`eth1addr`** (对应 `eth1`)、**`eth2addr`** 等。

#### MAC 地址分配逻辑

1. **使用 SROM 中的默认地址：**  
   如果网络接口的 SROM（串行只读存储器）存储了有效的 MAC 地址且环境变量中没有对应配置，则使用 SROM 的地址。

2. **环境变量覆盖 SROM 配置：**  
   如果环境变量中存在对应的 MAC 地址配置，则优先使用该配置覆盖 SROM 的设置。

3. **SROM 和环境变量地址一致：**  
   如果 SROM 和环境变量中都存在 MAC 地址，并且地址一致，则使用该 MAC 地址。

4. **SROM 和环境变量地址不一致：**  
   如果 SROM 和环境变量中都存在 MAC 地址，但地址不同，则使用环境变量中的值，同时打印警告。

5. **无地址时的行为：**  
   如果 SROM 和环境变量中都没有 MAC 地址，将引发错误。如果定义了 **`CONFIG_NET_RANDOM_ETHADDR`**，则在这种情况下会随机生成一个本地分配的 MAC 地址。

#### 硬件地址的编程
如果以太网驱动程序实现了 **`write_hwaddr`** 功能，则硬件初始化过程中会将有效的 MAC 地址写入硬件。  
可通过设置环境变量 **`ethmacskip`** 跳过此步骤。命名规则如下：

- **`ethmacskip`** (对应 `eth0`)  
- **`eth1macskip`** (对应 `eth1`)  
- 依此类推。

---

### 镜像格式：
==============

U-Boot 支持两种镜像格式，可以启动镜像或执行其他辅助操作：

#### 新的 uImage 格式 (FIT)
基于 **Flattened Image Tree (FIT)** 的灵活且强大的格式（类似于 Flattened Device Tree）。它支持包含多个组件（如多个内核、ramdisk 等）的镜像，且内容可以使用 **SHA1**、**MD5** 或 **CRC32** 进行保护。更多详情请参阅 **`doc/uImage.FIT`** 目录。

#### 旧的 uImage 格式
旧格式基于二进制文件，文件前有特殊的头部。头部定义镜像的以下属性（具体定义见 **`include/image.h`**）：

- **目标操作系统：** 支持的操作系统包括 Linux、NetBSD、VxWorks、QNX 等。
- **目标 CPU 架构：** 支持 ARM、Intel x86、MIPS、Nios II、PowerPC 等架构。
- **压缩类型：** 支持未压缩、gzip 和 bzip2。
- **加载地址和入口点：** 定义镜像加载和启动的内存地址。
- **镜像名称和时间戳：** 用于描述镜像的元数据。

头部包含一个特殊的“Magic Number”，并通过 **CRC32 校验和** 确保头部和数据部分的完整性。

---

### Linux 支持：
==============

尽管 U-Boot 支持任何操作系统或独立应用，其设计过程中始终以支持 Linux 为主要目标。

#### U-Boot 的 Linux 特性
- 将许多传统上属于 Linux 内核的“引导加载程序”代码移至 U-Boot 中。
- 支持独立的内核镜像和 `initrd` 镜像，而非将两者合并为一个大镜像。
- 通过压缩镜像减少 Flash 存储占用，同时提供操作系统无关的功能。
- 简化了新版本 Linux 内核的移植工作，硬件相关的底层操作由 U-Boot 完成。
- 可以轻松使用不同的内核镜像和 `initrd` 组合，便于测试和软件升级。

---

### Linux 使用指南：
============

#### 将 Linux 移植到基于 U-Boot 的系统：
1. **配置设备驱动：**  
   您仍需为目标硬件配置 Linux 设备驱动。

2. **忽略传统引导代码：**  
   不再需要使用内核中的传统引导代码（如 `arch/powerpc/mbxboot`）。

3. **确保关键定义一致：**  
   确保机器特定的头文件（如 **`include/asm-ppc/tqm8xx.h`**）与 U-Boot 的 **`include/asm-<arch>/u-boot.h`** 定义一致。

4. **驱动模型：**  
   U-Boot 提供统一的驱动模型。如果添加新驱动，应集成到驱动模型中。如无现成的 uclass，建议创建一个。

---

#### 配置 Linux 内核：
- 无需特定的 U-Boot 配置。
- 确保目标系统有根文件系统（如初始 ramdisk、NFS）。

---

#### 构建 Linux 镜像：
使用 U-Boot 时，传统的 `zImage` 或 `bzImage` 构建目标不再适用。新的目标 **`uImage`** 会自动生成适合 U-Boot 使用的镜像。  
**示例：**

```bash
make TQM850L_defconfig
make oldconfig
make dep
make uImage
```

#### `mkimage` 工具
`mkimage` 是一个 U-Boot 工具，用于封装压缩 Linux 内核镜像，并添加头部信息、CRC32 校验和等。  
**示例：生成 uImage：**

```bash
${CROSS_COMPILE}-objcopy -O binary -R .note -R .comment -S vmlinux linux.bin
gzip -9 linux.bin
mkimage -A ppc -O linux -T kernel -C gzip -a 0 -e 0 -n "Linux Kernel Image" -d linux.bin.gz uImage
```

`mkimage` 还可用于生成 ramdisk 镜像或组合镜像。通过 **`-l`** 选项可验证镜像头部信息，或使用 **`-d`** 选项创建新镜像。

---

#### 安装 Linux 镜像：
通过串口下载 U-Boot 镜像时，需要将镜像转换为 S-Record 格式：

```bash
objcopy -I binary -O srec examples/image examples/image.srec
```

将镜像安装到地址 `0x40100000` 的示例：

```bash
=> erase 40100000 401FFFFF
=> loads 40100000
~>examples/image.srec
=> imi 40100000
```

---

#### 启动 Linux：
使用 **`bootm`** 命令从内存中启动应用程序（RAM 或 Flash）。例如：

```bash
=> setenv bootargs root=/dev/nfs rw nfsroot=10.0.0.2:/LinuxPPC nfsaddrs=10.0.0.99:10.0.0.2
=> bootm 40020000
```

如果内核使用初始 RAM 磁盘镜像，则同时传递内核和 initrd 的内存地址：

```bash
=> bootm 40100000 40200000
```

### 启动 Linux 并传递平面设备树 (Flat Device Tree)

首先，必须使用适当的定义编译 U-Boot。有关详细信息，请参阅上文 “Linux Kernel Interface” 部分。以下是如何启动内核并传递更新的平面设备树的示例：

```bash
=> print oftaddr
oftaddr=0x300000
=> print oft
oft=oftrees/mpc8540ads.dtb
=> tftp $oftaddr $oft
Speed: 1000, full duplex
Using TSEC0 device
TFTP from server 192.168.1.1; our IP address is 192.168.1.101
Filename 'oftrees/mpc8540ads.dtb'.
Load address: 0x300000
Loading: #
done
Bytes transferred = 4106 (100a hex)
=> tftp $loadaddr $bootfile
Speed: 1000, full duplex
Using TSEC0 device
TFTP from server 192.168.1.1; our IP address is 192.168.1.2
Filename 'uImage'.
Load address: 0x200000
Loading:############
done
Bytes transferred = 1029407 (fb51f hex)
=> print loadaddr
loadaddr=200000
=> print oftaddr
oftaddr=0x300000
=> bootm $loadaddr - $oftaddr
## Booting image at 00200000 ...
   Image Name:	 Linux-2.6.17-dirty
   Image Type:	 PowerPC Linux Kernel Image (gzip compressed)
   Data Size:	 1029343 Bytes = 1005.2 kB
   Load Address: 00000000
   Entry Point:	 00000000
   Verifying Checksum ... OK
   Uncompressing Kernel Image ... OK
Booting using flat device tree at 0x300000
Using MPC85xx ADS machine description
Memory CAM mapping: CAM0=256Mb, CAM1=256Mb, CAM2=0Mb residual: 0Mb
[snip]
```

---

### 更多关于 U-Boot 镜像类型的信息

U-Boot 支持以下几种镜像类型：

1. **独立程序 (Standalone Programs)**  
   这些程序可以直接在 U-Boot 提供的环境中运行。如果程序运行良好，返回后可以继续在 U-Boot 中工作。

2. **操作系统内核镜像 (OS Kernel Images)**  
   通常是嵌入式操作系统的镜像，这些镜像会完全接管系统控制。通常，这些程序会安装自己的异常处理程序、设备驱动程序，并设置 MMU 等。这意味着，通常需要通过重置 CPU 来重新进入 U-Boot。

3. **RAMDisk 镜像 (RAMDisk Images)**  
   这些镜像本质上是数据块，其参数（地址和大小）会被传递给正在启动的操作系统内核。

4. **多文件镜像 (Multi-File Images)**  
   包含多个镜像，例如操作系统（Linux）内核镜像和一个或多个数据镜像（如 RAMDisk）。这种构造在通过网络启动时特别有用，例如使用 BOOTP 等，服务器仅提供一个镜像文件，但需要获取多个镜像（例如内核和 RAMDisk）。

   **多文件镜像结构**：  
   - 起始部分是镜像大小列表，每个大小以 `uint32_t` 格式（网络字节序）表示。
   - 列表以 `(uint32_t)0` 结束。
   - 随后是镜像数据，每个镜像按 `uint32_t` 对齐（大小向上取整到 4 字节的倍数）。

5. **固件镜像 (Firmware Images)**  
   包含固件的二进制镜像（如 U-Boot 或 FPGA 镜像），通常会被编程到闪存。

6. **脚本文件 (Script Files)**  
   包含由 U-Boot 命令解释器执行的命令序列。这种特性在使用 shell（如 hush）作为命令解释器时特别有用。

---

### 启动 Linux `zImage`

在某些平台上，可以使用 `bootz` 命令启动 Linux `zImage`。`bootz` 命令的语法与 `bootm` 命令相同。

- 如果定义了 **`CONFIG_SUPPORT_RAW_INITRD`**，用户可以为内核提供原始 initrd 镜像。  
- 语法稍有不同，`initrd` 的地址必须使用 “地址:大小” 的格式传递，例如：  
  `<initrd 地址>:<initrd 大小>`。

---

### 独立程序使用指南

U-Boot 的一个特性是可以动态加载和运行“独立”应用程序，这些程序可以使用 U-Boot 的一些资源，例如控制台 I/O 功能或中断服务。

#### 示例 1: "Hello World" 演示程序

`examples/hello_world.c` 包含一个简单的 "Hello World" 示例程序。它在构建 U-Boot 时会自动编译。配置为运行在地址 `0x00040004`。使用示例如下：

```bash
=> loads
## Ready for S-Record download ...
~>examples/hello_world.srec
1 2 3 4 5 6 7 8 9 10 11 ...
[file transfer complete]
[connected]
## Start Addr = 0x00040004

=> go 40004 Hello World! This is a test.
## Starting application at 0x00040004 ...
Hello World
argc = 7
argv[0] = "40004"
argv[1] = "Hello"
argv[2] = "World!"
argv[3] = "This"
argv[4] = "is"
argv[5] = "a"
argv[6] = "test."
argv[7] = "<NULL>"
Hit any key to exit ...

## Application terminated, rc = 0x0
```

#### 示例 2: CPM 中断处理程序示例

另一个示例是 `examples/timer.c`，展示了如何在 U-Boot 代码中注册 CPM 中断处理程序。运行后，CPM 定时器每秒生成一次中断，中断服务程序会简单打印 `.`。

---

### U-Boot 的实现细节

U-Boot 的实现受到以下约束的影响：

- U-Boot 通常从 ROM（Flash）启动，启动时尚未初始化系统 RAM。
- 此时没有可写的 Data 或 BSS 段，BSS 段未初始化为零。
- 为了能以 C 语言编写初始化代码，必须至少分配一个最小的堆栈。

#### 初始化限制

- 初始化的全局数据（Data 段）是只读的，不要尝试修改。
- 不要使用未初始化的全局数据或 BSS 段数据。
- 堆栈空间有限，避免使用大数据缓冲区。

#### 全局数据结构 (`gd_t`)

由于内存限制，U-Boot 使用一个全局数据结构（`gd_t`）来共享信息。通过 GCC 的全局寄存器变量功能，将全局数据的指针存储在寄存器中以便访问。

- **PowerPC**: 使用 R2 来存储全局数据指针。
- **ARM**: 使用 R9。
- **Nios II**: 使用 `gp`（全局指针）。
- **RISC-V**: 使用 `gp`。

---

### 系统初始化流程

1. **复位入口点**：  
   U-Boot 从复位入口点启动（例如大多数 PowerPC 系统在地址 `0x00000100`），此时映射的是 Flash 的镜像。

2. **初始堆栈设置**：  
   在 CPU 的内部双端口 RAM 或锁定的数据缓存中设置一个小型初始堆栈。

3. **CPU 和内存映射初始化**：  
   初始化 CPU 内核、缓存和内存控制器，并使用临时配置映射所有可用的内存。

4. **SDRAM 的检测和映射**：  
   使用临时配置，测试 SDRAM 的大小，并按从地址 `0x00000000` 开始的连续内存映射。

5. **监控程序自安装**：  
   在 SDRAM 的高端安装 U-Boot 监控程序，并为 `malloc` 和全局数据分配内存。

6. **代码重定位**：  
   将代码从 ROM 重定位到 RAM，之后进入完整的 C 环境。