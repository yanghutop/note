# Linux安装JDK



## 1.卸载Linux中原有的OpenJDK

```shell
yum  remove  java
yum  -y  remove  tzdata-java.noarch
```



## 2.下载JDK

去Oracle官网下载最新jdk，选择“*.tar.gz”版本。

i586是32位的，x64是64位的，根据Linux系统位数来选择。

http://www.oracle.com/technetwork/java/javase/downloads/index.html



## 3.将下载好的安装包上传到服务器

建议使用ssh服务 或者 ftp 服务

推荐工具  FileZilla

 

## 4. 将上传好的安装包拷贝到/usr目录中

```shell
mkdir  /usr/local/java
cp  jdk-8u191-linux-x64.tar.gz  /usr/local/java
```



## 5.解压jdk文件

```shell
cd /usr/local/java
tar -xzvf jdk-8u191-linux-x64.tar.gz
```

会得到jdk1.8.0_191目录



## 6. 配置环境变量

准备工作，有些系统没有安装vim，所以当vim不可用时，先安装vim

```shell
yum install vim
```

进行通过vim编辑/etc/profile

```shell
cd /etc
vim profile
```

找到用户配置的位置写入

```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_191

export JRE_HOME=$JAVA_HOME/jre

export CLASSPATH=$JAVA_HOME/lib/

export PATH=$PATH:$JAVA_HOME/bin
```



## 7.让配置文件生效

```shell
source /etc/profile

或者
reboot 
```



## 8.检查

```shell
java -version
```

