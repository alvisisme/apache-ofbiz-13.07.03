# OFBIZ 框架

本工程为apache-ofbiz-13.07.03的原始工程存档

## 如何运行本工程

windows下执行如下命令

加载测试数据

```
ant.bat load-demo
```

运行本工程

```
ant.bat start
```

访问如下地址测试

电子商城前台 http://localhost:8080/ecommerce

订单管理后台 https://localhost:8443/ordermgr

管理面板 https://localhost:8443/webtools

## 其他

* [使用mysql替换默认的derby数据库](docs/setup-mysql.md)
* [windows下运行防止中文乱码的设置](docs/windows-utf8-encoding.md)
