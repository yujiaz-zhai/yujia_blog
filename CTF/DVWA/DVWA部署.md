# DVWA部署

介绍使用`Docker`对`DVWA靶场`的部署方法：

## 查询镜像

```dockerfile
docker search DVWA
```

## 拉取镜像

```
docker pull infoslack/dvwa
```

## 查看镜像

```
docker images -a
```

## 启动镜像

```
docker run -d -p 8999:80 infoslack/dvwa
```

**注意：记得在服务器控制台的安全组中把8999端口放行，不然打不开**

## 启动DVWA

```
ip:8999
```

需要注册

* `username`: `admin`
* `password`: `password`

在`DVWA Security`中调整难度