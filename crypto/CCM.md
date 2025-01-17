
Counter with CBC-MAC (CCM) is a generic authenticated encryption
   block cipher mode.  CCM is only defined for use with 128-bit block
   ciphers, such as AES [AES].  The CCM design principles can easily be
   applied to other block sizes, but these modes will require their own
   specifications.

**CCM mode** (**counter with cipher block chaining message authentication code**; **counter with [CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC "CBC-MAC")**) 是块密码的一种工作模式。它是一种认证加密，旨在提供身份验证和机密性。 CCM 模式仅针对分组长度为 128 位的分组密码定义。 [1]

必须谨慎选择 CCM 的随机数，以免对同一个密钥重复使用。这是因为 CCM 是计数器 (CTR) 模式的派生，而后者实际上是一种流密码。 [2]

顾名思义，CCM 模式结合了用于加密的计数器 (CTR) 模式和用于身份验证的密码块链消息身份验证码 (CBC-MAC) 。这两个原语以“先验证后加密”（MAC then Encrypt, MtE）的方式应用：首先对消息计算 CBC-MAC，以获得消息验证码 (MAC) ，然后使用计数器模式对消息和 MAC 进行加密。主流观点认为，只要加密中使用的计数器值不与身份验证中使用的（预）初始化向量冲突，就可以将相同的加密密钥用于两者。基于底层分组密码的安全性，该组合存在可证明安全性[3] 。该证明还适用于任何块大小以及任何大小的强加密伪随机函数的 CCM 推广（因为在计数器模式和 CBC-MAC 中，块密码仅在一个方向上使用）。

CCM 需要对认证加密信息的每个块进行两次分组密码加密操作，并对关联的经过验证的数据的每个块进行一次加密。


### 模式规范

CCM 是一种通用的认证加密分组密码模式。CCM 仅定义用于 128 位分组密码（如 AES）。尽管 CCM 的理念可以轻松扩展到其他分组大小，但这需要额外的定义。

---

#### 1.1 通用 CCM 模式

对于通用 CCM 模式，有两个参数需要选择：

1. **M**：认证字段的大小。选择 M 的值需要在消息膨胀和攻击者能够未被检测地篡改消息的概率之间进行权衡。有效值为 4、6、8、10、12、14 和 16 个字节。
2. **L**：长度字段的大小。此值需要在消息的最大大小与 Nonce 的大小之间进行权衡。不同的应用需要不同的权衡，因此 L 是一个参数。L 的有效值范围为 2 至 8 个字节（L=1 的值被保留）。

**参数说明表：**

| 名称 | 描述                         | 字段大小 | 编码方式 |
|------|------------------------------|----------|----------|
| M    | 认证字段的字节数             | 3 位      | (M-2)/2  |
| L    | 长度字段的字节数             | 3 位      | L-1      |

---

#### 1.2 输入

为了发送消息，发送方需要提供以下信息：

- **加密密钥 K**：适用于分组密码的加密密钥。
- **Nonce N**：长度为 \(15-L\) 个字节。在任何加密密钥 K 的范围内，Nonce 值必须是唯一的。也就是说，与任意密钥一起使用的 Nonce 值集合不得包含重复值。如果对两个不同的消息使用相同的密钥和 Nonce，会破坏此模式的安全性。
- **消息 m**：由长度为 \(l(m)\) 字节的字符串组成，其中 \(0 \leq l(m) < 2^{8L}\)。长度限制确保 \(l(m)\) 能够编码在 L 字节的字段中。
- **附加认证数据 a**：由长度为 \(l(a)\) 字节的字符串组成，其中 \(0 \leq l(a) < 2^{64}\)。此附加数据会被认证但不会被加密，也不会包含在此模式的输出中。它可以用于认证明文包头，或影响消息解释的上下文信息。不希望认证附加数据的用户可以提供一个长度为零的字符串。

**输入参数说明表：**

| 名称 | 描述                     | 字段大小          | 编码方式          |
|------|--------------------------|-------------------|-------------------|
| K    | 分组密码密钥             | 取决于分组密码    | 不指定            |
| N    | Nonce                   | \(15-L\) 个字节   | 字节字符串         |
| m    | 要加密和发送的消息       | \(l(m)\) 个字节   | 字节字符串         |
| a    | 附加认证数据             | \(l(a)\) 个字节   | 字节字符串         |

### 1.3 认证

认证的第一步是计算认证字段 **T**，这通过 **CBC-MAC**（Cipher Block Chaining - Message Authentication Code）完成。具体实现步骤如下：

---

#### 构建块序列 B0, B1, ... Bn

我们首先定义一个块序列 \( B_0, B_1, \dots, B_n \)，然后对这些块应用 **CBC-MAC**。

- **第一个块 B0 的格式**如下，其中 \( l(m) \) 按高位字节优先顺序编码：

| 字节编号 (Octet no) | 内容 (Contents)       |
|---------------------|-----------------------|
| 0                   | Flags                |
| 1 ... \(15-L\)      | Nonce \(N\)          |
| \(16-L\) ... 15     | \(l(m)\)             |

---

#### **Flags 字段的格式**

在第一个块 \( B_0 \) 中，**Flags 字段**的格式如下：
![[Pasted image 20241202234239.png]]
- **Reserved 位**：为将来的扩展保留，必须始终设置为 0。
- **Adata 位**：如果 \( l(a) = 0 \)，则设置为 0；如果 \( l(a) > 0 \)，则设置为 1。
- **M 字段**：用 \( (M-2)/2 \) 编码 M 的值。由于 M 的可能值为 4 到 16（偶数），因此 3 位字段对应可能的值范围为 1 到 7。
- **L 字段**：用 \( L-1 \) 编码 L 的值。L 的可能值范围为 2 到 8（注意 L=1 是保留值），因此 3 位字段对应可能的值范围为 1 到 7。

---

#### **添加附加认证数据**

如果 \( l(a) > 0 \)（由 **Adata 位**指示），则需要添加一个或多个块来表示附加认证数据。这些块包括 \( l(a) \) 和 \( a \) 的可逆编码形式。具体步骤如下：

1. **编码 \( l(a) \)**：
   - 如果 \( 0 < l(a) < 2^{16} - 2^8 \)，长度字段用 2 个字节编码 \( l(a) \)，采用高位字节优先顺序。
   - 如果 \( 2^{16} - 2^8 \leq l(a) < 2^{32} \)，长度字段用 6 个字节编码，包括字节序列 `0xFF, 0xFE` 和 4 个字节的 \( l(a) \) 编码（高位字节优先）。
   - 如果 \( 2^{32} \leq l(a) < 2^{64} \)，长度字段用 10 个字节编码，包括字节序列 `0xFF, 0xFF` 和 8 个字节的 \( l(a) \) 编码（高位字节优先）。

---

#### **附加认证数据的长度编码规则**

以下表格总结了附加认证数据的长度编码规则。注意，所有字段均按高位字节优先顺序解释。

| 前两个字节 (First two octets) | 后续字段 (Followed by) | 说明 (Comment)                  |
|-------------------------------|------------------------|----------------------------------|
| 0x0000                        | 无                    | 保留 (Reserved)                 |
| 0x0001 ... 0xFEFF             | 无                    | \( 0 < l(a) < 2^{16} - 2^8 \)   |
| 0xFF00 ... 0xFFFD             | 无                    | 保留 (Reserved)                 |
| 0xFFFE                        | 4 个字节 \(l(a)\)      | \( 2^{16} - 2^8 \leq l(a) < 2^{32} \) |
| 0xFFFF                        | 8 个字节 \(l(a)\)      | \( 2^{32} \leq l(a) < 2^{64} \) |

### 块编码附加认证数据

#### 处理附加认证数据块
编码附加认证数据 \(a\) 的块是通过以下步骤生成的：

1. 将编码 \(l(a)\) 的字符串与 \(a\) 连接起来。
2. 将结果划分为 16 字节的块（如果最后一个块不足 16 字节，则用零填充）。
3. 这些块被追加到第一个块 \(B_0\) 之后。

---

#### 处理消息块
1. 在添加（可选的）附加认证数据块之后，我们再添加消息块。
2. 消息块通过将消息 \(m\) 分割为 16 字节的块创建（如果最后一个块不足 16 字节，则用零填充）。
3. 如果消息 \(m\) 是空字符串，则在此步骤中不会添加任何块。

---

#### 计算 CBC-MAC
结果的块序列为 \(B_0, B_1, \dots, B_n\)。通过以下公式计算 CBC-MAC：

1. 初始计算：  
   \( X_1 := E(K, B_0) \)

2. 依次计算：  
   \( X_{i+1} := E(K, X_i \oplus B_i) \) （对于 \(i = 1, \dots, n\)）

3. 最终生成认证字段 \(T\)：  
   \( T := \text{first-M-bytes}(X_{n+1}) \)

其中：
- \(E()\) 是分组加密函数。
- \(T\) 是 MAC（消息认证码）值。  
最后一个块 \(B_n\) 被与 \(X_n\) 进行 XOR 操作，并将结果通过加密生成。如果需要，可以截断密文以得到 \(T\)。

---

### 1.4 加密

为了加密消息数据，我们使用 **计数器模式（CTR）**。定义密钥流块的方式如下：

#### 密钥流块的定义
\( S_i := E(K, A_i) \quad \text{(对于 \(i=0, 1, 2, ...\))} \)

- \(A_i\) 的格式如下，其中 \(i\) 按高位字节优先顺序编码：

| 字节编号 (Octet no) | 内容 (Contents)       |
|---------------------|-----------------------|
| 0                   | Flags                |
| 1 ... \(15-L\)      | Nonce \(N\)          |
| \(16-L\) ... 15     | Counter \(i\)        |

---

#### **Flags 字段的格式**

在每个块 \(A_i\) 中，Flags 字段的格式如下：

| 位编号 (Bit no) | 内容 (Contents)                 |
|-----------------|---------------------------------|
| 7               | Reserved (保留位，始终为 0)      |
| 6               | Reserved (对应 \(B_0\) 中的 Adata 位，但未使用；设为 0) |
| 5, 4, 3         | Reserved (保留位，设为 0)       |
| 2, 1, 0         | \(L\) (长度字段大小的编码)      |

- **Reserved 位**：为将来的扩展保留，必须设置为 0。
- **L 字段**：与 \(B_0\) 中的编码方式相同，用 \(L-1\) 编码。

**注意**：由于第 3、4 和 5 位全为 0，因此所有 \(A\) 块与 \(B_0\) 是不同的。

---

#### 加密消息
消息通过以下方式加密：

1. 将消息 \(m\) 的字节与密钥流块 \(S_1, S_2, S_3, ...\) 的前 \(l(m)\) 字节逐字节 **XOR**。
2. 注意：\(S_0\) 不用于加密消息。

---

#### 计算认证值 \(U\)

认证值 \(U\) 是通过以下方式计算的：

1. 将 \(T\) 与密钥流块 \(S_0\) 的前 \(M\) 字节进行 XOR：
   \( U := T \oplus \text{first-M-bytes}(S_0) \)

---

### 总结
1. **认证**通过 CBC-MAC 计算，生成消息认证码 \(T\)。
2. **加密**通过 CTR 模式完成，将消息块与密钥流块按位 XOR。
3. 最终的认证值 \(U\) 是通过对 \(T\) 和 \(S_0\) 的部分 XOR 生成。