### Devicetree 简介

U-Boot 使用 **devicetree** 进行配置。配置内容包括：板载设备、通过 binman 创建镜像的格式、用于控制台的 UART、公钥（用于安全启动）以及其他许多内容。

有关更多信息，请查看 **[Devicetree Control in U-Boot](https://u-boot.readthedocs.io/)**。

---

### 为什么 U-Boot 会将某些内容放入 devicetree？

对于刚接触 U-Boot 的人来说，经常会提出这个问题，尤其是那些来自 Linux 背景的用户，因为他们习惯了 Linux 对 devicetree 内容的严格规定。

U-Boot 使用与 Linux 相同的 devicetree，但额外添加了一些适用于引导加载器环境的内容（请参阅 **[Adding tweaks for U-Boot](https://u-boot.readthedocs.io/)**）。

与 Linux 不同，U-Boot 没有用户空间来提供策略和配置。它无法像 Linux 那样运行程序或查找文件系统来确定如何启动。因此，配置和运行时信息需要放入 U-Boot 的 devicetree 中。

当然，也可以通过以下方法实现类似功能：

1. 将表格数据放入 U-Boot 二进制文件的 `rodata` 段中；
2. 将一些信息以不同格式附加到 U-Boot 的末尾；
3. 修改链接脚本，嵌入包含某些信息的文件；
4. 将内容放入 ACPI 表中；
5. 使用 UEFI 的 hand-off block 结构来存储信息。

**但是，请不要这样做！** 通常情况下，devicetree 是保存 U-Boot 配置的合理选择。事实上，devicetree 是一种非常适合存储 U-Boot 在运行时需要了解的几乎所有信息的数据结构。

因此，请不要再问为什么 U-Boot 会将某些内容放入 devicetree 中。因为这就是最合适的地方！

---

### 关于避免 devicetree 的一些建议

需要注意的是，在 SPL 阶段，可以直接使用 **platdata**，这样驱动程序可以避免依赖 devicetree。不过，现代方法是使用 **of-platdata** 来减少 devicetree 的开销，因此推荐使用这种方式。


### U-Boot 中的 Devicetree 控制

此功能通过**扁平设备树（flattened devicetree，fdt）**实现 U-Boot 的运行时配置。

该功能的目标是使单个 U-Boot 二进制文件支持多个开发板，并通过 fdt 控制每块开发板的具体配置。这种方法与 Linux 内核在 ARM 和 RISC-V 架构上的做法一致，并且已在 PowerPC 上使用了相当长的时间。

fdt 是一种实现运行时配置的便捷工具，原因有以下三点：

1. fdt 已经拥有完善的基础设施：编译器可以检查文本文件，并将其转换为紧凑的二进制格式；U-Boot 提供了一个用于处理该格式的库（`libfdt`）。
2. fdt 具有可扩展性，采用了层次化的节点和属性结构。
3. fdt 的增量读取效率较高。

U-Boot 的 Makefile 基础设施允许构建设备树 blob，并将其嵌入到 U-Boot 镜像中。这种方式非常实用，它允许 U-Boot 根据设备树中的信息自我配置。对于一些具有不同外设但结构相似的开发板，可以通过设备树文件描述每块开发板的特性，从而共享同一套通用代码。

要启用此功能，可通过 **Kconfig** 选择 `OF_CONTROL`。

---

### 什么是扁平设备树（Flattened Devicetree，fdt）？

fdt 可以以文本文件的形式定义为源格式。有关 fdt 语法的详细信息，请参考 **[设备树规范](https://devicetree.org/specifications/)**。

此外，还有一个用于设备树编译器及相关工具的邮件列表。

需要说明的是，`OF` 代表 **Open Firmware**，这一命名遵循了 Linux 的惯例。

---

### 工具

#### 创建扁平设备树
可以使用设备树编译器（`dtc`）创建扁平设备树。U-Boot 默认提供此工具。如果系统中存在 `dtc`（通常在 `device-tree-compiler` 包中），当前版本不会使用系统版本。

如果需要构建自己的 `dtc`，可以从以下地址获取：

```
git://git.kernel.org/pub/scm/utils/dtc/dtc.git
```

#### 编译与解码
可以使用以下命令解码二进制设备树文件：

```bash
dtc -I dtb -O dts <filename.dtb>
```

该仓库还包括 `fdtget`/`fdtput` 工具，用于读取和写入二进制文件中的属性。U-Boot 还添加了 `fdtgrep` 工具，用于创建文件的子集。

---

### 我的开发板的设备树文件在哪里？

设备树文件及其绑定由 Linux 内核 Git 仓库维护。传统上，U-Boot 会将 Linux 内核的设备树源文件副本放置在 `arch/<arch>/dts/<name>.dts` 目录下。然而，这要求每个开发板的维护者手动同步设备树文件，容易导致文件与 Linux 内核版本产生分歧。

U-Boot 现在维护了一个 Git 子树，位于 `dts/upstream/` 子目录中，并定期与 Linux 内核同步，因此无需手动同步设备树文件。你可以在 `dts/upstream/src/<arch>/<vendor>` 中找到适合你开发板的设备树文件。

如果没有现成的文件，你可以查找其他开发板的设备树文件，进行修改后使用。可以在开发板目录下找到 `.dts` 后缀的文件。

如果仍然没有合适的文件，你也可以从头编写！

---

### 同步设备树（Resyncing with devicetree-rebasing）

`devicetree-rebasing` 仓库维护了一份设备树文件及绑定的镜像副本，该副本会在每次 Linux 内核主要版本或中间版本发布时同步。U-Boot 维护者会在每次分支打开时（参阅 **[发布周期](https://u-boot.readthedocs.io/)**）将 `dts/upstream/` 子树与最新的主线 Linux 内核版本同步。

要同步 `dts/upstream/` 子树，可以运行：

```bash
./tools/update-subtree.sh pull dts <devicetree-rebasing-release-tag>
```

如果需要在下次同步前从 `devicetree-rebasing` 仓库中挑选修复内容，可以使用以下命令：

```bash
./tools/update-subtree.sh pick dts <devicetree-rebasing-commit-id>
```

---

### 配置

开发板和 SoC 的维护者被鼓励迁移到 `dts/upstream/src/<arch>/<vendor>` 中的同步副本。要实现这一点，可以通过 Kconfig 添加 `imply OF_UPSTREAM`，并在 Kconfig 提示时设置 `DEFAULT_DEVICE_TREE=<vendor>/<name>`。

如果 `dts/upstream/` 尚未包含你新添加的开发板的设备树文件，你可以选择以下方式：

1. 将设备树文件添加到 `arch/<arch>/dts/<name>.dts`，并在 Kconfig 中设置 `DEFAULT_DEVICE_TREE=<name>`。
2. 使用 `tools/update-subtree.sh` 的 `pick` 选项，从 `devicetree-rebasing` 仓库中拉取所需提交。

对于调试和开发，如果通过 Kconfig 选择了 `OF_EMBED`，设备树文件会嵌入到 U-Boot 镜像（如 `u-boot.bin`）中。不过，这种方式不推荐用于生产设备。

---

### 使用外部设备树文件

如果你想使用自己编译的设备树文件，可以通过 `make` 命令指定：

```bash
make EXT_DTB=<filename>
```

---

### 限制

设备树可以帮助减少支持使用相同 SoC/CPU 的开发板变体的复杂性。但是，U-Boot 的设计是针对单一架构和 CPU 类型构建的。例如，无法构建一个通用的 ARM 二进制文件来同时支持 AT91 和 OMAP 开发板，因为必须在构建时在 `arch/arm/cpu/arm926ejs` 目录中选择其中一种 CPU 系列。

此外，设备树仅用于选择平台/驱动程序中可用的选项，不能（目前）添加新驱动。因此，仍需通过 Kconfig 启用驱动。例如，需要启用 `SYS_NS16550` Kconfig 选项以引入 NS16550 驱动，但可以通过设备树指定 UART 时钟、外设地址等。

---

### 历史

U-Boot 配置以前是通过开发板配置文件中的 `CONFIG` 选项完成的，但随着选项数量接近 10,000 个，这种方法变得难以管理。

U-Boot 在与 Linux 内核大致相同的时间采用了设备树，早期的一些开发板（如 snow）甚至在 Linux 内核之前就开始使用设备树。这两个项目并行发展，某些开发板的绑定仍存在差异。尽管曾讨论过为设备树文件设立单独的仓库，但实际上 Linux 内核 Git 仓库已成为主要的存储位置，U-Boot 通过 `devicetree-rebasing` 仓库同步文件，并通过 `u-boot.dtsi` 文件添加调整。