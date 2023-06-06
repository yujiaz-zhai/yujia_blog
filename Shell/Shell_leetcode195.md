# Shell：Leetcode 195 第十行

> 关于shell 的基础语法移步[runoob](https://www.runoob.com/linux/linux-shell.html)

### 题目描述

给定一个文本文件 `file.txt`，请只打印这个文件中的第十行。

### 输入样例

```
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
Line 7
Line 8
Line 9
Line 10
```

### 输出样例

```
Line 10
```

### 题解

#### 1、最初想法

拿到题目的第一想法是读入文件后，使用循环找到第十行，如果不足十行，就输出空行

```shell
#! /bin/bash
IFS=$'\n'
i=0
for line in `cat file.txt`
do
    i=$(($i + 1))
    if [ $i -eq 10 ]
    then
        echo $line
    else
        continue
    fi
done
if [ $i -lt 10 ]
then
    echo ""
fi
```

1、`IFS=$'\n'`用于控制读入行的分割，以换行符分割

2、`$(())`相当于`expr`，用于计算表达式

3、`[ ]`中需要加表达式的时候，两端需要加空格

#### 2、`head`, `tail`, `wc` 命令

上面的程序显然可以通过题目，但是并不优雅，于是学习了一下其他命令

**`wc`命令**

`wc`命令用于统计文件中的字节数，字数，行数，并将结果统计输出

* `wc -c` 统计字节数
* `wc -w` 统计字数
* `wc -l` 统计行数

**`head`命令**

```shell
head -n file.txt
```

用于输出前 `n` 行

**`tail`命令**

```shell
tail -n file.txt
```

用于输出后n行

**`|`运算符**

```shell
command1 | command2
```

管道符号，用于将 `command1` 中的输出作为输入传递到 `command2`中

有了如上命令，我们可以进一步修改代码

```
#! /bin/bash
if [ $(cat file.txt | wc -l) -ge 10 ]
then
	cat file.txt | head -n 10 | tail -n 1  #由于 head 和 tail本身就是输出命令，这里就不需要echo了
else
	echo ""
fi
```

#### 3、`awk`,`sed`命令

上述代码还可以再优化一下

**`awk`命令**

文本处理命令，当前只需要用到

```shell
awk 'NR==10' file.txt
```

`NR` 表示当前处理的行号并输出

所以可以将代码改为：

```shell
#! /bin/bash
awk 'NR==10' file.txt
```

**`sed`命令**

这里需要用到：

```shell
sed -n '10p' file.txt
```

`-n` 表示行数

`p`表示打印操作



**`grep`, `sed`, `awk`命令是shell语言的三剑客，这里仅就题论题列举了很少的操作，后面会逐步整理相关操作**