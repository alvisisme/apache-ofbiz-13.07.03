# 使用Mysql数据库作为后端数据库配置过程

## 使用docker搭建数据库服务
```
version: '2'
services:
  mysql:
    image: mysql:5.7.17
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secret
    ports:
      - "3306:3306"
    volumes:
      - mysql:/var/lib/mysql

volumes:
  mysql:
```

使用navicat等图形化工具建立三个空数据库，**ofbiz**，**ofbizolap**和**ofbiztenant**

字符集为**utf-8**, 排序规则为**utf8-general-ci**

注意: 必须先创建这三个空数据库，否则后续无法正常启动。

linux下数据库区分大小写，需要配置启动参数

```
version: '2'
services:
  mysql:
    image: mysql:5.7.17
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secret
    ports:
      - "3306:3306"
    volumes:
      - mysql:/var/lib/mysql
      - ./docker.cnf:/etc/mysql/conf.d/docker.cnf

volumes:
  mysql:
```

docker.cnf 文件

```
[mysqld]
skip-host-cache
skip-name-resolve
lower_case_table_names=1
```

## 配置ofbiz

### 安装mysql驱动

执行如下命令安装mysql驱动

```
ant download-mySQL-JDBC
```

驱动包位于ofbiz工程下的**framework/entity/lib/jdbc** 目录下

### 配置数据库参数

打开**framework/entity/config/entityengine.xml**文件，修改 **delegator** 字段，将**datasource-name**字段改为对应的mysql数据源，

可以看到，总共有三个数据库代理，分别代理着三个数据库源。

```
<delegator name="default" entity-model-reader="main" entity-group-reader="main" entity-eca-reader="main" distributed-cache-clear-enabled="false">
    <group-map group-name="org.ofbiz" datasource-name="localmysql"/>
    <group-map group-name="org.ofbiz.olap" datasource-name="localmysqlolap"/>
    <group-map group-name="org.ofbiz.tenant" datasource-name="localmysqltenant"/>
</delegator>
<delegator name="default-no-eca" entity-model-reader="main" entity-group-reader="main" entity-eca-reader="main" entity-eca-enabled="false" distributed-cache-clear-enabled="false">
    <group-map group-name="org.ofbiz" datasource-name="localmysql"/>
    <group-map group-name="org.ofbiz.olap" datasource-name="localmysqlolap"/>
    <group-map group-name="org.ofbiz.tenant" datasource-name="localmysqltenant"/>
</delegator>

<!-- be sure that your default delegator (or the one you use) uses the same datasource for test. You must run "ant load-demo" before running "ant run-tests" -->
<delegator name="test" entity-model-reader="main" entity-group-reader="main" entity-eca-reader="main">
    <group-map group-name="org.ofbiz" datasource-name="localmysql"/>
    <group-map group-name="org.ofbiz.olap" datasource-name="localmysqlolap"/>
    <group-map group-name="org.ofbiz.tenant" datasource-name="localmysqltenant"/>
</delegator>
```

然后找到对应的**datasource-name**源配置项，修改配置项与建立的数据库对应

如 name 为 **localmysql** 的数据源配置项如下

```
    <datasource name="localmysql"
            helper-class="org.ofbiz.entity.datasource.GenericHelperDAO"
            field-type-name="mysql"
            check-on-start="true"
            add-missing-on-start="true"
            check-pks-on-start="false"
            use-foreign-keys="true"
            join-style="ansi-no-parenthesis"
            alias-view-columns="false"
            drop-fk-use-foreign-key-keyword="true"
            table-type="InnoDB"
            character-set="utf8"
            collate="utf8_general_ci">
        <read-data reader-name="tenant"/>
        <read-data reader-name="seed"/>
        <read-data reader-name="seed-initial"/>
        <read-data reader-name="demo"/>
        <read-data reader-name="ext"/>
        <read-data reader-name="ext-test"/>
        <read-data reader-name="ext-demo"/>
        <inline-jdbc
                jdbc-driver="com.mysql.jdbc.Driver"
                jdbc-uri="jdbc:mysql://127.0.0.1:3306/ofbiz?autoReconnect=true&amp;characterEncoding=UTF-8"
                jdbc-username="root"
                jdbc-password="secret"
                isolation-level="ReadCommitted"
                pool-minsize="2"
                pool-maxsize="250"
                time-between-eviction-runs-millis="600000"/><!-- Please note that at least one person has experienced a problem with this value with MySQL
                and had to set it to -1 in order to avoid this issue.
                For more look at http://markmail.org/thread/5sivpykv7xkl66px and http://commons.apache.org/dbcp/configuration.html-->
        <!-- <jndi-jdbc jndi-server-name="localjndi" jndi-name="java:/MySqlDataSource" isolation-level="Serializable"/> -->
    </datasource>
```

主要修改的是

数据库默认编码 **character-set="utf8"** 和 **collate="utf8_general_ci"**

数据库连接url **jdbc-uri="jdbc:mysql://127.0.0.1:3306/ofbiz?autoReconnect=true&amp;characterEncoding=UTF-8"**

数据库连接用户名和密码 **jdbc-username="root"** 和 **jdbc-password="secret"**

确保和需要连接的数据库参数保持一致。

注意配置三个数据源时参数的区别。

## 测试

配置完成后，进入ofbiz目录，执行如下命令启动

```
ant.bat load-demo start

```

稍等片刻后，访问 **http://localhost:8080/ecommerce** 测试能否正常进入。
