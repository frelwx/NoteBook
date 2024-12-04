
### QEMU ARM 虚拟化平台中文翻译与优化

#### QEMU ARM 概述
QEMU 支持一个专门为仿真与虚拟化设计的 `virt` 机器平台。本说明文档描述了如何在该平台上运行 U-Boot，该平台同时支持 32 位 ARM 和 AArch64 架构。

**`virt` 平台的基本功能：**
- 可自由配置的 CPU 核心数量
- 在地址 `0x0` 的模拟闪存中加载并运行的 U-Boot
- 放置在 RAM 起始位置的设备树 blob (Device Tree Blob, DTB)
- 可自由配置的 RAM（通过 DTB 描述）
- 可通过 DTB 发现的 PL011 串行端口
- ARMv7/ARMv8 结构化计时器
- 支持系统重启的 PSCI 接口
- 基于 ECAM 的通用 PCI 主机控制器，可通过 DTB 发现

此外，还可以向 PCI 总线添加多个可选外设。

**设备树 (Devicetree) 信息：**  
有关 QEMU 生成的设备树的信息，请参考 [QEMU 中的设备树](https://github.com/qemu/qemu/wiki/Documentation)（Devicetree in QEMU）。

---

#### 构建 U-Boot
按照常规设置 `CROSS_COMPILE` 环境变量，然后运行以下命令：

- **对于 ARM：**
  ```bash
  make qemu_arm_defconfig
  make
  ```
- **对于 AArch64：**
  ```bash
  make qemu_arm64_defconfig
  make
  ```

---

#### 运行 U-Boot
运行 U-Boot 的最小 QEMU 命令行如下：

- **ARM：**
  ```bash
  qemu-system-arm -machine virt -nographic -bios u-boot.bin
  ```
- **AArch64：**
  ```bash
  qemu-system-aarch64 -machine virt -nographic -cpu cortex-a57 -bios u-boot.bin
  ```

> 注意：对于 AArch64，需要显式指定使用 64 位 CPU（例如 `-cpu cortex-a57`），否则默认会以 32 位模式启动。`-nographic` 参数确保输出显示在终端中，通过 `Ctrl-A X` 退出。

---

#### 添加持久化的 U-Boot 环境支持
1. 使用 `qemu-img` 创建环境存储镜像：
   ```bash
   qemu-img create -f raw envstore.img 64M
   ```
2. 在 QEMU 命令行中添加 `pflash` 驱动参数：
   ```bash
   -drive if=pflash,format=raw,index=1,file=envstore.img
   ```

---

#### 添加可选外设
以下是一些在 U-Boot 和 Linux 环境中测试可用的外设及其命令行参数：

1. **视频控制台 (Video Console)：**
   删除 `-nographic` 参数，并添加以下参数：
   ```bash
   -serial stdio -device VGA
   ```

2. **串行 ATA 磁盘 (SATA Disk，Intel ICH9 AHCI 控制器)：**
   ```bash
   -drive if=none,file=disk.img,format=raw,id=mydisk \
   -device ich9-ahci,id=ahci -device ide-drive,drive=mydisk,bus=ahci.0
   ```

3. **Intel E1000 网络适配器：**
   ```bash
   -netdev user,id=net0 -device e1000,netdev=net0
   ```

4. **EHCI 兼容的 USB 主机控制器：**
   ```bash
   -device usb-ehci,id=ehci
   ```

5. **通过 xHCI 控制器连接的 USB 键盘：**
   ```bash
   -device qemu-xhci,id=xhci -device usb-kbd,bus=xhci.0
   ```

6. **NVMe 磁盘：**
   ```bash
   -drive if=none,file=disk.img,id=mydisk -device nvme,drive=mydisk,serial=foo
   ```

7. **随机数生成器：**
   ```bash
   -device virtio-rng-pci
   ```

> **测试环境**：上述配置在 QEMU 2.9.0 中测试通过，理论上兼容 QEMU 2.5.0 及更高版本。

---

#### 引导 Linux 发行版
可通过设置根磁盘安装并引导标准的 Linux 发行版。以下是具体步骤：

1. 创建根磁盘镜像：
   ```bash
   qemu-img create root.img 20G
   ```
2. 使用安装程序（例如 Debian 12）安装系统：
   ```bash
   qemu-system-aarch64 \
     -machine virt -cpu cortex-a53 -m 4G -smp 4 \
     -bios u-boot.bin \
     -serial stdio -device VGA \
     -nic user,model=virtio-net-pci \
     -device virtio-rng-pci \
     -device qemu-xhci,id=xhci \
     -device usb-kbd -device usb-tablet \
     -drive if=virtio,file=debian-12.0.0-arm64-netinst.iso,format=raw,readonly=on,media=cdrom \
     -drive if=virtio,file=root.img,format=raw,media=disk
   ```

**启动过程示例输出：**
```text
U-Boot 2023.10-rc2-00075-gbe8fbe718e35 (Aug 11 2023 - 08:38:49 +0000)

DRAM:  4 GiB
Core:  51 devices, 14 uclasses, devicetree: board
Flash: 64 MiB
Loading Environment from Flash... *** Warning - bad CRC, using default environment

In:    serial,usbkbd
Out:   serial,vidconsole
Err:   serial,vidconsole
Bus xhci_pci: Register 8001040 NbrPorts 8
Starting the controller
USB XHCI 1.00
scanning bus xhci_pci for devices... 3 USB Device(s) found
Net:   eth0: virtio-net#32
Hit any key to stop autoboot:  0
```

安装完成后，移除安装程序的 CD 镜像参数，再次运行 QEMU 即可引导到已安装的系统。

---

#### 启用 TPMv2 支持
可通过 `swtpm` 仿真 TPM。以下是具体步骤：

1. 克隆并构建 `swtpm`：
   ```bash
   git clone https://github.com/stefanberger/swtpm.git
   cd swtpm
   ./autogen.sh && ./configure && make && sudo make install
   ```

2. 在第一个终端中启动 `swtpm`：
   ```bash
   swtpm socket --tpmstate dir=/tmp/mytpm1 \
   --ctrl type=unixio,path=/tmp/mytpm1/swtpm-sock --log level=20
   ```

3. 在第二个终端中运行 QEMU：
   ```bash
   qemu-system-aarch64 \
     -chardev socket,id=chrtpm,path=/tmp/mytpm1/swtpm-sock \
     -tpmdev emulator,id=tpm0,chardev=chrtpm \
     -device tpm-tis-device,tpmdev=tpm0
   ```

4. 在 U-Boot 命令行中启用 TPM：
   ```bash
   tpm autostart
   ```

---

#### 调试 UART
ARM `virt` 平台上的调试 UART 使用以下配置：

```text
CONFIG_DEBUG_UART=y
CONFIG_DEBUG_UART_PL010=y
CONFIG_DEBUG_UART_BASE=0x9000000
CONFIG_DEBUG_UART_CLOCK=0
```


### QEMU 中的设备树 (Devicetree)

在 QEMU 的 ARM、RISC-V 和部分 PPC 目标平台上，设备树 (Devicetree) 是由 QEMU 动态生成的。它的主要用途是为 Linux 提供硬件描述，但也可以被 U-Boot 使用，只要将 U-Boot 所需的节点和属性合并进去即可。

---

#### 启用 `CONFIG_OF_BOARD`

如果在 U-Boot 中启用了 `CONFIG_OF_BOARD` 配置选项，则 U-Boot 可以直接处理 QEMU 动态生成的设备树。

---

### 如何获取 QEMU 生成的设备树

当 QEMU 生成自己的设备树传递给 U-Boot 时，可以使用 `-dtb u-boot.dtb` 参数强制 QEMU 使用 U-Boot 的内置设备树版本。

若想直接获取 QEMU 生成的设备树，可以通过以下命令将设备树导出为 `.dtb` 文件：

- **ARM 平台：**
  ```bash
  qemu-system-arm -machine virt -machine dumpdtb=qemu.dtb
  ```
- **AArch64 平台：**
  ```bash
  qemu-system-aarch64 -machine virt -machine dumpdtb=qemu.dtb
  ```
- **RISC-V 平台：**
  ```bash
  qemu-system-riscv64 -machine virt -machine dumpdtb=qemu.dtb
  ```

---

### 合并 U-Boot 所需的节点和属性

目前，QEMU 对 U-Boot 所需的特定节点和属性尚不了解。因此，为了使用这些功能，需要手动将必要的节点和属性合并到设备树中。

**使用 `dtc` 工具合并设备树的示例：**

以下命令使用 `dtc`（设备树编译器）将 QEMU 生成的设备树和 U-Boot 的设备树合并：

1. 导出 QEMU 生成的设备树：
   ```bash
   qemu-system-arm -machine virt -machine dumpdtb=qemu.dtb
   ```

2. 使用 `dtc` 将 `.dtb` 文件转换为文本格式，去除重复的头部信息，并合并文件：
   ```bash
   cat <(dtc -I dtb qemu.dtb) <(dtc -I dtb u-boot.dtb | grep -v /dts-v1/) | dtc - -o merged.dtb
   ```

3. 使用合并后的设备树运行 QEMU：
   ```bash
   qemu-system-arm -machine virt -nographic -bios u-boot.bin -dtb merged.dtb
   ```

---

### 注意事项

1. **设备树的兼容性问题：**  
   某些版本的 QEMU 存在一个已知问题：`dumpdtb` 导出的设备树可能与实际提供给 U-Boot 的设备树不完全匹配。如果在使用中发现问题，可以尝试对设备树的内容进行手动调整或升级 QEMU 版本。

2. **确保节点完整性：**  
   在合并设备树时，请特别注意 U-Boot 所需的关键节点和属性是否已成功加入合并后的设备树中，例如内存描述、串口节点等。

通过上述方法，可以高效地使用 QEMU 生成的设备树并合并 U-Boot 所需的扩展功能。