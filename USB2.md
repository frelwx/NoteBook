
# 第一章  
## 引言  

### 1.1 动机  

设计通用串行总线（USB）的初始动机源自以下三个相互关联的考虑：  

#### 1. PC与电话的连接  
众所周知，计算技术与通信技术的融合将成为下一代生产力应用的基础。从一个位置或环境向另一个位置或环境传输机器导向和人类导向的数据类型，依赖于普遍且低成本的连接性。然而，计算和通信行业一直各自独立发展。USB提供了一种普遍适用的连接方式，可用于广泛的PC与电话间的互联。  

#### 2. 易用性  
PC在重新配置方面的灵活性不足已被公认为其进一步推广的致命弱点。用户友好的图形界面与新一代总线架构的硬件和软件机制相结合，使计算机变得更易于接受且更易于重新配置。然而，从最终用户的角度来看，PC的输入/输出接口（如串行/并行端口、键盘/鼠标/游戏杆接口等），并不具有即插即用的特性。  

#### 3. 接口扩展  
外部外设的增加始终受到接口数量的限制。缺乏一个双向、低成本、低到中速的外设总线，限制了外设的创新和发展，例如电话/传真/调制解调器适配器、电话答录机、扫描仪、PDA、键盘、鼠标等。现有的接口通常是为一种或两种特定功能优化的。随着每种新功能或能力的引入，都会定义一个新的接口来满足需求。  

USB 2.0的更近期动机源于PC性能的不断提升，能够处理海量数据的能力。同时，PC外设也增加了更多性能和功能。用户应用（如数字成像）需要PC与这些日益复杂的外设之间提供高性能的连接。USB 2.0通过在原有的12 Mb/s和1.5 Mb/s传输速率基础上，增加了480 Mb/s的第三种传输速率，满足了这一需求。  

USB 2.0是USB的自然演进，它在保留原有动机的同时，实现了带宽的提升，并保持与现有外设的完全兼容性。因此，USB继续成为PC架构中连接的解决方案。它是一种快速、双向、同步、低成本、动态可连接的串行接口，符合当前和未来PC平台的需求。  

---

### 1.2 规范目标  

本文档定义了一个行业标准的USB规范，描述了总线的属性、协议定义、传输类型、总线管理以及设计和构建符合该标准的系统和外设所需的编程接口。  

目标是使不同供应商的设备能够在开放架构中实现互操作性。该规范旨在作为PC架构的增强，覆盖便携式设备、商业台式机和家庭环境。规范的设计意在为系统OEM和外设开发者提供足够的产品多样性和市场差异化空间，同时避免承载过时接口的负担，或失去兼容性。  

---

### 1.3 文档范围  

该规范主要面向外设开发者和系统OEM，同时也为平台操作系统/BIOS/设备驱动程序、适配器硬件/软件厂商（IHVs/ISVs）以及平台/适配器控制器供应商提供了有价值的信息。该规范可用于开发新产品及相关软件。  

---

### 1.4 USB产品合规性  

USB 2.0规范的采纳者已签署USB 2.0采纳者协议，该协议使其能够从推广者和其他采纳者处获得与USB 2.0规范兼容的产品中包含的某些知识产权的互惠免版税许可。  

采纳者可通过USB实施者论坛定义的测试程序证明其符合规范。符合规范的产品将被授予在符合Logo许可的情况下使用USB实施者论坛标志的某些权利。  

---

### 1.5 文档结构  

第1至第5章为所有读者提供概述，第6至第11章包含定义USB的详细技术信息。  

- **外设开发者**应特别关注第5至第11章。  
- **USB主机控制器开发者**应特别关注第5至第8章、第10章和第11章。  
- **USB设备驱动程序开发者**应特别关注第5章、第9章和第10章。  

本规范由《通用串行总线设备类别规范》补充和引用。设备类别规范涵盖了各种设备，请联系USB实施者论坛获取详细信息。  

同时，建议读者联系操作系统供应商，获取与USB相关的操作系统绑定信息。

# 第三章  
## 背景  

本章简要介绍了通用串行总线（USB）的背景，包括设计目标、总线特性以及现有技术。  

---

### 3.1 通用串行总线的目标  

USB被定义为PC架构的行业标准扩展，重点支持PC外设以实现消费者和商业应用。USB架构的定义依据了以下标准：  

- **易用性**：便于扩展PC外设。  
- **低成本**：支持高达480 Mb/s的传输速率。  
- **实时数据支持**：全面支持语音、音频和视频的实时数据传输。  
- **协议灵活性**：支持混合模式的同步（Isochronous）数据传输和异步消息传递。  
- **设备技术集成**：适应通用设备技术。  
- **兼容性**：适应各种PC配置和外形设计。  
- **标准化接口**：提供一个标准接口，便于快速应用于产品中。  
- **新设备类别**：支持开发增强PC能力的新设备类别。  
- **向后兼容性**：USB 2.0完全兼容基于早期版本规范的设备。  

---

### 3.2 应用空间分类  

图3-1描述了USB可支持的数据流量负载范围的分类。从图中可以看出，480 Mb/s的总线涵盖了高速、全速和低速数据范围。  

- **高速（High-Speed）**和**全速（Full-Speed）**数据通常用于同步（Isochronous）传输。  
- **低速（Low-Speed）**数据通常来自交互式设备。  
- **USB虽然主要是为PC设计的总线**，但也可以轻松应用于其他以主机为中心的计算设备。  
- **USB的软件架构**允许对未来扩展提供支持，例如支持多个USB主机控制器。  

#### 性能与应用分类  

| **性能类别** | **应用场景**             | **属性**                                                                 |
|--------------|--------------------------|--------------------------------------------------------------------------|
| **低速**     | 交互式设备（10–100 kb/s）| 键盘、鼠标、触控笔、游戏外设、虚拟现实外设<br>最低成本，易用性，动态连接/断开，多设备支持 |
| **全速**     | 电话、音频、压缩视频（500 kb/s–10 Mb/s）| 电话、宽带、音频、麦克风<br>低成本，易用性，动态连接/断开，多设备支持，保证带宽和延迟 |
| **高速**     | 视频、存储（25–400 Mb/s）| 视频、存储、成像、宽带<br>低成本，易用性，动态连接/断开，多设备支持，保证高带宽和低延迟 |  

**图3-1 应用空间分类**  


### 3.3 特性列表  

USB规范提供了一组属性，能够实现不同的价格/性能集成点，并支持在系统和组件级别实现功能差异化。这些特性按以下优势分类：  

---

#### **终端用户易用性**  
- 单一的布线和连接器模型。  
- 电气细节对终端用户透明（例如总线终端）。  
- 外设具备自识别功能，支持功能自动映射到驱动程序和配置。  
- 外设可以动态连接并重新配置。  

---

#### **广泛的工作负载和应用支持**  
- 适用于从几kb/s到数百Mb/s的设备带宽。  
- 在同一组线路上支持同步（Isochronous）和异步（Asynchronous）传输类型。  
- 支持多个设备的并发操作（多连接）。  
- 支持最多127个物理设备。  
- 支持主机与设备之间的多数据流和消息流传输。  
- 支持复合设备（即由多个功能组成的外设）。  
- 较低的协议开销，带来高总线利用率。  

---

#### **同步带宽**  
- 提供适用于电话、音频、视频等实时应用的**保障带宽**和**低延迟**。  

---

#### **灵活性**  
- 支持多种数据包大小，为设备提供多样化的缓存选项。  
- 通过适应数据包缓存大小和延迟，支持多种设备数据速率。  
- 协议内置了缓冲区处理的**流量控制**机制。  

---

#### **稳健性**  
- 协议内置错误处理和故障恢复机制。  
- 动态插入和移除设备时，可在用户感知的实时环境中识别。  
- 支持故障设备的识别。  

---

#### **与PC行业的协同**  
- 协议易于实现和集成。  
- 与PC的即插即用架构一致。  
- 利用现有的操作系统接口。  

---

#### **低成本实现**  
- 提供1.5 Mb/s的低成本子通道。  
- 优化了外设和主机硬件的集成。  
- 适合开发低成本外设。  
- 使用低成本电缆和连接器。  
- 利用通用技术实现。  

---

#### **升级路径**  
- USB架构可升级，以支持在同一系统中使用多个USB主机控制器。

# 第四章  
## 架构概述  

本章概述了通用串行总线（USB）架构及其关键概念。USB是一种电缆总线，用于支持主机计算机与多种可同时访问的外设之间的数据交换。连接的外设通过主机调度的基于令牌的协议共享USB带宽。USB总线允许在主机和其他外设运行期间，动态地连接、配置、使用和断开外设。  

后续章节将更详细地描述USB的各个组成部分。  

---

### 4.1 USB系统描述  

USB系统由以下三个定义领域组成：  
- **USB互连**  
- **USB设备**  
- **USB主机**  

**USB互连**是USB设备与主机连接并通信的方式，包括以下内容：  

1. **总线拓扑**：描述USB设备与主机之间的连接模型。  
2. **层间关系**：基于能力堆栈的模型，描述USB系统中每层完成的任务。  
3. **数据流模型**：描述数据在USB系统中生产者与消费者之间的传输方式。  
4. **USB调度**：USB提供了一个共享的互连结构。通过调度访问互连，支持同步（Isochronous）数据传输并消除仲裁开销。  

USB设备和USB主机将在后续章节中详细描述。  

---

### 4.1.1 总线拓扑  

USB通过物理连接将USB设备与USB主机连接在一起。USB的物理互连采用**分级星型拓扑**：每个星型的中心是一个**集线器（Hub）**。每段电缆是主机与集线器或功能设备之间，或集线器与另一个集线器或功能设备之间的点对点连接。图4-1展示了USB的拓扑结构。  

#### **层级限制**  
由于集线器和电缆的传播时间限制，允许的最大层级数是**七层**（包括根层）。需要注意的是，在七层拓扑中，从主机到任意设备的通信路径最多可以包含五个非根集线器。如果连接路径中包含复合设备（Compound Device，见图4-1），其占用两个层级，因此不能连接到第七层。只有功能设备可以启用在第七层。  

---

#### 4.1.1.1 USB主机  

在任何USB系统中只有一个主机。主机计算机系统的USB接口称为**主机控制器（Host Controller）**，它可以通过硬件、固件或软件的组合实现。主机系统内集成了一个**根集线器（Root Hub）**，提供一个或多个连接点。  

关于主机的更多信息，请参阅**4.9节**和**第十章**。  

---

#### 4.1.1.2 USB设备  

USB设备包含以下两类：  
1. **集线器（Hubs）**：为USB提供额外的连接点。  
2. **功能设备（Functions）**：为系统提供某些功能，例如ISDN连接、数字操纵杆或扬声器。  

USB设备通过以下标准USB接口进行交互：  
- 理解USB协议的能力。  
- 对标准USB操作（如配置和重置）的响应能力。  
- 提供标准功能描述信息。  

关于USB设备的更多信息，请参阅**4.8节**和**第九章**。

### 4.2 物理接口  

USB的物理接口在电气（第7章）和机械（第6章）规范中进行了定义。  

---

#### **4.2.1 电气特性**  

USB通过四线电缆传输信号和电源，如图4-2所示。每个点对点段上的信号通过两根线传输。  

USB支持以下三种数据速率：  
- **高速（High-Speed）**：480 Mb/s。  
- **全速（Full-Speed）**：12 Mb/s。  
- **低速（Low-Speed）**：1.5 Mb/s，功能有限。  

USB 2.0主机控制器和集线器具有以下能力：  
- 在主机控制器和集线器之间以**高速**传输全速和低速数据。  
- 从集线器到设备的数据传输以全速或低速进行。  

这种能力可以最大程度地减少全速和低速设备对高速设备带宽的影响。  

**低速模式**主要用于支持少量低带宽设备（例如鼠标），因为更广泛的使用会降低总线的利用率。  

USB的时钟信号与差分数据一起传输，并采用**NRZI（非归零反向）编码**和位填充技术，以确保足够的信号转换。每个数据包前面都有一个**SYNC字段**，允许接收方同步其位恢复时钟。  

USB电缆还包括**VBUS**和**GND**线，用于为设备供电：  
- **VBUS电压**通常为 +5V（由源提供）。  
- USB允许不同长度的电缆段（最长可达数米），通过选择适当的导体规格以匹配规定的电阻压降、设备功率预算和电缆柔韧性等属性。  

为了确保正确的输入电压水平和适当的终端阻抗，电缆两端使用了**偏置端接**。端接还可以检测每个端口的设备连接与断开，并区分高速/全速设备与低速设备。  

---

#### **4.2.2 机械特性**  

USB电缆和连接器的机械规范在**第6章**中定义：  
- 所有设备都具有**上游连接**。  
- 上游和下游连接器在机械结构上不可互换，从而避免集线器中的非法回环连接。  
- USB电缆具有四根导线：  
  - 一对标准规格的**信号双绞线**。  
  - 一对电源线，允许使用不同规格。  
- 连接器为四针设计，采用屏蔽外壳，具有规定的抗损性，并且支持便捷的插拔操作。  

---

### 4.3 电源  

USB规范涵盖了以下两个方面的电源管理：  

#### **4.3.1 电源分配**  
每个USB段通过电缆提供有限的电源：  
- 主机为直接连接的USB设备提供电源。  
- 任何USB设备也可以拥有自己的电源供应。  

USB设备根据电源来源分为两类：  
- **总线供电设备（Bus-Powered Devices）**：完全依赖于电缆提供的电源。  
- **自供电设备（Self-Powered Devices）**：有备用电源来源。  

集线器也为其连接的USB设备提供电源。架构允许在一定拓扑限制下使用总线供电的集线器，这将在**第11章**中讨论。  

#### **4.3.2 电源管理**  
USB主机可以拥有独立于USB的电源管理系统：  
- **USB系统软件**与主机的电源管理系统交互，处理系统电源事件（如挂起或恢复）。  
- USB设备通常实现额外的电源管理功能，使其能够被系统软件管理。  

USB的电源分配和电源管理功能，使其适合设计用于对电源敏感的系统，如电池供电的笔记本电脑。  

---

### 4.4 总线协议  

USB是一个**轮询总线**，所有数据传输均由**主机控制器**发起。  

#### **4.4.1 总线事务**  
大多数总线事务涉及最多三个数据包的传输：  
1. 主机控制器在计划的时间点发送一个**令牌包（Token Packet）**，描述事务的类型、方向、USB设备地址和端点编号。  
2. 被寻址的USB设备通过解码地址字段选择自身。  
3. 在事务中，数据从主机传输到设备或从设备传输到主机。数据传输方向由令牌包指定。  

事务的源头发送一个数据包，或者指示没有数据要传输。目标设备通常通过**握手包（Handshake Packet）**响应，指示传输是否成功。  

#### **4.4.2 主机与集线器的事务**  
某些主机控制器与集线器之间的总线事务涉及**四个数据包**，用于管理主机与全速/低速设备之间的数据传输。  

#### **4.4.3 数据流模型**  
USB在主机和设备端点之间的数据传输模型称为**管道（Pipe）**：  
- 管道分为两种类型：**数据流管道（Stream Pipes）**和**消息管道（Message Pipes）**。  
  - **数据流管道**：没有USB定义的结构。  
  - **消息管道**：具有USB定义的结构。  

管道与以下特性相关联：  
- 数据带宽  
- 传输服务类型  
- 端点特性（如方向性和缓冲区大小）  

大多数管道在设备配置时生成。一个特殊的**默认控制管道（Default Control Pipe）**始终在设备通电时存在，用于访问设备的配置、状态和控制信息。  

#### **4.4.4 流量控制**  
事务调度为某些数据流管道提供流量控制：  
- 在硬件层面，通过**NAK握手**机制防止缓冲区欠载或过载，从而节流数据速率。  
- 被NAK的事务会在总线空闲时重新尝试。  

这种流量控制机制支持灵活的调度，允许同时为多种类型的数据流管道提供服务。不同的数据流管道可以以不同的间隔和数据包大小进行服务，满足异构数据流混合处理的需求。

### 4.5 稳健性  

USB具备多项增强稳健性的特性，具体包括：  
- 通过差分驱动器、接收器以及屏蔽技术确保信号完整性。  
- 控制和数据字段使用**循环冗余校验（CRC）**保护。  
- 自动检测设备的连接与断开，并在系统层面配置资源。  
- 协议自恢复能力，通过超时机制处理丢失或损坏的数据包。  
- 对数据流的流量控制，确保同步性并管理硬件缓冲区。  
- 数据和控制管道结构，确保不同功能之间的相互独立，避免不良交互。  

---

#### **4.5.1 错误检测**  

USB的核心位误码率接近于背板总线（非常低），任何信号故障通常是短暂性的。为防止这些短暂故障，每个数据包都包含错误保护字段。  

对于需要数据完整性的设备（如无损数据设备），协议可以通过硬件或软件触发错误恢复程序。  

USB协议对每个数据包的控制字段和数据字段分别进行**CRC校验**：  
- CRC校验可以100%覆盖单比特和双比特错误。  
- 如果CRC校验失败，则认为数据包已损坏。  

---

#### **4.5.2 错误处理**  

USB协议允许通过硬件或软件进行错误处理：  
- **硬件错误处理**：包括报告和重试失败的传输。  
  - 主机控制器会对出错的传输尝试**最多三次**，如果仍然失败，会通知客户端软件进行处理。  
- **软件错误处理**：客户端软件可以根据实现要求决定如何恢复错误。  

---

### 4.6 系统配置  

USB支持设备随时连接或断开，因此系统软件必须能够适应物理总线拓扑的动态变化。  

---

#### **4.6.1 USB设备的连接**  

所有USB设备通过专用USB设备（称为**集线器**）上的端口连接到USB总线：  
- 集线器具有**状态位**，用于报告其端口上的USB设备连接或移除情况。  
- 主机通过查询集线器获取这些状态位信息：  
  - 当检测到设备连接时，主机启用该端口，并通过设备的默认控制管道（Control Pipe）以默认地址与其通信。  
  - 主机为新连接的USB设备分配一个唯一的USB地址，并确定该设备是**集线器**还是**功能设备**。  
  - 主机通过分配的USB地址和端点0建立控制管道的主机端。  

如果连接的USB设备是一个集线器，并且其端口上还连接了其他USB设备，则主机会对每个连接的设备重复上述过程。  

如果连接的设备是一个功能设备，则主机会通过适当的软件处理设备连接通知。  

---

#### **4.6.2 USB设备的移除**  

当USB设备从集线器的端口移除时：  
- 集线器会禁用该端口，并向主机提供设备移除的指示信息。  
- 主机通过相应的USB系统软件处理设备移除：  
  - 如果移除的是集线器，则USB系统软件需要同时处理集线器及其下连接的所有USB设备的移除。  

---

#### **4.6.3 总线枚举**  

**总线枚举**是指对连接到总线的设备进行识别并分配唯一地址的过程：  
- 由于USB设备可以随时连接或断开，**总线枚举是USB系统软件的持续过程**。  
- 除了设备识别，枚举还包括检测设备移除并进行相应处理。  

---

### 4.7 数据流类型  

USB支持主机与设备之间的功能数据和控制信息交换，主要通过**单向或双向管道（Pipes）**实现：  
- USB数据传输在主机软件与设备上特定端点之间进行。  
- 主机软件与设备端点之间的这种关联被称为**管道**。  
- 通常，一个管道中的数据流与其他管道的数据流相互独立。  
- 单个USB设备可能支持多个管道，例如：  
  - 一个端点支持向设备传输数据的管道。  
  - 另一个端点支持从设备传输数据的管道。  

USB架构定义了以下**四种基本的数据传输类型**：  

1. **控制传输（Control Transfers）**  
   - 用于设备连接时的配置，也可用于其他设备特定的目的（例如控制设备上的其他管道）。  

2. **批量数据传输（Bulk Data Transfers）**  
   - 用于传输相对较大且突发性的数据量，对传输的时间约束较宽松。  

3. **中断数据传输（Interrupt Data Transfers）**  
   - 用于及时且可靠地传输数据，例如字符或坐标，这些数据需要具有人类可感知的响应特性（如回显或反馈）。  

4. **同步数据传输（Isochronous Data Transfers）**  
   - 占用预先协商的USB带宽，并有预先协商的传输延迟（也称为实时流传输）。  

---

#### **管道与数据流类型的关系**  

- 一个管道在设备任何特定配置下仅支持上述一种数据传输类型。  
- USB的数据流模型将在**第5章**中进行更详细的描述。
### 4.7 数据传输类型  

USB支持主机与设备之间通过**管道（Pipes）**进行的功能数据和控制信息交换。以下是四种主要的数据传输类型：  

---

#### **4.7.1 控制传输（Control Transfers）**  
- 控制数据由USB系统软件在设备首次连接时用于设备配置。  
- 其他驱动程序软件也可以根据具体实现选择使用控制传输。  
- 数据传输是无损的，确保数据完整性。  

---

#### **4.7.2 批量传输（Bulk Transfers）**  
- 批量数据通常是较大数量的数据，例如打印机或扫描仪传输的数据。  
- **批量数据是顺序传输**，硬件层面通过错误检测和有限次数的重试确保数据可靠性。  
- 批量数据传输的带宽根据总线活动动态变化。  

---

#### **4.7.3 中断传输（Interrupt Transfers）**  
- **中断数据**是一种低延迟的数据传输，用于与设备进行及时交互。  
- 数据可以在任何时间由设备准备好进行传输，并由USB以不低于设备指定的速率传输。  
- 中断数据通常是事件通知、字符或坐标（例如鼠标的坐标）。  
- 尽管不需要明确的时间同步，但交互数据可能需要USB支持一定的响应时间限制。  

---

#### **4.7.4 同步传输（Isochronous Transfers）**  
- 同步数据是**连续且实时**的，包括创建、传输和消费三个阶段。  
- 同步数据的时间信息通过数据的稳定接收和传输速率隐含传递。  
- **传输速率必须保持不变**以维持数据的时间特性，且可能对传输延迟敏感。  
- 同步管道所需的带宽通常由相关功能的采样特性决定，延迟则与端点的缓冲能力有关。  

**示例**：语音数据是典型的同步数据。如果数据流的传输速率无法维持，可能会由于缓冲区不足或过载导致数据丢失（如语音卡顿）。  

- 同步数据的及时传输是通过牺牲短暂的数据丢失来实现的，即硬件不会通过重试等机制纠正电气传输错误。  
- USB硬件会为同步数据流分配专用带宽，确保传输速率，并尽可能减少延迟。  

---

#### **4.7.5 USB带宽分配**  
- USB带宽是按照管道分配的。  
- 在建立管道时，USB为某些管道分配带宽。需要更多带宽的设备通常会提供更大的缓冲区。  
- USB的设计目标是将缓冲区引起的硬件延迟限制在**几毫秒以内**。  
- USB的带宽容量可以在多个数据流之间分配，支持不同设备的连接，并允许同时处理不同速率的数据流。  
- USB规范定义了每种传输类型访问总线的规则。  

---

### 4.8 USB设备  

USB设备根据功能被划分为不同的**设备类别**，例如集线器（Hub）、人机接口设备（Human Interface Device）、打印机、成像设备或大容量存储设备。  

- 集线器设备类别指的是一种特殊的USB设备，它提供额外的USB连接点（详见第11章）。  
- USB设备必须具备**自识别**和**通用配置**的信息，并始终表现出与定义的USB设备状态一致的行为。  

---

#### **4.8.1 设备特性**  
- 所有USB设备都通过在连接和枚举时分配的**USB地址**进行访问。  
- 每个USB设备支持一个或多个管道，主机可以通过这些管道与设备通信。  
- 所有USB设备必须支持一个特殊指定的管道（端点0），与设备的**控制管道（Control Pipe）**相连。  
- 端点0的控制管道包含描述USB设备的所有信息，主要包括以下类别：  
  1. **标准信息**：适用于所有USB设备的通用信息，例如供应商标识、设备类别和电源管理能力。设备、配置、接口和端点描述符提供设备的配置相关信息（详见第9章）。  
  2. **类别信息**：根据设备类别的不同而定义。  
  3. **供应商信息**：设备供应商可以任意定义的信息，格式未在USB规范中规定。  

此外，每个USB设备还包含USB控制和状态信息。  

---

#### **4.8.2 设备描述**  
USB设备主要分为两类：**集线器（Hub）**和**功能设备（Function）**：  
- **集线器**：提供额外的USB连接点。  
- **功能设备**：为主机提供额外的功能。  

##### **4.8.2.1 集线器（Hubs）**  
- 集线器是USB即插即用架构中的关键元素。  
- 集线器简化了用户的USB连接操作，同时以较低的成本和复杂性提供稳健的连接功能。  

**集线器的特性**：  
- 集线器是**布线集线器**，支持USB的多点连接特性。  
- **端口**是集线器的连接点，每个集线器将一个连接点扩展为多个连接点。  
- USB架构支持多个集线器的级联连接。  

**集线器的连接**：  
- 集线器的**上游端口**连接到主机。  
- 集线器的**下游端口**连接到其他集线器或功能设备。  
- 集线器能够检测每个下游端口的设备连接或断开，并为下游设备分配电源。  
- 每个下游端口可单独启用，支持高速、全速或低速设备。  

**USB 2.0集线器的组成**：  
1. **集线器控制器（Hub Controller）**：负责与主机通信。  
2. **集线器中继器（Hub Repeater）**：一种协议控制的开关，连接上游端口和下游端口，支持重置和挂起/恢复信号的硬件功能。  
3. **事务转换器（Transaction Translator）**：支持高速传输，同时为集线器后面的全速/低速设备提供数据传输机制。

### 4.8.2.2 功能设备（Functions）  

**功能设备**是能够通过USB总线发送或接收数据或控制信息的USB设备。通常，功能设备以单独的外设形式实现，并通过电缆连接到集线器的端口。然而，一个物理设备包可以同时实现多个功能设备和一个嵌入式集线器，并使用单根USB电缆连接到主机。这种设备被称为**复合设备（Compound Device）**。  

- **复合设备**在主机上表现为一个集线器及一个或多个不可移除的USB设备。  

每个功能设备都包含描述其能力和资源需求的**配置信息**。  
在功能设备被使用之前，必须由主机对其进行配置，包括：  
1. 分配USB带宽。  
2. 选择功能设备特定的配置选项。  

**功能设备示例**：  
- **人机接口设备（HID）**：如鼠标、键盘、绘图板或游戏控制器。  
- **成像设备**：如扫描仪、打印机或摄像头。  
- **大容量存储设备**：如CD-ROM驱动器、软盘驱动器或DVD驱动器。  

---

### 4.9 USB主机：硬件与软件  

USB主机通过**主机控制器（Host Controller）**与USB设备交互，主机负责以下任务：  
- 检测USB设备的连接与断开。  
- 管理主机与USB设备之间的**控制流**。  
- 管理主机与USB设备之间的**数据流**。  
- 收集状态和活动统计信息。  
- 为连接的USB设备提供电源。  

主机中的**USB系统软件**负责管理USB设备与主机设备软件之间的交互。这些交互涉及以下五个方面：  
1. **设备枚举与配置**：识别并初始化设备。  
2. **同步数据传输**：如实时音频或视频流。  
3. **异步数据传输**：如批量数据或中断数据。  
4. **电源管理**：控制和优化设备的供电状态。  
5. **设备和总线管理信息**：管理设备状态及总线拓扑结构。  

---

### 4.10 架构扩展  

USB架构在**主机控制器驱动程序（Host Controller Driver）**和**USB驱动程序（USB Driver）**之间的接口处提供了良好的可扩展性：  
- 支持实现多个主机控制器及其相关的主机控制器驱动程序。  
- 这种扩展性允许主机系统支持多个并发的USB控制器，以满足更复杂的系统需求。

### 第5章 USB数据流模型  

本章介绍了USB中数据如何传输的相关信息。这些内容对于所有开发人员都具有重要意义。章节内容超越了系统信号和协议定义的具体细节，提供了更高层次的框架信息。  
- **第7章**和**第8章**提供了关于USB系统部分的更多细节。  
- **第9章到第11章**进一步扩展了本章所提供的框架信息。  

建议所有USB开发人员阅读本章，以便理解USB的关键概念。  

---

### 5.1 实施者视角  

USB为主机与连接的USB设备之间提供通信服务。然而，用户看到的简单场景（如图5-1所示）——将一个或多个USB设备连接到主机——在实际实现中比图示的内容复杂得多。  

为了更好地解释USB的具体要求，从不同开发者的视角来看USB系统需要不同的理解。为了向最终用户提供可靠的操作体验，USB必须支持多个重要概念和特性。  
USB以**分层方式**呈现，便于解释并使特定USB产品的开发者可以专注于与其产品相关的细节。  

---

#### **图5-1 简单的USB主机/设备视图**  
（此处为图示描述：简单的主机连接到一个USB设备）

---

图5-2提供了USB更深层次的概览，标识了系统中不同的层级，这些层级将在规范的其他部分详细描述。USB的实现主要包含以下四个关键领域：  

1. **USB物理设备（USB Physical Device）**  
   - 通过USB电缆连接的硬件设备，用于执行对用户有用的功能。  

2. **客户端软件（Client Software）**  
   - 在主机上运行的软件，与特定的USB设备对应。  
   - 客户端软件通常由操作系统提供，或者随USB设备一起提供。  

3. **USB系统软件（USB System Software）**  
   - 支持USB的操作系统相关软件。  
   - USB系统软件通常由操作系统提供，与具体的USB设备或客户端软件无关。  

4. **USB主机控制器（Host Controller，主机端总线接口）**  
   - 硬件和软件的结合，允许USB设备连接到主机。  

---

#### **图5-2 分层的USB系统视图**  
（此处为图示描述：展示USB主机与设备的交互涉及多个层级和实体）

---

这四个USB系统组件之间存在共享的权利与责任，规范的其余部分将描述支持功能与客户端之间稳健可靠通信所需的细节。  

图5-2显示，主机与设备的简单连接需要多个层级和实体之间的交互：  
- **USB总线接口层**：提供主机与设备之间的物理、信号和数据包连接。  
- **USB设备层**：USB系统软件通过此层与设备执行通用的USB操作。  
- **功能层（Function Layer）**：通过适配的客户端软件层，为主机提供额外功能。  

USB设备层和功能层的逻辑通信视图依赖于USB总线接口层完成数据传输。  

---

#### **物理与逻辑通信的关系**  

- **第6章、第7章和第8章**描述了USB通信的物理视图。  
- **第9章和第10章**描述了USB通信的逻辑视图。  

本章描述了影响USB开发者的关键概念，建议在阅读规范其他部分之前先阅读本章，以便了解与自身产品最相关的细节。  

---

### 描述和管理USB通信的关键概念  

为了更好地描述和管理USB通信，本章介绍以下重要概念：  

1. **总线拓扑（Bus Topology）**  
   - **第5.2节**介绍了USB的主要物理和逻辑组件及其相互关系。  

2. **通信流模型（Communication Flow Models）**  
   - **第5.3节到第5.8节**描述了主机与设备之间通信的方式，并定义了四种USB传输类型。  

3. **总线访问管理（Bus Access Management）**  
   - **第5.11节**描述了主机如何管理总线访问，以支持USB设备的多种通信流。  

4. **同步传输的特殊考虑（Special Considerations for Isochronous Transfers）**  
   - **第5.12节**介绍了USB在支持需要同步数据传输的设备时的特性。  
   - 非同步设备的开发者无需阅读**第5.12节**。
### 5.2 总线拓扑（Bus Topology）  

USB拓扑主要由以下四部分组成：  

1. **主机与设备（Host and Devices）**  
   - USB系统的核心组件。  

2. **物理拓扑（Physical Topology）**  
   - 描述USB元素如何物理连接在一起。  

3. **逻辑拓扑（Logical Topology）**  
   - 定义各种USB元素的角色和职责，以及从主机和设备的角度看USB的表现形式。  

4. **客户端软件与功能关系（Client Software-to-function Relationships）**  
   - 描述客户端软件与USB设备上相关功能接口之间的相互关系。  

---

### 5.2.1 USB主机（USB Host）  

USB主机的**逻辑组成**如图5-3所示，包括以下部分：  

- **USB主机控制器（Host Controller）**  
- **聚合的USB系统软件**（包括USB驱动程序、主机控制器驱动程序和主机软件）  
- **客户端软件（Client Software）**  

作为USB系统的协调实体，USB主机占据了独特的位置。除了其特殊的物理位置外，主机还承担了以下具体职责：  

1. **控制USB的所有访问权限**：  
   - USB设备只有在主机授予访问权限时才能访问总线。  

2. **监控USB拓扑结构**：  
   - 主机负责管理USB的拓扑，并确保设备的正确连接和功能。  

有关主机及其职责的完整讨论，请参阅**第10章**。  

---

### 5.2.2 USB设备（USB Devices）  

USB物理设备的**逻辑组成**如图5-4所示，包括以下部分：  

- **USB总线接口（USB Bus Interface）**  
- **USB逻辑设备（USB Logical Device）**  
- **功能模块（Function）**  

USB物理设备为主机提供额外的功能。这些功能的类型差异很大，但所有USB逻辑设备都向主机呈现相同的基础接口。这种统一的接口使得主机能够以相同的方式管理不同USB设备的USB相关部分。  

#### **USB设备的识别与配置**  
为了帮助主机识别和配置USB设备，每个设备都会携带并报告配置信息：  

1. **通用信息**：  
   - 所有逻辑设备共有的信息，例如设备类型及基本能力。  

2. **特定信息**：  
   - 与设备提供的功能相关的特定信息。  

配置信息的详细格式会因设备类别的不同而有所变化。  

有关USB设备的完整讨论，请参阅**第9章**。


### 5.2.3 物理总线拓扑（Physical Bus Topology）  

USB设备通过**分层星形拓扑（Tiered Star Topology）**物理连接到主机，如图5-5所示。  

- **USB连接点**由一种特殊类别的USB设备提供，这种设备称为**集线器（Hub）**。  
- 集线器提供的额外连接点称为**端口（Ports）**。  
- 主机包含一个嵌入式集线器，称为**根集线器（Root Hub）**，通过根集线器提供一个或多个连接点。  
- 提供额外功能的USB设备称为**功能设备（Functions）**。  

为了防止循环连接，USB的星形拓扑采用分层顺序，这种结构形成了如图5-5所示的**树形配置（Tree-like Configuration）**。  

---

#### **复合设备与复合式功能设备**  

- 多个功能可以组合在一个物理设备中。例如，一个键盘和轨迹球可以封装在一个设备中。在设备内部，各个功能永久连接到一个集线器，而集线器连接到USB。  
- 当多个功能与一个集线器组合在一个设备中时，此设备被称为**复合设备（Compound Device）**。  
  - 集线器和每个功能都分配有独立的设备地址。  

- **复合式功能设备（Composite Device）**：  
  - 此类设备具有多个接口，每个接口独立控制，但只有一个设备地址。  
  - 从主机的角度来看，复合设备与单独的集线器及其附加功能没有区别。  

图5-5中还展示了复合设备的例子。  

---

#### **高速系统中的集线器角色**  

在高速USB系统中，集线器起着特殊作用：  
- 集线器将**全速/低速信号环境**与**高速信号环境**隔离。  
- 图5-6展示了一个支持高速设备的高速集线器。  
- 集线器允许USB 1.1集线器以全速/低速运行，并支持其他仅支持全速/低速的设备连接。  
- 主机控制器也可以直接支持全速/低速设备的连接。  

**集线器的作用**：  
- 每个高速运行的集线器本质上新增了一个或多个全速/低速总线（即每个集线器支持额外的12 Mb/s全速/低速带宽）。  
- 这使得系统可以连接更多的全速/低速总线，而无需增加主机控制器的数量。  

尽管可以有多个12 Mb/s全速/低速总线，但每个主机控制器最多只能连接127个USB设备。  

有关集线器如何实现信号环境隔离的详细信息，请参阅**第11章**。  

---

### 5.2.4 逻辑总线拓扑（Logical Bus Topology）  

尽管USB设备在物理上以**分层星形拓扑**方式连接到主机，但主机与每个逻辑设备的通信表现为设备直接连接到**根端口（Root Port）**的形式。这种逻辑视图如图5-7所示，对应于图5-5中的物理拓扑。  

- **集线器也是逻辑设备**，但为了简化图示，图5-7未显示集线器。  
- 大多数主机与逻辑设备的活动使用这种逻辑视图，但主机仍会保持对物理拓扑的感知。  

---

#### **处理集线器移除的逻辑拓扑更新**  

当一个集线器被移除时：  
- 主机必须从逻辑拓扑中移除与该集线器连接的所有设备。  

有关集线器更详细的讨论，请参阅**第11章**。