
### U-Boot 中的日志记录功能

#### 简介

U-Boot 的内部操作涉及多个步骤和操作，从设置板级硬件到显示启动界面，再到加载操作系统，包含许多组件和行为。在大多数情况下，这些内部细节对用户并无帮助，直接显示在控制台上会延迟启动（启动是 U-Boot 的主要任务）并可能让用户感到困惑。

然而，当需要调试特定问题或了解某个模块的详细运行情况时，查看 U-Boot 更详细的操作日志是非常有用的。U-Boot 的日志功能旨在满足用户和开发者的这一需求。

---

#### 日志级别

U-Boot 提供了多个日志级别。  
具体参见 `enum log_level_t`。

---

#### 日志分类

U-Boot 中的日志可能来自多个模块或组件。每条日志消息都带有一个分类属性，用于根据来源对日志进行过滤。

具体参见 `enum log_category_t`。

---

#### 启用日志功能

以下选项用于在编译时启用日志功能：

- **`CONFIG_LOG`**：启用日志系统。
- **`CONFIG_LOG_MAX_LEVEL`**：设置编译时支持的最大日志级别（超过此级别的日志将被编译时剔除）。
- **`CONFIG_LOG_CONSOLE`**：启用将日志记录写入控制台的功能。

如果未定义 **`CONFIG_LOG`**，日志功能将不可用。

以上选项还支持 SPL 和 TPL 的版本，例如：`CONFIG_SPL_LOG_MAX_LEVEL` 和 `CONFIG_TPL_LOG_MAX_LEVEL`。

- **日志禁用时的默认行为**：
  - 默认情况下，日志级别为 `LOGL_INFO` 及以下的消息将直接输出。
  - 如果日志被禁用且在源文件顶部定义了 `DEBUG`，则日志级别为 `LOGL_DEBUG` 的消息也将被输出。

---

#### 在单个文件中临时启用日志

有时仅需要在某个文件中临时启用日志记录，可以在文件顶部添加以下代码：

```c
#define LOG_DEBUG
```

将其放置在任何 `#include` 语句之前。这将强制编译并输出该文件中所有日志消息，而无论全局配置 **`CONFIG_LOG_DEFAULT_LEVEL`** 如何设置。

---

#### 使用 `DEBUG`

U-Boot 传统上使用 `#define DEBUG` 来在文件级别启用调试功能，但推荐逐步用 `LOG_DEBUG` 替代它。  

- 当启用日志功能时，`debug()` 语句将被解释为日志记录，日志级别为 `LOGL_DEBUG`，日志分类为 `LOG_CATEGORY`。
- 当禁用日志功能时：
  - 如果定义了 `DEBUG`，`debug()` 宏将被编译为 `printf()` 语句。
  - 如果未定义 `DEBUG`，`debug()` 宏将被编译为空语句。

---

#### 日志语句

日志的主要函数为：

```c
log(category, level, format_string, ...)
```

此外，`debug()` 和 `error()` 也会生成日志记录。这些函数使用 `LOG_CATEGORY` 作为分类，因此需要在源文件顶部定义该分类：

```c
#define LOG_CATEGORY LOGC_ALLOC
```

日志消息的格式字符串通常以换行符 `\n` 结尾。如果没有换行符，则下一条日志语句会带上 `LOGRECF_CONT` 标志，将日志内容继续写在同一行，而不会重复输出分类和级别等信息。这种行为由 `log_console` 实现。例如：

```c
log_debug("Here is a list:");
for (i = 0; i < count; i++)
    log_debug(" item %d", i);
log_debug("\n");
```

此外，还提供特殊的日志分类 `LOGL_CONT` 和日志级别 `LOGC_CONT`，用于处理上述情况。

---

#### 错误返回日志

通过定义 **`CONFIG_LOG_ERROR_RETURN`**，可以启用 `log_ret()` 宏。当函数返回错误值时，此宏会记录一条日志。例如：

```c
return log_ret(uclass_first_device_err(UCLASS_MMC, &dev));
```

当检测到错误代码（小于 0 的值）时，此宏将记录日志，便于追踪深层调用栈中的错误。

另外，`log_msg_ret()` 变体允许打印短字符串，帮助定位函数中具体的失败调用。例如：

```c
ret = gpio_request_by_name(dev, "cd-gpios", 0, &desc, GPIOD_IS_IN);
if (ret)
    return log_msg_ret("gpio", ret);
```

对于返回值为 `0` 表示成功，非零值表示错误的函数，可以使用 `log_retz()` 和 `log_msg_retz()`。

---

#### 便捷函数

为了简化日志记录代码，U-Boot 提供了一些便捷函数，可直接根据日志级别命名调用：

- **`log_err(_fmt...)`**  
- **`log_warning(_fmt...)`**  
- **`log_notice(_fmt...)`**  
- **`log_info(_fmt...)`**  
- **`log_debug(_fmt...)`**  
- **`log_content(_fmt...)`**  
- **`log_io(_fmt...)`**

这些函数中，日志级别由函数名称隐含。日志分类由 `LOG_CATEGORY` 决定，需在文件顶部定义一次，例如：

```c
#define LOG_CATEGORY LOGC_ALLOC
```

或者：

```c
#define LOG_CATEGORY UCLASS_SPI
```

**注意**：所有 U-Boot 的 uclass ID 也是日志分类。

---

### 总结

U-Boot 的日志功能提供了灵活的日志级别、多样化的日志分类以及便捷的日志记录方式。通过合理使用日志功能，开发者可以更高效地调试和分析 U-Boot 的内部行为，同时避免对启动性能产生不必要的影响。