在序言里的例子稍作修改：

```asm
mov dword ptr es:[eax + ecx * 8 + 0x11223344], 0x12345678
```

将 word ptr 指示字改回 **dword ptr**，这是个具有典型指令编码意义的指令。

它的 encodes（机器编码）是：**26 c7 84 c8 44 33 22 11 78 56 34 12** (共12个字节)。

# 1. 编码序列

![format](https://github.com/user-attachments/assets/6d32e43f-b797-4ad9-97b0-cf816f3af8b2)

如上图所示：这是 x86/x64 体系的 General-Pupose Instruction（通用体系指令）的编码格式，记住这个编码序列很重要，这是解析指令编码的基石。

这个编码序列分为：

1. Legacy Prefix
2. REX prefix
3. Opcode
4. ModRM
5. SIB
6. Displacement
7. Immediate

按功能组别，我将这个指令序列分为 4 个部分：

- Prefix
- Opcode
- ModRM 与 SIB
- Displacement 与 Immediate

其中，只有 opcode 是必须的，其它组成部分都可选！

## 1.1 Prefix(前缀)
AMD推出 x86 扩展 64 位技术时，增加了一个用于访问扩展的 64 位数据 prefix，它是：

- REX prefix

而 x86 原有的 prefix 则变为了指令格式中的：

- Legacy prefix

REX prefix 仅存在于 x64 的 64-bit 模式中，在 legacy x86 模式下，REX prefix 是无效的，但是在 x64 的 64-bit 模式下 Legacy prefix 是有效的。
REX prefix 的取值范围是：

- 40H - 4FH

在非 64 位模式下，它们是 inc 与 dec 指令，也就是说这些指令在 64 位模式下被重定义为 REX prefix

## 1.2 Opcode（操作码）

大多数通用指令的 Opcode 是单字节，最多是 2 字节。但是对 x87 FPU 指令和 SSEx 等 SMID 指令来说可以有 3 个字节的，Opcode 码代表着指令是做什么操作。

一些 Opcode 并不是完整的 Opcode 码，它需要 ModRM 字节进行辅助，这类型的 Opcode 被分成以 Group（组）为单位的一组 Opcode，同一组的 Opcode 有着相同的 operand 属性。

1.3 ModRM 与 SIB

ModRM 字节为指令**提供 operand 的寻址**。从某个角度看可以认为是 operand 的属性！包括：

- operand 类型
- operand size

某些 Opcode 还需要 ModRM 字节进行辅助，ModRM 的结构分为三部分：

- ModRM [7:6] - Mod 部分
- ModRM [5:3] - Reg 部分
- ModRM [2:0] - R/M 部分

ModRM.mod 提供 operand 的寻址模式，可以从 memory 或 register 中进行选择，ModRM.reg 提供具体 register 的 ID 值，ModRM.r/m 可以选择 Register/Memory 寻址由 ModRM.mod 决定。 也就是：提供 register 的 ID 值，或者内存寻址中的 base 寄存器的 ID 值！

SIB 提供 memory 操作数中的 base 寄存器，index 寄存器以及 scale 值，同样分为三部分：

- SIB [7:6] - Scale 值
- SIB [5:3] - Index 寄存器 ID
- SIB [2:0] - Base 寄存器 ID

这两个字节都是用来提供指令操作数，实现操作数的寻址。

## 1.4 Displacment 与 Immediate

displacement 与 immediate 直接嵌在指令编码序列里。 displacement 需要 ModRM 甚至 SIB 提供寻址，而 immediate 则在 Opcode 里提供寻址。

从严格意义来说：D**isplacement 必须由 ModRM 字节提供寻址，它最大为 4 个字节**。然而在 64 位模式下，某些 MOV 指令支持 64 位 offset 值，可是这个 64 位的 offset 值严格来说它并不是 displacment，我们可以认为它是一个 immediate 值。

immediate 最大可为 8 个字节，**在 x64 的 64 位下的某些情况才会有的**
```
注意：displacement 和 immediate 的两种情况：signed 与 **unsigned
```

### ★ 符号数（signed）

在下面的情况下，它们是符号：

- 进行运算时，它们是符号数
- operand size 扩展时，它们是符号数

> 实际上：operand size 具有 z 或 b 属性的指令，才可能进行 sign-extended（符号扩展）行为。
>
> 1. 当指令的 operand size 是 z 属性时，如果 immediate 是小于 effective operand size 的，immediate 会进行 sign-extended 行为。
> 2. 同样 operand size 是 b 属性的，如果  immediate 是小于 effective operand size 的，immediate 也会进行 sign-extended 行为。
> 3. 当指令的 displacement size 小于 address size  的，displacement 会进行 sign-extended 行为。

有关指令的属性，详见 operand type 表 以及 operand size 表

有关 effective operand size 方面，详见  default（缺省）与 effective（有效）

### ★ 无符号数（unsigned）

在这种情况下，displacement 是一个绝对地址值，immediate 是一个无符号立即数值。

- mov eax, dword ptr [0x11223344]
- mov eax, 0x12345678

上面例子中的 displacement 和 immediate 是 unsigned 数

> 本质上：所有 displacement 都是 signed（符号数），displacement 是基于 base 地址的一个 offset（偏移量）。
> ★ 当地址形式中无 base 寄存器时，虽然它是绝对值形式，实际上它仍是基于 segment base 地址。

因此，只有 immediate 才具有的 unsigned 的时候。

# 2. 指令长度

上图中显示，指令长度最长是 15 个字节，在什么时候达到饱和的 15 个字节呢？

答案是如下类似指令：

```asm
lock add dword ptr es:[eax+ecx*8+0x11223344], 0x12345678
```

当在 16 位模式代码下，这条指令将达到饱和的 15 个字节长度

注意：仅在 16 位下，这条指令的编码是：

- **26 66 67 F0 81 84 C8 44 33 22 11 78 56 34 12**（正好 15 个字节)

这个 encode 的意义是：

- **26 66 67 F0**：这 4 个字节是 prefix，这 4 个字节达到了饱和的 prefix 状态。
  - **26** 是 ES segment register prefix
  - **66** 是 operand-size override prefix
  - **67** 是 address-size override prefix
  - **F0** 是 Lock prefix
- **81**：Opcode
- **84**：ModRM
- **C8**：SIB
- **44 33 22 11**：displacement
- **78 56 34 12**：immediate

有没有超过 15 个字节的指令编码，答案是：没有。那么在 64 位下呢？答案同样是没有！

## 既然这样，顺便提一提：为什么指令长度最长是 15 字节

4 个字节的 prefix 已经达到饱和度了： 1 个字节用来改写 segment selector registers，1 个字节用来改写 Operand effective size，1 个字节用来改写 Address effective size，还有 1 个字节用来 lock 总线。已经无法再增加 prefix 了。

若是采用 2 个字节的 Opcode 码，则寻址模式上不会有 mem32, imm32 这种操作法。所以还是采用 1 个字节的 Opcode，而得到 4 个字节的立即数。

ModRM + SIB：2 个字节。

4 个字节的 displacement 值。

4 个字节的 immediate 值。

从理论上来讲：这样每个组成部分都呈饱和状态，加起来总共 15 个字节，而只有采用 mem32, imme32　这种寻址模式可能会达到饱和状态。 在 64 位下，若采用 mem, imme 的寻址模式，这和 32 位是一致的，所以不会超越 15 个字节。

实际上，processor 将保证实施这一限制，当检测到超过 15 bytes 的指令，**processor 将会抛出 #GP 异常**，终止指令的执行。

# 3. encodes 的字节序

x86/x64 指令的 encodes 在内存中是以 little endian 存储的，这主要体现在 displacement 和 immediate 部分。

在 encode 序列里，legacy prefix 在低端，依次往上，最后的 immediate 部分在最高端。

```asm
mov dword ptr es:[eax + ecx * 8 + 0x11223344], 0x12345678
```

指令中的 displacement 是 0x11223344，在内存的 little endian 序列是：44 33 22 11

immediate 是 0x12345678 在内存的 lttle endian 序列是：78 56 34 12

关于指令长度的论述，在《解开 x86/x64 指令长度的迷惑》一文