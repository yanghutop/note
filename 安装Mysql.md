# Linux安装MySql



## 1.获取MySql

```shell
https://cdn.mysql.com/archives/mysql-8.0/mysql-8.0.18-el7-x86_64.tar.gz
```



## 2.解决依赖工具

```shell
yum -y install wget cmake gcc gcc-c++ ncurses ncurses-devel libaio-devel openssl openssl-devel
```





## 3.创建目录和用户

```shell
mkdir  /usr/local/mysql
mkdir /usr/local/mysql/data
groupadd  mysql
useradd -r -g mysql -s /bin/false mysql
tar -zvxf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz 
cd /usr/local/mysql
chown -R mysql:mysql ./ (在/usr/local/mysql)

```


## 4. 创建mysql安装初始化配置文件my.cnf

```shell
vi  /etc/my.cnf
```

```config
[mysqld]

port=3306

basedir=/usr/local/mysql/mysql8

datadir=/usr/local/mysql/data

max_connections=500

max_connect_errors=10

character-set-server= utf8mb4

default-storage-engine=INNODB

default_authentication_plugin=mysql_native_password

lower_case_table_names=1

[mysql]

default-character-set= utf8mb4
[client]

port=3306
default-character-set= utf8mb4

```






## 5. 初始化

```shell
cd /usr/local/mysql/mysql8xxxx/bin
./mysqld --initialize --user=mysql  --datadir=/usr/local/mysql/data/ 
```

## 6.启动mysql

```shell
./mysqld_safe --user=mysql &
```

## 7.修改账号密码

```shell
./mysql -uroot -p（输入在初始成功后显示的密码）

alter user 'root'@'localhost' identified by "123456";
create user root@'%' identified by '123456';
grant all privileges on *.* to root@'%';
flush privileges;

```

## 8.外部工具尝试登录服务器MySQL进行测试

