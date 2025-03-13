原文：[ScriptLZ](https://www.uni-ulm.de/fileadmin/website_uni_ulm/iui.inst.190/Lehre/SS14/Datenkompression/ScriptLZ.pdf)

# 1、后缀数组
&emsp;&emsp;对于字符串 $`S`$，它的后缀数组是其所有后缀按字典序排序的数组（也称为字母顺序，字典或词汇顺序），此排序方式由字符集 $`\Sigma`$ 决定的。$`\Sigma`$ 是一个具有固定大小 $`\sigma`$ 的集合，为了方便起见将 $`\Sigma`$ 视为一个大小为 $`\sigma`$ 的数组，其中的字符在 $`\Sigma[1\ldots\sigma]`$ 按照升序排序即 $`\Sigma[1] \lt \Sigma[2] \lt \ldots \lt \Sigma[\sigma] `$，反过来 $`\Sigma`$ 的每个符号被映射到数字 $`\{1, \ldots, \sigma\}`$，最小的被映射到 1，次小的被映射到 2，这样可以将字符用作数组的索引。

> ## 定义 1.1
> 令 $`\lt`$ 是字符集 $`\Sigma`$ 的排序方式，同时也是 $`\Sigma_{}^*`$（字符串集）的排序方式：对于 $`s,t\in\Sigma_{}^*`$，$`s\lt{t}`$ 当且仅当 $`s`$ 是 $`t`$ 的前缀，或有字符串 $`u,v,w\in\Sigma_{}^*`$ 且字符 $`a, b\in\Sigma`$ 并且 $`a\lt{b}`$，则 $`s=uav`$，$`t=ubw`$。

&emsp;&emsp;为确定两个字符串的字典序，从开始依次比较每一个字符，找到第一个不同的字符并对比，在字符集中靠前的字母所在的字符串在字典序的前面，如果在到达任意一个字符串尾部后没有找到不同的字符，则较短的字符串在字典序的前面。
&emsp;&emsp;在为了确定同一字符串 $`S`$ 的两个后缀的字典序的算法中，为避免繁琐地区分“有更多字符”或“没有更多字符”，可以在 $`S`$ 后附加特殊字符 $（标识符）且假定 $ 在字符集中 $`\Sigma`$ 中最小，如果 $`S`$ 中其他地方没有出现 $，并且 $`S`$ 的两个后缀的字典顺序如上所述确定，则一个后缀没有更多字符可供比较的情况就不会发生。我们稍后会看到，在其他情况下，将特殊符号 $ 附加到字符串也很方便。

>> | $`i`$ |SA|ISA| $`S_{SA[i]}`$ |
>> |:-:|:-:|:-:|:--|
>> |1|3|5|$`aataatg`$|
>> |2|6|7|$`aatg`$|
>> |3|4|1|$`ataatg`$|
>> |4|7|3|$`atg`$|
>> |5|1|8|$`ctaataatg`$|
>> |6|9|2|$`g`$|
>> |7|2|4|$`taataatg`$|
>> |8|5|9|$`taatg`$|
>> |9|8|6|$`tg`$|
> **图 1**：字符串 $`S = ctaataatg`$ 的后缀数组和逆后缀数组

> ## 定义 1.2
> 令 $`S`$ 是长度为 $`n`$ 的字符串，对于任何 $`i (1\leq{i}\leq{n})`$，$`S_i`$ 表示 $`S`$ 的第 $`i`$ 个后缀 $`S[i\ldots{n}]`$，字符串 $`S`$ 的后缀数组 $`SA`$ 是一个 $`1\ldots{n}`$ 的整数数组，指定字符串 $`S`$ 的 $`n`$ 个后缀的字典序，并且满足 $`S_{SA[1]}\lt{S_{SA[2]}}\lt{S_{SA[3]}}\lt\ldots\lt{S_{SA[n]}}`$，字符串 $`S`$ 的逆后缀数组 $`ISA`$ 是一个长度为 $`n`$ 的数组，对于任何 $`k (1 \leq k \leq n)`$，满足 $`ISA[SA[k]] = k`$。
> 总之，$`SA[i]=k`$ 是“第 $`i`$ 小的后缀是从 $`k`$ 位置开始的后缀”，$`ISA[k]=i`$ 是 ”$`k`$ 位置开始的后缀是第 $`i`$ 小的后缀”

&emsp;&emsp;逆后缀数组有时也称为秩数组，因为 $`ISA[i]`$ 指定了按字典顺序排列的后缀中第 $`i`$ 个后缀的秩。更准确地说，当 $`ISA[i] = j`$，则 $`S_i`$ 是按字典序第 $`j`$ 小的字符串，显然可以通过后缀数组线性计算逆后缀数组，图 1 显示了字符串 $`S = ctaataatg`$ 的后缀数组和逆后缀数组。
&emsp;&emsp;后缀数组由 Manber 和 Myers [MM93] 提出，并由 Gonnet 等人 [GBYS92] 独立设计，命名为 PAT 数组。十年后，Kärkkäinen 和 Sanders [KS03]、Kim 等人 [KSPP03]、Ko 和 Aluru [KA03] 以及 Hon 等人 [HSS03] 同时独立证明了后缀数组的直接线性时间构造是可能的。迄今为止，已知有 20 多种不同的后缀数组构造算法 (SACA) [PST07]。

# 2、Lempel-Ziv 分解

&emsp;&emsp;30 年来，字符串的 Lempel-Ziv 因式分解 [ZL77] 在数据压缩中发挥着重要作用（例如，它用于 gzip），最近它被用作线性时间算法的基础，用于检测字符串中所有最大重复（运行时）。

> ## 算法 1：根据 LZ分解 $`(p_1,l_1),\ldots,(p_m,l_m)`$ 重建字符串 $`S`$
> i$`\leftarrow`$1
> **for** j$`\leftarrow`$1 **to** $`m`$ **do**
> &emsp;**if** $`l_j`$ = 0 **then**
> &emsp;&emsp;$`S[i]\leftarrow{p_j}`$
> &emsp;&emsp;$`i\leftarrow{i+{1}}`$
> &emsp;**else**
> &emsp;&emsp;**for** k$`\leftarrow`$0 **to** $`l_j-1`$ **do**
> &emsp;&emsp;&emsp;$`S[i]\leftarrow{S[p_j+k]}`$
> &emsp;&emsp;&emsp;$`i\leftarrow{i+{1}}`$

> ## 定义 2.1
> 令 $`S`$ 为字符集 $`\Sigma`$ 构成长度为 $`n`$ 的字符串，其 Lempel-Ziv 分解（或称LZ分解）为 $`S \Rightarrow s_1s_2\ldots{s_m}`$，对于任一因子 $`s_j (1\leq{j}\leq{m})`$，其具有两种情况：
> （a）是字符 $`c\in\Sigma`$ 且不在 $`s_1s_2\ldots{s_{j-1}}`$ 中
> （b）是 $`s_1s_2\ldots{s_j}`$ 中至少出现两次的最长子串

&emsp;&emsp;Lempel-Ziv 分解可用一系列二元组 $`(p_1,l_1),\ldots,(p_m,l_m)`$ 来表示，对于模式（a），$`p_j=c, l_j=0`$，对于模式（b），$`p_j`$ 是 $`s_j`$ 在 $`s_1s_2\ldots{s_{j-1}}`$ 中出现的位置且 $`l_j=|{s_j}|`$ （$`s_j`$ 的长度）
&emsp;&emsp;例如，字符串 $`S=acaaacatat`$ 的LZ分解是 $`s_1=a,s_2=c,s_3=a,s_4=aa,s_5=ca,s_6=t,s_7=at`$，可以按以下表示：$`(a,0),(c,0),(1,1),(3,2),(2,2),(t,0),(7,2)`$
&emsp;&emsp;考虑一个全由字符 $`a`$ 构成的长度为 $`n`$ 的字符串，它的LZ分解是 $`(a,0),(1,n−1)`$。

>> |$`i`$|SA|ISA|$`S_{SA[i]}`$|$`PSV_{lex}[i]`$|$NSV_{lex}[i]`$|
>> |:-:|:-:|:-:|:--|:-:|:-:|
>> |0|0||$`\epsilon`$|||
>> |1|$`psv=3`$|3|aaacatat|0|3|
>> |2|$`k=4`$|7|aacatat|1|3|
>> |3|$`nsv=1`$|1|acaaacatat|0|11|
>> |4|5|2|acatat|3|7|
>> |5|9|4|at|4|6|
>> |6|7|8|atat|4|7|
>> |7|2|6|caaacatat|3|11|
>> |8|6|10|catat|7|11|
>> |9|10|5|t|8|10|
>> |10|8|9|tat|8|11|
>> |11|0|0|$`\epsilon`$|||
> **图 2**：字符串 $`S=acaaacatat`$ 的后缀数组（以及其它数组）

&emsp;&emsp;在此节中，将展示通过计算LZ分解来高效地压缩字符串的算法，这些是近期研究成果 [CI08、OG11、GB13、KKP13]。相应的解压算法非常简单，它在算法1中给出。我们首先直观地解释如何计算 $`S`$ 中从位置 $`k`$ 开始的下一个 LZ 因子，考虑当 $`S=acaaacatat`$ 且 $`k=4`$，如图 2所示，在 $`S`$ 的后缀数组 $`SA`$ 中，我们从索引 $`i=ISA[k]`$ 开始，向上找到第一个满足 $`SA[i_{psv}]\lt{k}`$ 的 $`i_{psv}`$，令 $`psv=SA[i_{psv}]`$，向下找到第一个满足 $`SA[i_{nsv}]\lt{k}`$ 的 $`i_{nsv}`$，令 $`nsv=SA[i_{nsv}]`$。
&emsp;&emsp;如图 2 所示，对于 $`k=4`$ 有 $`i=2,psv=3,nsv=4`$，然后计算 $`s_{psv}=lcp(S_{psv},S_k), s_{nsv}=lcp(S_{nsv},S_k)`$ 其中 $`lcp(u,v)`$ 用于计算 $`u`$ 和 $`v`$ 的最长公共前缀，如果 $`|s_{psv}|\gt|s_{nsv}|`$，那么下一个LZ因子是 $`s_{psv}`$，表示为 $`(psv, |s_{psv}|)`$，否则是是 $`s_{nsv}`$，表示为 $`(nsv, |s_{nsv}|)`$。在当前示例中，$`lcp(S_{psv},S_k)=lcp(S_3,S_4)=aa`$，$`lcp(S_{nsv},S_k)=lcp(S_1,S_4)=a`$，所以 $`aa`$ 是在字符串 $`S`$ 位置 4 的下一个LZ因子，表示为 $`(3,2)`$

> 注：此过程首先确定在字符串 $`k`$ 位置开始的后缀在字典序中的位置，在上述示例中，$`k=4\Rightarrow2`$，然后向前、后分别查找，找到第一个长于该后缀的后缀并分别计算最长公共前缀，然后使用较长的前缀作为LZ因子

> ## 算法 2：程序 $`LZ\_Factor(k, psv, nsv)`$
> $`l_{psv}\leftarrow`$|**lcp**$`(S_{psv},S_k)`$|
> $`l_{nsv}\leftarrow`$|**lcp**$`(S_{nsv},S_k)`$|
> **if** $`l_{psv}\gt{l_{nsv}}`$ **then**
> &emsp;$`(p,l)\leftarrow(psv,l_{psv})`$
> **else**
> &emsp;$`(p,l)\leftarrow(nsv,l_{nsv})`$
> **if** $`l=0`$ **then**
> &emsp;$`p\leftarrow{S[k]}`$
> **output** factor $`(p,l)`$
> **return** $`k`$ + max{$`l`$, 1}

&emsp;&emsp;算法 2 基于值 $`psv`$ 和 $`nsv`$，输出在字符串 $`S`$ 的 $`k`$ 位置的LZ因子。此外，它返回下一个要计算的LZ因子的位置。