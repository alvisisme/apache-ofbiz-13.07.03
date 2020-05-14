# windows下防止中文乱码

修改启动脚本，启动时指明以utf-8编码模式启动

修改**tools/startofbiz.bat**脚本，启动参数加入**-Dfile.encoding=UTF-8**，如下

```
"%JAVA_HOME%\bin\java" -Xms128M -Xmx512M -XX:MaxPermSize=512m -Dfile.encoding=UTF-8 -jar ofbiz.jar
```

如果通过执行**ant start**命令启动，还需要修改**ant.bat**文件

修改启动参数，加入 **-Dfile.encoding=UTF-8"** 一行

```
"%JAVA_HOME%\bin\java" -jar framework/base/lib/ant-1.9.0-ant-launcher.jar -lib framework/base/lib/ant -Dfile.encoding=UTF-8 %1 %2 %3 %4 %5 %6
```
