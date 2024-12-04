### 绑定/解绑驱动

本文档旨在描述 `bind` 和 `unbind` 命令的使用方法。

---

在调试过程中，从 U-Boot 命令行绑定或解绑驱动程序可能非常有用。

- **`unbind` 命令**  
  调用设备驱动的 `remove` 回调函数，并将设备与驱动程序解绑。

- **`bind` 命令**  
  将设备绑定到驱动程序。

在某些情况下，从命令行绑定设备到驱动可能非常实用。例如：
1. **多功能设备**：如 USB gadget 设备。
2. **延迟绑定场景**：某些设备在启动时尚未就绪，需要完成一些设置后才绑定驱动（例如，FPGA 的比特流从大容量存储设备或以太网中加载）。

---

### 使用方法

#### `bind` 命令
```bash
bind <node path> <driver>
bind <class> <index> <driver>
```

#### `unbind` 命令
```bash
unbind <node path>
unbind <class> <index>
unbind <class> <index> <driver>
```

---

### 参数说明

- **`<node path>`**  
  设备树中节点的路径。

- **`<class>`**  
  可用的设备类，可以通过 `dm uclass` 命令或 `dm tree` 命令输出的第一列获取。

- **`<index>`**  
  父节点的索引值，可以通过 `dm tree` 命令输出的第二列获取。

- **`<driver>`**  
  驱动名称，可以通过 `dm drivers` 命令或 `dm tree` 命令输出的第四列获取。

---

### 示例

#### 使用设备类和索引
- 绑定设备：  
  ```bash
  bind usb_dev_generic 0 usb_ether
  ```
- 解绑设备：  
  ```bash
  unbind usb_dev_generic 0 usb_ether
  ```
  或者：
  ```bash
  unbind eth 1
  ```

#### 使用设备树节点路径
- 绑定设备：  
  ```bash
  bind /ocp/omap_dwc3@48380000/usb@48390000 usb_ether
  ```
- 解绑设备：  
  ```bash
  unbind /ocp/omap_dwc3@48380000/usb@48390000
  ```

---

通过以上命令，您可以在调试过程中灵活地绑定或解绑设备驱动，满足不同的调试需求。

### 调试驱动模型

本文档旨在帮助您解决驱动模型未按预期工作的情况。

---

### 常用的调试技巧

以下是一些常用的调试技巧：

1. **在沙盒环境中开发新特性**  
   如果您正在开发新功能，建议在沙盒环境中进行，而不是直接在开发板上操作。沙盒环境没有硬件限制，便于调试（例如使用 gdb），并且可以为大多数常见设备编写模拟器。

2. **启用文件中的调试输出**  
   在文件顶部加入 `#define DEBUG`，即可激活文件中所有的 `debug()` 和 `log_debug()` 调试语句。

3. **调整日志级别**  
   如果使用了日志记录功能，可以通过以下配置调整日志级别：
   - 在 SPL 中，设置 `CONFIG_SPL_LOG_MAX_LEVEL=7`（对应 `LOGL_DEBUG`）
   - 设置 `CONFIG_LOG_DEFAULT_LEVEL=7`，以启用最高日志级别。

4. **记录返回值的日志**  
   如果通过 `log_msg_ret()` 实现了返回值的日志记录，可以设置 `CONFIG_LOG_ERROR_RETURN=y`，以便准确定位错误发生的位置。

5. **启用调试 UART**  
   确保已启用调试 UART（参见 `CONFIG_DEBUG_UART`）。通过调试 UART，您可以在串口驱动运行之前获取串口输出（例如 `printf()` 等）。

6. **使用 JTAG 仿真器**  
   通过 JTAG 仿真器设置断点并单步调试代码。

> 注意：启用上述调试选项会在一定程度上增加代码和数据的大小。

---

### 无法找到设备

假设您调用了 `uclass_first_device_err()`，但未能找到任何设备。

#### 返回错误代码的情况

如果函数返回了错误代码，这可以为您提供一些线索。请查阅 `linux/errno.h` 文件中的错误定义。常见的错误包括：

- **-ENOMEM**  
  表示内存不足。如果该错误发生在 SPL 或 U-Boot 重定位之前，请检查 `CONFIG_SPL_SYS_MALLOC_F_LEN` 和 `CONFIG_SYS_MALLOC_F_LEN` 是否需要增大。在 `malloc_simple.c` 文件顶部加入 `#define DEBUG`，可以帮助您了解内存的分配情况。

- **-EINVAL**  
  通常表示设备树节点中缺少某些内容或存在错误。请检查设备树节点是否正确，并查看驱动中的 `of_to_plat()` 方法。

#### 无错误返回的情况

如果函数未返回错误，则需要检查设备是否已绑定：

1. **调用 dm_dump_tree()**  
   在尝试定位设备之前调用 `dm_dump_tree()`，以确保设备存在于设备树中。

2. **检查设备树兼容字符串**  
   如果设备不存在，请检查设备树中的兼容字符串是否与驱动所期望的字符串匹配（在 `struct udevice_id` 数组中定义）。

3. **检查 of-platdata**  
   如果使用了 `of-platdata`（例如启用了 `CONFIG_SPL_OF_PLATDATA`），请确保驱动名称与设备树中第一个兼容字符串一致（无效变量字符需转换为下划线）。

4. **查看核心列表日志**  
   如果仍然无法解决问题，请在 `drivers/core/lists.c` 文件顶部加入 `#define LOG_DEBUG`，以查看具体的运行情况。

### 设计细节

本 README 包含关于驱动模型的高级信息，这是在 U-Boot 中声明和访问驱动的一种统一方式。最初的工作由以下人员完成：

- **Marek Vasut** (<marex@denx.de>)  
- **Pavel Herrmann** (<morpheus.ibis@gmail.com>)  
- **Viktor Křivák** (<viktor.krivak@gmail.com>)  
- **Tomas Hlavacek** (<tmshlvck@gmail.com>)  

目前的实现经过 **简化和扩展**，由以下人员完成：

- **Simon Glass** (<sjg@chromium.org>)  

---

### 术语

#### **Uclass（类）**
指一组以相同方式操作的设备。Uclass 提供了一种访问组内各个设备的方式，但始终使用相同的接口。例如：  
- **GPIO Uclass** 提供 `get/set` 值的操作。  
- **I2C Uclass** 可能包含 10 个 I2C 端口，其中 4 个使用一个驱动，另外 6 个使用另一个驱动。

#### **Driver（驱动）**
指与某个外设通信的代码，并向其提供一个更高级别的接口。

#### **Device（设备）**
某个驱动的一个实例，绑定到特定的端口或外设。

---

### 如何尝试

构建 U-Boot 的 sandbox（沙箱环境）并运行：

```bash
make sandbox_defconfig
make
./u-boot -d u-boot.dtb
```

（输入 `reset` 退出 U-Boot）

---

### 示例：demo 类

U-Boot 中有一个名为 `demo` 的 uclass。该 uclass 负责输出问候语并报告其状态。在该 uclass 中有两个驱动：

1. **simple**：仅输出问候语消息，不实现状态功能。  
2. **shape**：输出图形，同时以打印的字符数作为状态信息。

`demo` 类非常简单，但并非完全无意义。它的设计目的是用于测试，因此实现了所有驱动模型功能，并提供了良好的代码覆盖率。其特点包括：  
- 包含多个驱动  
- 处理参数数据和平台特定数据（`plat`，用于告诉驱动在特定平台上如何操作）  
- 使用私有驱动数据（`private driver data`）

要尝试它，请参阅以下示例会话：