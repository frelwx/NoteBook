

早期的 ARM 架构实现严格按照程序顺序执行所有指令，并且每条指令在下一条指令开始之前会完全执行完毕。

新一代处理器采用了多种优化技术，这些优化与指令的执行顺序和内存访问方式有关。如我们所知，核心执行指令的速度显著高于外部内存的速度。为了部分掩盖这种速度差异所带来的延迟，使用了缓存和写缓冲区。其潜在的一个影响是内存访问的重新排序。核心执行的加载和存储指令的顺序不一定与外部设备看到的访问顺序相同。


以下是一个程序顺序的示例：

```assembly
STR R12, [R1]        @ 访问1：存储操作
LDR R0, [SP], #4     @ 访问2：加载操作
LDR R2, [R3,#8]      @ 访问3：加载操作
```


在图 10-1 中，列出了按程序顺序执行的三条指令。第一条指令执行对外部内存的写操作，在本例中写入了写缓冲区（访问1）。随后按程序顺序执行了两次读取操作，一次在缓存中未命中（访问2），另一次在缓存中命中（访问3）。两个读取操作可能会在写缓冲区完成访问1的写操作之前完成。缓存中的“命中-未命中”行为意味着缓存中命中的加载（如访问3）可以在程序中较早的未命中加载（如访问2）之前完成。

尽管如此，仍然可以保持硬件按你编写的顺序执行指令的假象。通常，只有在少数情况下才需要关注这种效应。例如，如果你修改 CP15 寄存器、复制或更改内存中的代码，可能需要显式地让核心等待这些操作完成。

对于支持推测性数据访问、多发射指令、缓存一致性协议和乱序执行的高性能核心，为了提高性能，存在更多重新排序的可能性。通常，在单核系统中，这种重新排序的效果对你是不可见的。硬件会处理许多潜在的风险，确保数据依赖关系得到遵守，并在读取时返回正确的值，考虑到早期写操作可能引起的修改。

然而，在多核系统中，当多个内核通过共享内存通信（或以其他方式共享数据）时，内存排序的考虑变得更加重要。通常，你最关心内存排序的情况是当多个执行线程必须同步时。


符合 ARMv7-A 架构的处理器采用了一种**弱排序**的内存模型，这意味着对于加载和存储操作，内存访问的顺序不要求与程序顺序相同。该模型可以重新排序内存读取操作（如 LDR、LDM 和 LDD 指令），相对于其他读取、存储操作和某些其他指令。硬件可以对普通内存的读取和写入进行重新排序，这种重新排序仅受数据依赖性和显式内存屏障指令的限制。在需要更强的排序规则的情况下，可以通过描述该内存的翻译表条目中的内存类型属性将这些规则传达给核心。

对核心强制排序规则会限制可能的硬件优化，因此会降低性能并增加功耗。

### 10.1 ARM 内存排序模型

Cortex-A 系列处理器采用了一种**弱排序**的内存模型。然而，在这个模型中，特定的内存区域可以标记为**强序内存**（Strongly-ordered）。在这种情况下，内存事务将保证按照它们发布的顺序发生。

ARM 架构定义了三种互斥的内存类型。所有内存区域都被配置为以下三种类型之一：

1. **强序内存 (Strongly-ordered)**  
2. **设备内存 (Device)**  
3. **普通内存 (Normal)**  

此外，对于**普通内存**，可以指定内存是否是**可共享的**（即可以被其他代理访问）。对于普通内存，还可以指定内存的**内层缓存**和**外层缓存**属性。

在表 10-1 中，A1 和 A2 是对不同地址的两个访问。A1 在程序代码中发生在 A2 之前，但写操作可能会被乱序发布。

##### 表 10-1：内存类型访问顺序

| A1       | A2       | 访问顺序       |
|----------|----------|----------------|
| 普通内存 | 普通内存 | 不强制顺序     |
| 设备内存 | 设备内存 | 不强制顺序     |
| 强序内存 | 强序内存 | 按程序顺序发布 |

#### 10.1.1 强序内存与设备内存

强序内存和设备内存具有相同的内存排序模型。其访问规则如下：

- 访问的**数量和大小**将被保留。访问是**原子的**，不会在中途被中断。
- 读写访问可能会对系统产生副作用。访问不会被缓存，且**永远不会进行推测性访问**。
- 访问**不能未对齐**。
- 对设备内存的访问顺序保证与访问该内存的指令的程序顺序一致。此保证仅适用于同一外设或内存块内的访问。这个内存块的大小是实现定义的，但最小为 1KB。
- 在 ARMv7 架构中，核心可以在强序或设备内存访问周围对普通内存访问进行重新排序。

**强序内存**与**设备内存**的唯一区别在于：

- 对强序内存的写入必须**等到写入到达外设或内存组件**时才能完成。
- 对设备内存的写入则**可以在到达外设或内存组件之前完成**。

系统外设通常会映射为设备内存类型。

设备内存类型的区域可以使用**可共享属性**进行描述。

在一些 ARMv6 处理器上，设备访问的可共享属性用于确定将使用哪个内存接口。对标记为**设备-不可共享**的区域的内存访问将使用专用接口（称为私有外设端口）。该机制在 ARMv7 处理器上不再使用。

##### 注意

这些内存排序规则仅对**显式内存访问**（即由加载和存储指令引发的访问）提供保证。架构不对指令获取或翻译表遍历与此类显式内存访问的排序提供类似的保证。

#### 10.1.2 普通内存 (Normal Memory)

**普通内存**用于描述内存系统的大部分区域。所有的 ROM 和 RAM 设备都被视为普通内存。

普通内存的属性如下：

- **核心可以重复读取**和某些写入操作。
- **核心可以预取或推测性访问**额外的内存位置，且不会产生副作用（如果 MMU 访问权限设置允许）。但是，核心不会执行推测性写入。
- **可以执行未对齐的访问**。
- **多个访问可以由核心硬件合并**为较少数的、更大规模的访问。例如，多个字节写入可以合并为单个双字写入。

普通内存区域必须有**可缓存性属性**，详见第 8 章关于支持的缓存策略的描述。ARM 架构支持针对普通内存的两级缓存的可缓存性属性，分别为**内层缓存**和**外层缓存**。这些缓存级别与实际物理缓存级别的映射是实现定义的。**内层**指的是最内层的缓存，始终包括核心的 L1 缓存。某些实现可能没有外层缓存，或者将外层可缓存性属性应用于 L2 或 L3 缓存。例如，在包含 Cortex-A9 处理器和 L2C-310 二级缓存控制器的系统中，L2C-310 被视为外层缓存。而 Cortex-A8 的 L2 缓存可以配置为使用内层或外层缓存策略。

#### 可共享性 (Shareability)

普通内存还必须指定为**可共享 (Shareable)** 或 **不可共享 (Non-Shareable)**。具有不可共享属性的普通内存区域仅供该核心使用。核心不需要使访问该内存位置与其他核心保持一致性。如果其他核心共享该内存，则任何一致性问题必须由软件处理。例如，可以通过让各个核心执行缓存维护和屏障操作来实现这一点。

**外层可共享属性 (Outer Shareable)** 使得可以定义含有多个一致性控制级别的系统。例如，一个内层可共享域可以由一个 Cortex-A15 集群和 Cortex-A7 集群组成。在一个集群内，核心的数据缓存对所有具有内层可共享属性的数据访问保持一致性。而外层可共享域可能由这个集群和一个具有多个核心的图形处理器组成。一个外层可共享域可以包含多个内层可共享域，但一个内层可共享域只能属于一个外层可共享域。

设置了**可共享属性**的内存区域是可以由系统中其他代理访问的区域。同一可共享域内的其他处理器对该内存区域的访问是一致的。也就是说，您不需要处理数据或缓存的一致性问题。如果没有可共享属性，在多个核心之间共享内存的区域中没有维护缓存一致性，则您需要显式地管理一致性。

ARMv7 架构允许您将可共享内存指定为**内层可共享**或**外层可共享**（后者意味着该位置同时为内层和外层可共享）。

### 10.2 内存屏障

**内存屏障**是一条指令，用于要求核心在执行程序中的内存屏障指令前后，对内存操作施加顺序约束。在其他架构中，这类指令也可以称为**内存栅栏**（Memory Fences）。

内存屏障有时也用于指代**编译器机制**，防止编译器在优化时将数据访问指令跨越屏障重新调度。例如，在 GCC 中，您可以使用内联汇编中的内存屏障 `memory clobber`，来指示该指令修改了内存，因此优化器不能跨越屏障重新排序内存访问。其语法如下：

```c
asm volatile("" ::: "memory");
```

ARM RVCT 包含类似的内置函数，称为 `__schedule_barrier()`。

但是这里我们主要讨论**硬件内存屏障**，这些屏障是通过专门的 ARM 汇编语言指令提供的。正如我们所看到的，诸如缓存、写缓冲区和乱序执行等核心优化可能导致内存操作的顺序与执行代码中指定的顺序不同。通常，这种重新排序对开发者是不可见的。应用程序开发者通常不必考虑内存屏障。然而，在某些情况下（例如设备驱动程序或涉及多个数据观察者需要同步的场合），您可能需要关注这些顺序问题。

ARM 架构定义了一些内存屏障指令，它们使您能够强制核心等待内存访问完成。这些指令可以在 ARM 和 Thumb 代码中使用，且在用户模式和特权模式下都可用。在较早的架构版本中，这些操作仅通过 ARM 代码中的 CP15 操作完成。虽然这些用法已被弃用，但仍保留以保证兼容性。

我们先从这些指令在单核系统中的实际效果开始讨论。此描述是 ARM 架构参考手册中内容的简化版本，旨在介绍这些指令的用法。术语**显式访问**是指由于程序中的加载或存储指令而导致的数据访问。它不包括指令获取。

#### 数据同步屏障 (DSB)

`DSB` 指令强制核心等待所有挂起的显式数据访问完成，然后才能执行后续的指令阶段。它对指令的预取没有影响。

#### 数据内存屏障 (DMB)

`DMB` 指令确保程序顺序中位于该屏障之前的所有内存访问在系统中被观察到，然后再观察到程序顺序中位于该屏障之后的显式内存访问。它不影响核心上执行的其他指令的顺序，也不影响指令获取的顺序。

#### 指令同步屏障 (ISB)

`ISB` 刷新核心中的流水线和预取缓冲区，使屏障之后的所有指令在该指令完成后从缓存或内存中获取。这确保了上下文改变操作的效果（例如 CP15 或 ASID 更改，或 TLB 或分支预测器操作）在 ISB 指令执行后，对后续获取的指令可见。该指令本身不会引起数据和指令缓存之间的同步，但它是此类操作的一部分。

#### DMB 和 DSB 指令的选项

DMB 和 DSB 指令可以指定几个选项，用来提供访问类型和适用的可共享域，如下所示：

- **SY**：这是默认选项，表示屏障适用于整个系统，包括所有核心和外设。
- **ST**：屏障仅等待存储操作完成。
- **ISH**：屏障仅适用于内层可共享域（Inner Shareable）。
- **ISHST**：屏障仅等待存储操作完成，且只适用于内层可共享域。
- **NSH**：屏障仅适用于统一点 (PoU)（参见第 8-19 页的“一致性与统一点”）。
- **NSHST**：屏障仅等待存储操作完成，且只到达统一点。
- **OSH**：屏障仅适用于外层可共享域（Outer Shareable）。
- **OSHST**：屏障仅等待存储操作完成，且只适用于外层可共享域。

要理解这些选项的含义，您需要在多核系统中使用更通用的 DMB 和 DSB 操作定义。以下文本中的“处理器”或“代理”不仅仅指核心，它还可能指 DSP、DMA 控制器、硬件加速器或任何其他访问共享内存的模块。

#### DMB 和 DSB 指令在多核系统中的作用

`DMB` 指令的作用是在共享域中强制内存访问顺序。共享域中的所有处理器都保证在观察到 DMB 指令之前的所有显式内存访问之前，不会观察到 DMB 指令之后的显式内存访问。

`DSB` 指令具有与 `DMB` 相同的效果，但除此之外，它还将内存访问与整个指令流同步。这意味着，当发出 `DSB` 时，执行将暂停，直到所有挂起的显式内存访问完成。当所有挂起的读取完成并且写缓冲区被清空后，执行将恢复正常。

#### 内存屏障示例：Cortex-A9 集群

为了更好地理解这些屏障的效果，考虑一个四核 Cortex-A9 集群的情况。该集群构成了一个内层可共享域。当集群中的一个核心执行 `DMB` 指令时，该核心将确保程序顺序中屏障之前的所有数据内存访问完成，然后再执行屏障之后的显式内存访问。这样可以保证集群中的所有核心在观察这些访问时，顺序与执行它们的核心相同。如果使用 `DMB ISH` 变体，则无法保证外部观察者（如 DMA 控制器或 DSP）也能看到相同的顺序。

#### 10.2.1 内存屏障使用示例

考虑一个有两个核心 A 和 B 的情况，两个地址 `Addr1` 和 `Addr2` 位于普通内存中并存储在核心寄存器中。每个核心执行两条指令，如 **示例 10-1** 所示：

#### 示例 10-1：展示内存排序问题的代码示例

```assembly
Core A:              Core B:  
STR R0, [Addr1]      LDR R1, [Addr2]  
STR R2, [Addr2]      LDR R3, [Addr1]
```

在这个例子中，没有任何排序要求，您无法对任何事务的执行顺序做出声明。`Addr1` 和 `Addr2` 是独立的地址，两个核心不需要按程序中编写的顺序执行加载和存储操作，也不需要关心其他核心的活动。

因此，这段代码可能有四种合法的结果，最终在核心 A 的寄存器 R1 和核心 B 的寄存器 R3 中会得到四组不同的内存值：

1. A 获取旧值，B 获取旧值。
2. A 获取旧值，B 获取新值。
3. A 获取新值，B 获取旧值。
4. A 获取新值，B 获取新值。

如果引入第三个核心 C，您还需要注意，它没有要求与其他核心以相同的顺序观察到存储操作。完全可能 A 和 B 在 `Addr1` 和 `Addr2` 中看到旧值，但 C 看到的是新值。

接下来考虑以下情况：核心 B 等待核心 A 设置的一个标志，然后读取内存，例如，您正在从核心 A 向核心 B 传递消息。代码可能类似于 **示例 10-2**：

#### 示例 10-2：存在排序问题的邮筒模式

```assembly
Core A:              Core B:  
STR R0, [Msg]        Poll_loop:  
STR R1, [Flag]       LDR R1, [Flag]  
                     CMP R1, #0  
                     BEQ Poll_loop  
                     LDR R0, [Msg]  
```

在这个例子中，程序的行为可能不会如预期那样。没有理由禁止核心 B 在读取 `[Flag]` 之前推测性地从 `[Msg]` 读取数据。这是正常的、弱排序的内存，核心并不知道这两者之间可能存在依赖性。您必须通过插入内存屏障来显式地强制这种依赖性。在这个例子中，实际上需要两个内存屏障：核心 A 需要在两个存储操作之间插入 `DMB`，以确保它们按照您最初指定的顺序发生。核心 B 需要在 `LDR R0, [Msg]` 之前插入 `DMB`，以确保在读取消息之前，标志已被设置。

---

### 10.2.2 使用屏障避免死锁

如果不使用屏障指令，还可能导致死锁的另一个情况是，核心写入一个地址，然后轮询某个外设应用的确认值。

**示例 10-3** 展示了可能导致问题的代码：

#### 示例 10-3：死锁示例

```assembly
STR R0, [Addr]  @ 向外设寄存器写入命令  
DSB  
Poll_loop:  
LDR R1, [Flag]  
CMP R1, #0      @ 等待确认/状态标志被设置  
BEQ Poll_loop  
```

如果没有多处理扩展，ARMv7 架构并不严格要求对 `[Addr]` 的存储操作必须完成（它可能会停留在写缓冲区中，而内存系统正忙于读取标志），因此两个核心可能会陷入死锁，彼此等待。为核心的 `STR` 操作插入 `DSB` 强制其存储操作在读取 `Flag` 之前被观察到。

实现多处理扩展的核心要求在有限的时间内完成访问（即它们的写缓冲区必须清空），因此不需要使用屏障指令。

---

### 10.2.3 WFE 和 WFI 与屏障的交互

`WFE`（等待事件）和 `WFI`（等待中断）指令使核心停止执行并进入低功耗状态。为了确保在执行 `WFI` 或 `WFE` 之前的所有内存访问已完成（并对其他核心可见），您必须插入 `DSB` 指令。

在 MP（多处理）系统中还需考虑 `WFE` 和 `SEV`（发送事件）的使用。您可以使用这些指令减少锁获取循环（自旋锁）相关的功耗。尝试获取互斥锁的核心可能会发现另一个核心已经持有锁。此时，您可以使用 `WFE` 指令暂停执行并进入低功耗状态，而不是让核心不断轮询锁。

当中断或其他异步异常被识别时，或者另一个核心通过 `SEV` 指令发送事件时，核心将被唤醒。持有锁的核心会在释放锁后使用 `SEV` 指令唤醒处于 `WFE` 状态的其他核心。对于内存屏障指令来说，事件信号**并不被视为显式内存访问**。因此，我们必须确保在执行 `SEV` 指令之前，释放锁的更新操作对其他处理器是可见的。这就需要使用 `DSB` 指令。

`DMB` 是不够的，因为它仅影响内存访问的顺序，而不会将其同步到特定的指令。相比之下，`DSB` 会阻止 `SEV` 的执行，直到所有之前的内存访问已被其他核心观察到。

### 10.3 缓存一致性问题

缓存对应用程序开发者来说大多是不可见的。然而，当系统中的其他地方改变内存位置，或应用程序代码进行的内存更新需要对系统的其他部分可见时，缓存的一致性问题就会显现出来。

例如，包含一个外部 DMA 设备和一个处理器核心的系统，提供了一个可能出现问题的简单示例。在以下两种情况下，缓存一致性可能会崩溃：

- 如果 DMA 从主内存读取数据，而较新的数据仍保留在核心缓存中，DMA 将读取到旧的数据。
- 同样地，如果 DMA 向主内存写入数据，而核心缓存中存在过时数据，核心可能会继续使用旧数据。

因此，在 DMA 启动之前，核心数据缓存中的脏数据必须显式地清除。同样地，如果 DMA 正在复制数据并且需要核心读取这些数据，就必须确保核心数据缓存中不含有过时的数据。DMA 写入内存不会自动更新缓存，这可能需要核心清除或失效缓存中的受影响内存区域，然后再启动 DMA。由于所有 ARMv7-A 处理器都可能进行推测性内存访问，因此在使用 DMA 之后也需要执行失效操作。

#### 10.3.1 复制代码时的问题

引导代码、内核代码或 JIT 编译器可能会将程序从一个位置复制到另一个位置，或者修改内存中的代码。硬件没有机制来保持指令缓存与数据缓存之间的同步一致性。因此，必须通过失效受影响区域来使指令缓存中的过时代码失效，并确保写入的代码实际已经到达主内存。如果核心接下来要跳转到修改后的代码，则需要特定的代码序列和指令屏障。

#### 10.3.2 编译器的重新排序优化

需要理解的是，内存屏障指令仅适用于**硬件**重新排序的内存访问。插入硬件内存屏障指令可能不会对编译器的操作重新排序产生直接影响。在 C 语言中，`volatile` 类型限定符告诉编译器，该变量可以被当前执行代码以外的其他内容更改。这常用于 C 语言访问内存映射的 I/O，使得这样的设备可以通过指向 `volatile` 变量的指针安全地访问。

然而，C 标准并未提供关于在多核系统中使用 `volatile` 的规则。因此，尽管可以确保 `volatile` 加载和存储相对于彼此按程序指定的顺序发生，但并没有关于相对于非 `volatile` 加载或存储的重新排序保证。这意味着 `volatile` 并不能作为实现互斥锁的捷径。
