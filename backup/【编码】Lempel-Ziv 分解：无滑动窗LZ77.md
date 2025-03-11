原文：[ScriptLZ](https://www.uni-ulm.de/fileadmin/website_uni_ulm/iui.inst.190/Lehre/SS14/Datenkompression/ScriptLZ.pdf)

# 1、后缀数组
对于字符串 $`S`$，它的后缀数组是其所有后缀按字典序排序的数组（也称为字母顺序，字典或词汇顺序），此排序方式由字母表 $`\Sigma`$ 决定的。$`\Sigma`$ 是一个具有固定大小 $`\sigma`$ 的字母集合，为了方便起见将 $`\Sigma`$ 视为一个大小为 $`\sigma`$ 的数组，其中的字符在 $`\Sigma[1…\sigma]`$ 按照升序排序即 $`\Sigma[1] \lt \Sigma[2] \lt … \lt \Sigma[\sigma] `$，反过来 $`\Sigma`$ 的每个符号被映射到数字 $`\{1, …, \sigma\}`$，最小的被映射到 1，次小的被映射到 2，这样可以将字符用作数组的索引。

> ## 定义 1.1
> 令 $`\lt`$ 是字母集合 $`\Sigma`$ 的排序方式，同时也是 $`\Sigma_{}^*`$ 的排序方式：对于 $`s, t \in \Sigma_{}^*`$
> 