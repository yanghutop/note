# Linux安装TomCat



## 1.下载TomCat

官网网址

http://tomcat.apache.org/



## 2.将下载的压缩包上传到服务器

建议使用ssh服务 或者 ftp 服务

推荐工具 FileZilla



## 3.将上传好的安装包拷贝到/usr目录中

```shell
mkdir  /usr/local/tomcat
cp  apache-tomcat-8.5.49.tar.gz  /usr/local/tomcat
```



## 4.解压tomcat文件

```shell
cd /usr/local/tomcat
tar -xzvf apache-tomcat-8.5.49.tar.gz
```

会得到apache-tomcat-8.5.49目录



## 5.运行tomcat

先进入tomcat的bin目录

```shell
cd /usr/local/tomcat/apache-tomcat-8.5.49/bin
```

执行运行脚本

```shell
sh startup.sh
```



## 6.测试

通过tailf 来追踪tomcat日志

```shell
tailf catalina.out

按ctrl+c 结束
```

