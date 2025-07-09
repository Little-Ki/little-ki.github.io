原文来自：http://www.mouseos.com/x64/index.html 现已失效

# C 的经典入门例子：

```c
int main() {
    printf("Hello, World\n"); 
    return 0;
}
```

# 1. 在当前 32 位系统下，有下面汇编语句（Intel 格式）：
```asm
mov word ptr es:[eax + ecx * 8 + 0x11223344], 0x12345678
```

### 注 1: “32位系统下”指的是：指令的 default operand size（默认操作数宽度）为 32 位！（CS.D = 1）

从这条语句，我们可以得出以下信息：

- 这是一条 **mov** 指令
- first operand（目标操作数）是 **memory**
- second operand（源操作数）是 **immediate**

对于 memory 操作数，我们还了解到：

- operand size（操作数大小）是 word
- address size（地址大小）是 32 位
- memmory 寻址是 **[base + index * 8 + displacement32]**

对于 immediate 操作数，我们了解到：

- 它的值为 0x12345678，因此，我们可以判断它的 operand size 是 Double Word（32位）!

## 显然：

first operand（目标操作数）的 operand size 大小是 word（2个字节），而 second operand（源操作数）却是一个 double word 大小的 0x12345678，在这条指令中存在 operand size 不兼容的情况。

## 那么：对于这条汇编语句，编译器应该如何处理呢？

- 应该怎样处理 immediate 部分
- 应该选择哪条 mov 指令
- 指令最终的 operand size 是多少
- 在 32 位环境下，如何生成 16 位 operand 或 address 的代码
- 寻址模式是什么

这些问题是我们最终需要掌握的知识，这是本栏目的最终目的。

实际上，大多数的编译会最终形成这样的机器编码是：**26 66 C7 84 C8 44 33 22 11 78 56**

1. double word 大小的“立即数操作数”，会被编译器截断为 word 大小，形成值 0x5678
2. 选择 **MOV Ev, Iz** 指令，它的 opcode 是 **C7**
3. 最终指令的 operand size 是 word 大小，取决于 first operand 的 size
4. 由于**默认的 operand size 是 32 位**，编译器通过插入 operand size override prefix（66H）可以改写为 16 位 operands size！
5. （**注 2：**可以使用 address size override prefix（67H）来来改写为 16 位 address size！）
6. 它的寻址模式是：基址＋变址＋disp32 寻址，它需要提供 SIB 字节

解释：

```asm
mov word ptr es:[eax + ecx * 8 + 0x11223344], 0x12345678
```

|*-*|*-*|
|-|-|
|operand-size override|66H -- prefix|
|segment override|26H -- prefix|
|base|SIB.base = 000|
|index|SIB.index = 001|
|scale|SIB.scale = 11|
|displacement|44 33 22 11|
|immediate|78 56 34 12（截断后为 78 56）|

这条指令的 encodes 各个部分的意义是：

- **26**：指令的 prefix 部分，它是 semgent overrride prefix ，作用是改写内存操作数的段选择子
- **66**：指令的 prefix 部分，它是 operand size override prefix ，作用是改写操作数的默认size
- **C7**：指令的 Opcode 部分，是 mov 指令是操作码
- **84**：指令的 ModRM 部分，定义操作数的属性
- **C8**：指令的 SIB 部分，定义内存操作数的属性
- **44332211**：指令的 displacement 值
- **7856**：指令的 immediate 值

# 2. 随便找一个机器码如：FF 15 D4 81 DF 00

那么它的汇编语句是什么呢？

在解码时，依次对第 1 个字节，第 2 个字节等 ... 进行判断：

- 第 1 个字节是什么？ prefix 还是 opcode
- 同样再判断第 2 个字节是不是 prefix，如果不是 prefix ，那么它就是 opcode 码
- 如果第 2 个字节是 opcode 码，再判这个 opcode 需不需 ModRM 字节，如果需要，第 3 个字节就是 ModRM 字节
- 根据 ModRM 字节判断需不需要 SIB 字节，第 4 个字节如果需它就是 SIB 字节
- 判断 ModRM 字节是否需要 displacment 字节和 immediate 字节

根据上面的逻辑，得出：

- **FF**：这个字节是个具有 Group 属性的 Opcode 码，它进行什么操作需要依赖于 ModRM.Reg 域。 换句话来说，FF 并不是完整的 Opcode 码，它要联合 ModRM.reg 才能确定具体的操作。
- **15**：这个是 ModRM 字节（mod-reg-r/m）: ModRM.mod = 00，ModRM.reg = 010，ModRM.r/m = 101。 其中 ModRM.reg 域被 FF（opcode） 作为确定具体操作的索引值。
- **Opcode + ModRm.reg**: FF /2 最终确定为：Call 指令。
- ModRM.mod = 00：表示操作数是 memory
- ModRM.r/m = 101：表示 memory 操作数是一个直接内存 offset 值（32 位的 displacement）

所以，这个机器码最终被解码为： **call dword ptr [00DF81D4]**

解释：
```
FF 15 D4 81 DF 00
```

|*-*|*-*|
|-|-|
|opcode|FF + ModRM.reg = FF /2 $`\Rightarrow`$ call Ev|
|ModRM| ModRM.mod + ModRM.r/m $`\Rightarrow`$ [disp32]|

这 2 个例子，作为对学习 x86/x64 指令编码的一个感性认识。接下来的章节逐一剖析 x86/x64 指令编码的来龙去脉。