
### 使用 GCC 构建 U-Boot

---

### 依赖项

构建 U-Boot 需要一个适用于主机平台的 **GCC 编译器**。如果不是在目标平台上构建，还需要一个 **GCC 交叉编译器**。

#### **Debian 系统**

在基于 Debian 的系统上，交叉编译器的包名为 `gcc-<architecture>-linux-gnu`。

例如，安装适用于 ARMv8 架构的 GCC 和 GCC 交叉编译器：

```bash
sudo apt-get install gcc gcc-aarch64-linux-gnu
```

根据构建目标，还可能需要安装以下额外的包：

```bash
sudo apt-get install bc bison build-essential coccinelle \
  device-tree-compiler dfu-util efitools flex gdisk graphviz imagemagick \
  liblz4-tool libgnutls28-dev libguestfs-tools libncurses-dev \
  libpython3-dev libsdl2-dev libssl-dev lz4 lzma lzma-alone openssl \
  pkg-config python3 python3-asteval python3-coverage python3-filelock \
  python3-pkg-resources python3-pycryptodome python3-pyelftools \
  python3-pytest python3-pytest-xdist python3-sphinxcontrib.apidoc \
  python3-sphinx-rtd-theme python3-subunit python3-testtools \
  python3-virtualenv swig uuid-dev
```

#### **SUSE 系统**

在基于 SUSE 的系统上，交叉编译器的包名为 `cross-<architecture>-gcc<version>`。

例如，安装适用于 ARMv8 架构的 GCC 和 GCC 10 交叉编译器：

```bash
sudo zypper install gcc cross-aarch64-gcc10
```

根据构建目标，还可能需要安装以下额外的包：

```bash
zypper install bc bison flex gcc libopenssl-devel libSDL2-devel make \
  ncurses-devel python3-devel python3-pytest swig
```

#### **Alpine Linux**

在 Alpine Linux 上构建 U-Boot 至少需要以下包：

```bash
apk add alpine-sdk bc bison dtc flex gnutls-dev linux-headers ncurses-dev \
  openssl-dev py3-elftools py3-setuptools python3-dev swig util-linux-dev
```

根据构建目标可能需要以下额外包：

- **带 LCD 的沙箱环境**：`sdl2-dev`
- **RISC-V S 模式目标**：`opensbi`
- **部分 ARM64 目标**：`arm-trusted-firmware`

---

### 前置要求

对于某些开发板，在构建 U-Boot 之前需要先构建一些前置文件。例如，对于某些开发板，需要事先构建 ARM Trusted Firmware。具体请参考开发板的文档 **Board-specific doc**。

---

### 配置

`configs/` 目录包含了维护的开发板的模板配置文件，文件名遵循以下命名规则：

```
<board name>_defconfig
```

这些文件已移除了默认设置，因此无法直接使用。它们的名称可用作 `make` 的目标，用于生成实际的配置文件 `.config`。例如，Odroid C2 开发板的配置模板文件名为 `odroid-c2_defconfig`，对应的 `.config` 文件可通过以下命令生成：

```bash
make odroid-c2_defconfig
```

你可以使用以下命令调整配置：

```bash
make menuconfig
```

---

### 构建

在交叉编译时，需要指定交叉编译器的前缀。可以通过 `make` 命令行设置 `CROSS_COMPILE` 变量，或者提前通过 `export` 设置。

例如，在 Debian 系统上为 ARMv8 进行交叉编译：

```bash
CROSS_COMPILE=aarch64-linux-gnu- make
```

---

#### **树外构建（Out-of-tree building）**

默认情况下，构建过程会在源码目录中生成目标文件。为了保持源码目录干净，可以选择树外构建，方法如下：

1. 在 `make` 命令行中添加 `O=` 参数：

   ```bash
   make O=/tmp/build distclean
   make O=/tmp/build NAME_defconfig
   make O=/tmp/build
   ```

2. 使用环境变量 `KBUILD_OUTPUT`：

   ```bash
   export KBUILD_OUTPUT=/tmp/build
   make distclean
   make NAME_defconfig
   make
   ```

**注意**：命令行参数 `O=` 优先级高于环境变量 `KBUILD_OUTPUT`。

---

#### **构建参数**

可以通过以下命令获取所有可用的 `make` 参数列表：

```bash
make help
```

常用的构建参数包括：

- `O=<dir>`：将所有输出文件存放在 `<dir>` 目录中，包括 `.config` 文件。
- `V=1`：启用详细日志模式。
- `-j`：启用并行编译以加快速度，例如：

  ```bash
  CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc)
  ```

---

#### **设备树编译器（Devicetree Compiler）**

使用 `CONFIG_OF_CONTROL` 的开发板（几乎所有开发板）需要设备树编译器（`dtc`）。如果启用了 `CONFIG_PYLIBFDT`，还需要 `pylibfdt`（一个用于访问设备树数据的 Python 库）。U-Boot 中自带了合适版本的这些工具，位于 `scripts/dtc`，在需要时会自动构建。

如果希望使用系统版本的工具，可以通过 `DTC` 参数指定，例如：

```bash
DTC=/usr/bin/dtc make
```

在这种情况下，`dtc` 和 `pylibfdt` 不会被重新构建。

**注意**：主机工具始终使用包含的 `libfdt` 版本，因此目前无法使用系统版本的 `libfdt` 构建 U-Boot 工具。

---

### 链接时优化（LTO）

U-Boot 支持 **链接时优化（Link-Time Optimization, LTO）**，可以显著减小最终 U-Boot 二进制文件的大小，尤其是 SPL。

目前，ARM 开发板可以通过在配置文件中添加以下选项启用 LTO：

```text
CONFIG_LTO=y
```

沙箱环境默认启用 LTO。启用 LTO 会增加链接时间（通常为几秒钟）。在开发过程中，为了加快增量构建速度，可以通过以下命令禁用 LTO：

```bash
NO_LTO=1 make
```

---

### 其他构建目标

通过以下命令可以获取所有 `make` 构建目标的列表：

```bash
make help
```

常用构建目标包括：

- `clean`：移除大部分生成的文件，但保留配置文件。
- `mrproper`：移除所有生成的文件、配置文件及各种备份文件。

---

### 安装

将 U-Boot 安装到目标设备的过程因设备而异。请参考开发板的文档 **Board-specific doc**。


### 主机工具的构建指南

---

#### **在 Linux 上构建工具**

为了让发行版以通用方式分发所有可能的工具，避免针对每台机器单独构建特定工具，提供了一个仅用于工具的配置文件 `tools-only_defconfig`。

使用该配置文件可以按照以下步骤构建工具：

```bash
$ make tools-only_defconfig
$ make tools-only
```

---

#### **在 Windows 上构建工具**

如果需要为 Windows 生成 `tools` 目录中的工具，可以使用 **MSYS2**（一个适用于 Windows 的软件发行和构建平台）。

1. **下载 MSYS2 安装程序**

   从 [MSYS2 官方网站](https://www.msys2.org) 下载 MSYS2 安装程序。

2. **安装所需的包**

   确保安装了下列所需的软件包以便构建这些主机工具：

   - `gcc`（例如版本 9.1.0）
   - `make`（例如版本 4.2.1）
   - `bison`（例如版本 3.4.2）
   - `diffutils`（例如版本 3.7）
   - `openssl-devel`（例如版本 1.1.1.d）

   **注意**：上述括号中的版本号为撰写本文时的软件包版本。

   已测试的 MSYS2 安装程序版本为：  
   [http://repo.msys2.org/distrib/x86_64/msys2-x86_64-20190524.exe](http://repo.msys2.org/distrib/x86_64/msys2-x86_64-20190524.exe)

3. **了解 MSYS2 的子系统**

   MSYS2 安装后提供以下 3 个子系统：

   - **MSYS2 环境**：用于在 Windows 上构建 POSIX 兼容的软件，依赖一个仿真层。
   - **MinGW32 子系统**：用于构建原生 Windows 应用程序，针对 32 位 Windows。
   - **MinGW64 子系统**：用于构建原生 Windows 应用程序，针对 64 位 Windows。

   每个子系统使用 GNU/Linux 工具链（如 `gcc` 和 `bash`）提供不同的构建环境。

4. **启动 MSYS2 Shell 并构建工具**

   打开 **MSYS2 Shell**（MSYS2 环境），运行以下命令：

   ```bash
   $ make tools-only_defconfig
   $ make tools-only
   ```

---

#### **无 Python 环境的构建**

默认情况下，`tools-only` 构建会生成 `pylibfdt`。如果需要禁用 Python 支持，可以通过设置 `NO_PYTHON` 变量完成：

```bash
$ NO_PYTHON=1 make tools-only_defconfig tools-only
```

### 构建 U-Boot 文档指南

U-Boot 文档基于 **Sphinx** 文档生成器构建。以下是构建文档所需的依赖和步骤。

---

### **依赖项**

除了 `doc/sphinx/requirements.txt` 中列出的 Python 包外，以下依赖也需要安装：

- `fontconfig`
- `graphviz`
- `imagemagick`
- `texinfo`（如果需要构建 Infodoc 文档）

---

### **HTML 文档**

HTML 文档通过 `htmldocs` 目标构建，使用了 **Read the Docs** Sphinx 主题。

#### **构建步骤**

1. 创建 Python 虚拟环境：

   ```bash
   python3 -m venv myenv
   ```

2. 激活虚拟环境：

   ```bash
   . myenv/bin/activate
   ```

3. 安装构建依赖：

   ```bash
   python3 -m pip install -r doc/sphinx/requirements.txt
   ```

4. 构建 HTML 文档：

   ```bash
   make htmldocs
   ```

5. 停止使用虚拟环境：

   ```bash
   deactivate
   ```

6. 使用图形化浏览器查看文档：

   ```bash
   x-www-browser doc/output/index.html
   ```

#### **HTML 文档发布**

HTML 文档发布在 [https://docs.u-boot.org](https://docs.u-boot.org)。其构建过程由 `.readthedocs.yml` 文件控制。

---

### **Infodoc 文档**

Infodoc 文档构建会生成 **texinfo 文件** 和 **info 文件**。

#### **构建步骤**

1. 创建并激活 Python 虚拟环境：

   ```bash
   python3 -m venv myenv
   . myenv/bin/activate
   ```

2. 安装构建依赖：

   ```bash
   python3 -m pip install -r doc/sphinx/requirements.txt
   ```

3. 构建 Infodoc 文档：

   ```bash
   make infodocs
   ```

4. 停止使用虚拟环境：

   ```bash
   deactivate
   ```

5. 查看生成的文档：

   ```bash
   info doc/output/texinfo/u-boot.info
   ```

---

### **PDF 文档**

PDF 文档通过 `pdfdocs` 目标构建。然而，从 **v2023.01** 版本开始，该目标可能会因 **LaTeX 错误：嵌套太深** 而失败。

#### **解决方法：使用 texi2pdf**

1. 创建并激活 Python 虚拟环境：

   ```bash
   python3 -m venv myenv
   . myenv/bin/activate
   ```

2. 安装构建依赖：

   ```bash
   python3 -m pip install -r doc/sphinx/requirements.txt
   ```

3. 构建 texinfo 文档：

   ```bash
   make texinfodocs
   ```

4. 停止使用虚拟环境：

   ```bash
   deactivate
   ```

5. 转换为 PDF：

   ```bash
   texi2pdf doc/output/texinfo/u-boot.texi
   ```

---

### **Texinfo 文档**

Texinfo 文档可通过 `texinfodocs` 目标单独构建。

#### **构建步骤**

1. 创建并激活 Python 虚拟环境：

   ```bash
   python3 -m venv myenv
   . myenv/bin/activate
   ```

2. 安装构建依赖：

   ```bash
   python3 -m pip install -r doc/sphinx/requirements.txt
   ```

3. 构建 texinfo 文档：

   ```bash
   make texinfodocs
   ```

4. 停止使用虚拟环境：

   ```bash
   deactivate
   ```

#### **输出文件**

生成的文件位于 `doc/output/texinfo/u-boot.texi`。

---

### **总结**

根据需求选择目标构建所需的文档类型（HTML、Infodoc、PDF 或 Texinfo）。通过 Python 虚拟环境和 Sphinx 工具，可以轻松完成文档的构建与管理。