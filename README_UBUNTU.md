# Ubuntu 下安装编译指南

## 依赖
要先升级openssl
【依赖文件】
redis

zookeeper		
apt install zookeeper
kafka			
apt install kafka
无法安装时用源码安装

librdkafka
github源码安装
https://github.com/edenhill/librdkafka.git
要注释掉configure.self里面的
   	# mkl_check "libsasl2" disable
   	# mkl_check "zstd" disable
libev			
源码安装
libmpdec		
源码安装
jansson			
源码安装
libmysqlclient-dev (apt-get install libmysqlclient-dev)
apt-get install libmysqlclient-dev
http_parser
github源码安装
https://github.com/nodejs/http-parser.git
libcurl
源码安装
注意configure时关闭一些功能
./configure --disable-ldap --disable-ldaps


## 编译

### 编译顺序
    先编译 network,utils  
    然后编译 accesshttp,accessws  
    接着编译 marketprice,matchengine  
    最后编译 alertcenter,readhistory  
### 编译错误
#### utils
- openssl/bio.h: No such file or directory  
    先安装libssl-dev
    ```
    sudo apt install libssl-dev
    ```

#### accessws
- curl出错  
    makefile里面-lcurl移到末尾改成动态链接

- cannot find -llz4
    ```
    apt-get install -y liblz4-dev
    ```