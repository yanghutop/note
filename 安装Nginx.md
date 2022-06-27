# Linux安装Nginx



## 1.下载Nginx

官网网址

https://nginx.org/en/download.html



## 2.解决依赖工具

```shell
yum install -y gcc  gcc-c++  pcre pcre-devel  zlib zlib-devel openssl openssl-devel
```



## 3.将上传好的安装包拷贝到/usr目录中

```shell
mkdir  /usr/local/nginx
cp  nginx-1.16.0.tar.gz  /usr/local/nginx
```



## 4.解压nginx文件

```shell
cd /usr/local/nginx
tar -xzvf nginx-1.16.0.tar.gz
```

会得到nginx-1.16.0目录



## 5.安装nginx

先进入nginx的目录执行configure

```shell
./configure
```

执行编译并安装

```shell
make  &&  make install
```



## 6.启动nginx

```shell
cd ./sbin
./nginx
```



## 7.停止nginx，重启nginx

```shell
cd ./sbin
./nginx -s stop
./nginx -s reload
```

